# Methods in C# — Stored Procedures That Return Values

## A Method Is a Stored Procedure With Superpowers

In T-SQL, a stored procedure is a named, reusable block of code. A C# **method** is the same concept — but more flexible:

| T-SQL Stored Procedure | C# Method |
|---|---|
| `CREATE PROCEDURE ProcName` | `ReturnType MethodName()` |
| `@param INT` | `int param` |
| `@result INT OUTPUT` | `out int result` or just return a value |
| Can't return a value directly | Returns a value directly |
| Can only have one "result set" | Can return any type, including complex objects |
| Same name = error | Overloading: same name, different parameters ✓ |

---

## Anatomy of a Method

```csharp
// access modifier  return type   name          parameters
   public           decimal       CalculateFee  (decimal amount, string transactionType)
{
    // method body
    decimal fee = amount * 0.01m;
    return fee;    // returns the value to the caller
}
```

### Return Types

```csharp
// Returns nothing (like a proc with no OUTPUT and no SELECT)
public void LogTransaction(int transactionId)
{
    Console.WriteLine($"Logged transaction {transactionId}");
    // no return statement needed
}

// Returns a single value
public decimal CalculateFee(decimal amount)
{
    return amount * 0.01m;
}

// Returns a complex object
public Transaction GetTransaction(int id)
{
    return new Transaction { Id = id, Amount = 100m };
}

// Returns true/false (common for validation)
public bool IsValidAmount(decimal amount)
{
    return amount > 0 && amount <= 1_000_000m;
}
```

### Access Modifiers

```csharp
public decimal Calculate()    { }  // Callable from anywhere
private decimal Helper()      { }  // Only callable within this class
protected decimal Base()      { }  // Callable within this class and subclasses
internal decimal Module()     { }  // Callable within this project/assembly
```

> **SQL analogy:** `public` is like `GRANT EXECUTE` to all. `private` is like `DENY EXECUTE` to everyone except the procedure's own schema.

---

## Calling a Method

```csharp
// Define the method in a class
public decimal CalculateFee(decimal amount)
{
    return amount * 0.01m;
}

// Call it — capture the return value
decimal fee = CalculateFee(500m);   // fee = 5.0m

// Call it inline in an expression
Console.WriteLine($"Fee: {CalculateFee(500m):C}");

// Call void methods without capturing anything
LogTransaction(42);
```

---

## Parameters

### Required Parameters

```sql
-- T-SQL
CREATE PROCEDURE CalculateFee
    @amount DECIMAL(18,2),
    @type VARCHAR(10)
AS ...
```

```csharp
// C#
public decimal CalculateFee(decimal amount, string type)
{
    ...
}
// Must be called with both arguments:
decimal fee = CalculateFee(100m, "DEP");
```

### Optional Parameters (Default Values)

```sql
-- T-SQL: no default values, must use workarounds
CREATE PROCEDURE CalculateFee
    @amount DECIMAL(18,2),
    @type VARCHAR(10) = 'DEP'    -- default value
AS ...
```

```csharp
// C#: default values in the signature
public decimal CalculateFee(decimal amount, string type = "DEP")
{
    ...
}

// Can omit the optional parameter:
decimal fee1 = CalculateFee(100m);           // type defaults to "DEP"
decimal fee2 = CalculateFee(100m, "WIT");    // explicit override
```

### Named Arguments

C# lets you call methods using parameter names, in any order:

```csharp
decimal fee = CalculateFee(type: "WIT", amount: 250m);
```

This is useful when a method has many parameters and you want to be explicit.

---

## Output Parameters — `out` and `ref`

### T-SQL OUTPUT Parameters

```sql
CREATE PROCEDURE ProcessPayment
    @amount DECIMAL(18,2),
    @transactionId INT OUTPUT,
    @fee DECIMAL(18,2) OUTPUT
AS
BEGIN
    SET @transactionId = NEXT VALUE FOR dbo.TransactionSeq;
    SET @fee = @amount * 0.01;
END;
```

### C# `out` Parameters

The `out` keyword is the direct equivalent of `OUTPUT` in T-SQL:

```csharp
// Method with out parameters
public bool ProcessPayment(decimal amount, out int transactionId, out decimal fee)
{
    transactionId = GenerateNextId();
    fee = amount * 0.01m;
    return true;    // success/failure as return value
}

// Calling a method with out parameters
if (ProcessPayment(500m, out int txId, out decimal calculatedFee))
{
    Console.WriteLine($"Transaction {txId} processed, fee: {calculatedFee:C}");
}

// If you don't need an out value, use discard _
ProcessPayment(500m, out int txId, out _);
```

> **Key difference from T-SQL OUTPUT:** In T-SQL, you pass a variable that gets modified. In C#, `out` parameters *must* be assigned inside the method before it returns. The compiler enforces this.

### C# `ref` Parameters

`ref` passes a variable by reference — the method can both read AND modify it:

```csharp
public void ApplyDiscount(ref decimal amount, decimal discountPercent)
{
    amount = amount * (1 - discountPercent / 100m);
}

decimal price = 100m;
ApplyDiscount(ref price, 10m);   // price is now 90m
```

| | `out` | `ref` |
|---|---|---|
| Must be initialized before calling | No | Yes |
| Must be assigned inside the method | Yes (compiler enforced) | No |
| Use case | Return multiple values | Modify an existing variable |

> **Tip:** In modern C#, prefer returning a tuple or a result object over `out`/`ref` parameters. They exist mainly for interoperability with older APIs (like `int.TryParse`).

### `int.TryParse` — The Classic `out` Example

```sql
-- T-SQL
DECLARE @result INT;
BEGIN TRY
    SET @result = TRY_CAST(@input AS INT);
END TRY ...
```

```csharp
// C# — TryParse uses out parameter pattern
if (int.TryParse(userInput, out int result))
{
    Console.WriteLine($"Parsed: {result}");
}
else
{
    Console.WriteLine("Not a valid integer");
}
```

---

## params — Variable Number of Arguments

```csharp
// params lets you pass any number of arguments
public decimal Sum(params decimal[] amounts)
{
    decimal total = 0m;
    foreach (var amount in amounts)
        total += amount;
    return total;
}

// Call with 2, 3, or any number of arguments
decimal a = Sum(10m, 20m);
decimal b = Sum(10m, 20m, 30m, 40m);
decimal c = Sum(100m);
```

> **SQL analogy:** No direct equivalent. In T-SQL you'd use a table-valued parameter or a delimited string.

---

## Method Overloading

In T-SQL, two procedures cannot have the same name. In C#, methods *can* share a name if their parameter lists differ:

```sql
-- T-SQL: CANNOT do this — would be an error
CREATE PROCEDURE CalculateFee @amount DECIMAL AS ...
CREATE PROCEDURE CalculateFee @amount DECIMAL, @type VARCHAR AS ...  -- ERROR!
```

```csharp
// C#: Overloading is allowed and common
public decimal CalculateFee(decimal amount)
{
    return CalculateFee(amount, "STANDARD");    // delegate to the full version
}

public decimal CalculateFee(decimal amount, string transactionType)
{
    return transactionType switch
    {
        "STANDARD" => amount * 0.01m,
        "EXPRESS"  => amount * 0.02m,
        "VIP"      => 0m,
        _          => amount * 0.015m
    };
}

public decimal CalculateFee(decimal amount, string transactionType, bool isWeekend)
{
    decimal fee = CalculateFee(amount, transactionType);
    return isWeekend ? fee * 1.5m : fee;
}
```

The compiler picks the right overload based on what arguments you pass:

```csharp
CalculateFee(100m);                        // calls first overload
CalculateFee(100m, "EXPRESS");             // calls second overload
CalculateFee(100m, "EXPRESS", true);       // calls third overload
```

### When to overload vs use optional parameters

| Use overloading when | Use optional parameters when |
|---|---|
| Different logic for different cases | Same logic, just a missing optional argument |
| Parameters have different types | Same types, just optional |
| Behavior differs significantly | Behavior is the same, just a default applied |

---

## Expression-Bodied Methods

When a method body is a single expression, you can use `=>` shorthand:

```csharp
// Full method
public decimal CalculateVatAmount(decimal amount)
{
    return amount * 0.20m;
}

// Expression-bodied (same thing, shorter)
public decimal CalculateVatAmount(decimal amount) => amount * 0.20m;

public bool IsPositive(decimal amount) => amount > 0;
public string FormatAmount(decimal amount) => $"{amount:C}";
```

---

## Local Functions

Methods can contain other methods, called **local functions**. These are useful for helper logic that only makes sense inside one method:

```csharp
public decimal CalculateComplexFee(decimal amount, string type, bool isVip)
{
    decimal baseFee = GetBaseFee(type);         // local function call
    decimal multiplier = isVip ? 0.5m : 1.0m;
    return baseFee * multiplier;

    // Local function — only accessible inside CalculateComplexFee
    decimal GetBaseFee(string transactionType) => transactionType switch
    {
        "DEP" => amount * 0.005m,
        "WIT" => amount * 0.01m,
        _     => 5.00m
    };
}
```

---

## Summary: T-SQL Procedure → C# Method

```sql
-- T-SQL stored procedure
CREATE PROCEDURE usp_CalculateFee
    @amount     DECIMAL(18,2),
    @type       VARCHAR(10) = 'STANDARD',
    @fee        DECIMAL(18,2) OUTPUT
AS
BEGIN
    IF @type = 'EXPRESS'
        SET @fee = @amount * 0.02;
    ELSE IF @type = 'VIP'
        SET @fee = 0;
    ELSE
        SET @fee = @amount * 0.01;
END;
```

```csharp
// C# equivalent — cleaner, more flexible
public decimal CalculateFee(decimal amount, string type = "STANDARD")
{
    return type switch
    {
        "EXPRESS" => amount * 0.02m,
        "VIP"     => 0m,
        _         => amount * 0.01m
    };
}
```

The C# version:
- Returns the value directly (no `OUTPUT` parameter needed)
- Uses a switch expression instead of `IF/ELSE IF`
- Has a default parameter value
- Is shorter and clearer
