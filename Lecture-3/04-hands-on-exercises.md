# Hands-On Exercises — Session 3

## The Goal

Rewrite a T-SQL fee calculation stored procedure as a set of C# methods. By the end, you will have a working console application that reproduces the exact same business logic — without a single line of SQL.

---

## The Original T-SQL Procedure

This is the procedure you're converting. Study it carefully before writing any C# code.

```sql
CREATE PROCEDURE usp_CalculateTransactionFee
    @AccountId      INT,
    @Amount         DECIMAL(18,2),
    @TransType      VARCHAR(10),   -- 'DEP', 'WIT', 'TRF', 'FEE'
    @IsWeekend      BIT,
    @IsPremium      BIT,
    @Fee            DECIMAL(18,2) OUTPUT,
    @FeeDescription VARCHAR(200)  OUTPUT
AS
BEGIN
    DECLARE @BaseFee    DECIMAL(18,2);
    DECLARE @Multiplier DECIMAL(18,4);

    -- Validate amount
    IF @Amount <= 0
    BEGIN
        RAISERROR('Amount must be positive.', 16, 1);
        RETURN;
    END

    IF @Amount > 1000000
    BEGIN
        RAISERROR('Amount exceeds maximum limit.', 16, 1);
        RETURN;
    END

    -- Calculate base fee by transaction type
    IF @TransType = 'DEP'
        SET @BaseFee = 0.00;
    ELSE IF @TransType = 'WIT'
    BEGIN
        IF @Amount <= 500
            SET @BaseFee = 2.50;
        ELSE IF @Amount <= 5000
            SET @BaseFee = @Amount * 0.005;
        ELSE
            SET @BaseFee = @Amount * 0.008;
    END
    ELSE IF @TransType = 'TRF'
        SET @BaseFee = @Amount * 0.0025;
    ELSE IF @TransType = 'FEE'
        SET @BaseFee = @Amount;   -- fee is the amount itself
    ELSE
    BEGIN
        RAISERROR('Unknown transaction type.', 16, 1);
        RETURN;
    END

    -- Apply multipliers
    SET @Multiplier = 1.0;

    IF @IsWeekend = 1
        SET @Multiplier = @Multiplier * 1.5;    -- 50% weekend surcharge

    IF @IsPremium = 1
        SET @Multiplier = @Multiplier * 0.7;    -- 30% premium discount

    SET @Fee = @BaseFee * @Multiplier;

    -- Build description
    SET @FeeDescription = 'Type: ' + @TransType
        + ', Base: ' + CAST(@BaseFee AS VARCHAR(20))
        + ', Multiplier: ' + CAST(@Multiplier AS VARCHAR(20))
        + ', Final: ' + CAST(@Fee AS VARCHAR(20));
END;
```

---

## Exercise 1: Validate the Input

Create a console project and write a method that validates the transaction amount. It should throw an exception (we'll cover exceptions properly in Session 9 — for now, use `throw new ArgumentException(...)`) if invalid.

### Requirements
- Amount must be greater than 0
- Amount must not exceed 1,000,000
- `TransType` must be one of: `DEP`, `WIT`, `TRF`, `FEE`

```csharp
// Starter structure — fill in the method body
static void ValidateTransaction(decimal amount, string transType)
{
    // TODO: validate amount > 0
    // TODO: validate amount <= 1_000_000
    // TODO: validate transType is one of the valid values
}
```

**Hint:** Use an array and `.Contains()`, or a `switch` expression that throws on default.

---

## Exercise 2: Calculate the Base Fee

Write a method that calculates the base fee based on transaction type and amount.

```csharp
// Return the base fee — no side effects, no printing
static decimal CalculateBaseFee(decimal amount, string transType)
{
    // TODO: implement the same logic as the T-SQL procedure
    // DEP  → 0
    // WIT  → tiered: <=500 = 2.50, <=5000 = 0.5%, >5000 = 0.8%
    // TRF  → 0.25%
    // FEE  → amount itself
    // else → throw ArgumentException
}
```

**Challenge:** Use a switch expression for the transaction type, and a ternary or nested conditions for the withdrawal tiers.

---

## Exercise 3: Apply Multipliers

Write a method that applies the weekend and premium multipliers.

```csharp
static decimal ApplyMultipliers(decimal baseFee, bool isWeekend, bool isPremium)
{
    // TODO: start with multiplier = 1.0m
    // TODO: if isWeekend, multiply by 1.5
    // TODO: if isPremium, multiply by 0.7
    // TODO: return baseFee * finalMultiplier
}
```

---

## Exercise 4: Build the Description

Write a method that builds the fee description string.

```csharp
static string BuildFeeDescription(string transType, decimal baseFee, decimal multiplier, decimal finalFee)
{
    // TODO: return a string like:
    // "Type: WIT, Base: 12.50, Multiplier: 1.50, Final: 18.75"
    // Use string interpolation: $"..."
    // Format decimals with 2 decimal places: {value:F2}
}
```

---

## Exercise 5: Compose the Full Calculation

Write the main method that calls all the above methods in sequence. This replaces the entire stored procedure:

```csharp
static (decimal Fee, string Description) CalculateTransactionFee(
    int accountId,
    decimal amount,
    string transType,
    bool isWeekend,
    bool isPremium)
{
    // TODO:
    // 1. Call ValidateTransaction — let exception propagate if invalid
    // 2. Call CalculateBaseFee
    // 3. Determine multiplier (you can inline or extract to a method)
    // 4. Calculate final fee
    // 5. Call BuildFeeDescription
    // 6. Return (fee, description) as a tuple
}
```

---

## Exercise 6: Test It in Main

Write a `Main` method (or top-level statements) that tests several cases and prints the results. Your output should match the expected values below.

```csharp
// Test cases to run:
var testCases = new[]
{
    (AccountId: 1,  Amount: 1000m,  Type: "DEP", IsWeekend: false, IsPremium: false),
    (AccountId: 2,  Amount: 300m,   Type: "WIT", IsWeekend: false, IsPremium: false),
    (AccountId: 3,  Amount: 3000m,  Type: "WIT", IsWeekend: true,  IsPremium: false),
    (AccountId: 4,  Amount: 10000m, Type: "TRF", IsWeekend: false, IsPremium: true),
    (AccountId: 5,  Amount: 50m,    Type: "FEE", IsWeekend: false, IsPremium: false),
};

foreach (var tc in testCases)
{
    var (fee, desc) = CalculateTransactionFee(
        tc.AccountId, tc.Amount, tc.Type, tc.IsWeekend, tc.IsPremium);
    Console.WriteLine($"Account {tc.AccountId}: {desc}");
}
```

### Expected Output

```
Account 1: Type: DEP, Base: 0.00, Multiplier: 1.00, Final: 0.00
Account 2: Type: WIT, Base: 2.50, Multiplier: 1.00, Final: 2.50
Account 3: Type: WIT, Base: 15.00, Multiplier: 1.50, Final: 22.50
Account 4: Type: TRF, Base: 25.00, Multiplier: 0.70, Final: 17.50
Account 5: Type: FEE, Base: 50.00, Multiplier: 1.00, Final: 50.00
```

---

## Exercise 7: Edge Cases and Error Handling

Add these calls after the main test loop and handle the exceptions gracefully with `try/catch`:

```csharp
// These should throw exceptions:
try { CalculateTransactionFee(99, -50m,      "WIT", false, false); }
catch (ArgumentException ex) { Console.WriteLine($"Error: {ex.Message}"); }

try { CalculateTransactionFee(99, 2000000m,  "TRF", false, false); }
catch (ArgumentException ex) { Console.WriteLine($"Error: {ex.Message}"); }

try { CalculateTransactionFee(99, 100m,      "XYZ", false, false); }
catch (ArgumentException ex) { Console.WriteLine($"Error: {ex.Message}"); }
```

---

## Bonus: Method Overloading

Provide an overloaded version of `CalculateTransactionFee` that takes no boolean parameters and assumes defaults (not weekend, not premium):

```csharp
static (decimal Fee, string Description) CalculateTransactionFee(
    int accountId, decimal amount, string transType)
{
    // TODO: delegate to the full version with defaults
}
```

---

## Reflection Questions

After completing the exercises, think about these:

1. **How does the C# version compare to the T-SQL procedure in terms of readability?**
2. **The T-SQL procedure uses `RAISERROR` for validation. How is `throw new ArgumentException(...)` similar/different?**
3. **In the T-SQL procedure, `@Fee` and `@FeeDescription` are OUTPUT parameters. In C#, we returned a tuple. What are the tradeoffs?**
4. **Would you rather have one large method that does everything, or several small focused methods? Why?**

---

## Reference: Key Concepts Used in These Exercises

| Concept | Where Used |
|---|---|
| `if/else if/else` | Withdrawal fee tiers, validation |
| `switch` expression | Transaction type routing |
| `decimal` arithmetic | Fee calculations |
| Methods with return types | All helper methods |
| Method with multiple returns | `CalculateTransactionFee` returns a tuple |
| `params` or array | Test cases array |
| `foreach` loop | Iterating test cases |
| `try/catch` | Error handling exercise |
| String interpolation `$"..."` | Description building |
| Tuple `(T1, T2)` | Return multiple values without `out` |
