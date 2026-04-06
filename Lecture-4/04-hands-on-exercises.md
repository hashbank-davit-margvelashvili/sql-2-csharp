# Hands-On Exercises — Session 4: Model Real HSP Entities as C# Classes

## The Goal

Transform **real HSP database tables and functions** into proper C# classes with properties, constructors, methods, and access control. In Session 3 you wrote static methods. Now you'll give them a home — **objects that own their data and behavior**.

---

## Project Setup

Continue using the same solution from Session 3, or create a new one:

### 1. Class Library (your implementations)

```bash
dotnet new classlib -n HspModels
```

Organize your classes — one class per file:

```
HspModels/
  CurrencyCode.cs
  CurrencyPair.cs
  CurrencyRate.cs
  BankAccount.cs
  CurrencyConverter.cs    (static class)
  AccountNumberService.cs (static class)
```

### 2. xUnit Test Project

```bash
dotnet new xunit -n HspModels.Tests
cd HspModels.Tests
dotnet add reference ../HspModels/HspModels.csproj
```

If you need your Session 3 functions (e.g., `IbanMod97`), also add a reference to the HspFunctions project:

```bash
dotnet add reference ../HspFunctions/HspFunctions.csproj
```

Run tests with:
```bash
cd HspModels.Tests
dotnet test
```

---

## Exercise 1: Currency Code — `hw.CurrencyCodes3` as a Class

**Concepts:** Class with properties, constructor with validation, `{ get; }` (immutable), `ToString()` override

### The HSP Table

```sql
-- Source: hw.CurrencyCodes3
CREATE TABLE hw.CurrencyCodes3 (
    CurrencyCode   VARCHAR(5),    -- "GEL", "USD", "EUR"
    CurrencyNumber VARCHAR(5),    -- "981", "840", "978"
    CurrencyScale  INT            -- 2 (for cents)
);
```

### Your Task

```csharp
// CurrencyCode.cs
public class CurrencyCode
{
    // TODO: Properties (all read-only after construction)
    // - Code (string): 3-letter ISO code like "GEL", "USD"
    // - NumericCode (string): numeric ISO code like "981", "840"
    // - Scale (int): decimal places, typically 2

    // TODO: Constructor
    // - Takes code, numericCode, scale
    // - Validates:
    //   - code must not be null/empty and must be exactly 3 characters
    //   - numericCode must not be null/empty
    //   - scale must be >= 0
    // - Stores code as uppercase

    // TODO: Override ToString()
    // - Returns: "GEL (981)" format

    // TODO: Static factory methods for common currencies
    // - public static CurrencyCode GEL() => new CurrencyCode("GEL", "981", 2);
    // - public static CurrencyCode USD() => new CurrencyCode("USD", "840", 2);
    // - public static CurrencyCode EUR() => new CurrencyCode("EUR", "978", 2);
}
```

### Test Cases — Implement in `CurrencyCodeTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Constructor sets Code | `new CurrencyCode("GEL", "981", 2).Code` | `"GEL"` |
| 2 | Constructor sets NumericCode | `new CurrencyCode("GEL", "981", 2).NumericCode` | `"981"` |
| 3 | Constructor sets Scale | `new CurrencyCode("GEL", "981", 2).Scale` | `2` |
| 4 | Code is stored as uppercase | `new CurrencyCode("gel", "981", 2).Code` | `"GEL"` |
| 5 | Null code throws ArgumentException | `new CurrencyCode(null, "981", 2)` | throws |
| 6 | Empty code throws | `new CurrencyCode("", "981", 2)` | throws |
| 7 | Code with wrong length throws | `new CurrencyCode("US", "840", 2)` | throws |
| 8 | Negative scale throws | `new CurrencyCode("GEL", "981", -1)` | throws |
| 9 | ToString returns formatted string | `new CurrencyCode("USD", "840", 2).ToString()` | `"USD (840)"` |
| 10 | Static GEL() factory | `CurrencyCode.GEL().Code` | `"GEL"` |
| 11 | Static USD() factory | `CurrencyCode.USD().NumericCode` | `"840"` |
| 12 | Static EUR() factory | `CurrencyCode.EUR().Code` | `"EUR"` |

**xUnit hint:** Testing for exceptions:
```csharp
[Fact]
public void Constructor_NullCode_ThrowsArgumentException()
{
    Assert.Throws<ArgumentException>(() => new CurrencyCode(null!, "981", 2));
}
```

---

## Exercise 2: Currency Pair — `hp.CurrencyPairs` as a Class

**Concepts:** Computed properties, `{ get; init; }`, `bool` properties with defaults, `DateTime` defaults

### The HSP Table

```sql
-- Source: hp.CurrencyPairs
CREATE TABLE hp.CurrencyPairs (
    PairID        INT IDENTITY PRIMARY KEY,
    BaseCurrency  VARCHAR(3) NOT NULL,
    QuoteCurrency VARCHAR(3) NOT NULL,
    PairCode      AS (BaseCurrency + '/' + QuoteCurrency),           -- computed
    IsActive      BIT NOT NULL DEFAULT 1,
    CreateDate    DATETIME NOT NULL DEFAULT GETDATE(),
    NormalizedKey AS (UPPER(BaseCurrency) + '/' + UPPER(QuoteCurrency)) -- computed
);
```

### Your Task

```csharp
// CurrencyPair.cs
public class CurrencyPair
{
    // TODO: Properties
    // - PairId (int): get-only, set in constructor
    // - BaseCurrency (string): get-only, set in constructor
    // - QuoteCurrency (string): get-only, set in constructor
    // - PairCode (string): COMPUTED property → "USD/GEL" (from BaseCurrency + "/" + QuoteCurrency)
    // - NormalizedKey (string): COMPUTED property → uppercase version of PairCode
    // - IsActive (bool): get/set, default true
    // - CreateDate (DateTime): get-only, defaults to DateTime.Now

    // TODO: Constructor(int pairId, string baseCurrency, string quoteCurrency)
    // - Validates:
    //   - baseCurrency and quoteCurrency must be non-empty, 3 chars
    //   - baseCurrency != quoteCurrency
    // - Stores currencies as uppercase

    // TODO: Methods
    // - public CurrencyPair Invert()
    //     Returns a NEW CurrencyPair with base and quote swapped
    //     e.g., USD/GEL → GEL/USD
    //     The inverted pair gets a new PairId (use negative of original, or 0)

    // TODO: Override ToString()
    // - Returns PairCode (e.g., "USD/GEL")
}
```

### Test Cases — Implement in `CurrencyPairTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | PairCode computed correctly | `new CurrencyPair(1, "USD", "GEL").PairCode` | `"USD/GEL"` |
| 2 | NormalizedKey is uppercase | `new CurrencyPair(1, "usd", "gel").NormalizedKey` | `"USD/GEL"` |
| 3 | IsActive defaults to true | `new CurrencyPair(1, "USD", "GEL").IsActive` | `true` |
| 4 | IsActive can be changed | set `pair.IsActive = false` | `false` |
| 5 | CreateDate is set automatically | `new CurrencyPair(...)` | CreateDate close to DateTime.Now |
| 6 | Same currency throws | `new CurrencyPair(1, "GEL", "GEL")` | throws |
| 7 | Empty base currency throws | `new CurrencyPair(1, "", "GEL")` | throws |
| 8 | Invert swaps currencies | `new CurrencyPair(1, "USD", "GEL").Invert().PairCode` | `"GEL/USD"` |
| 9 | Invert creates a new object | `pair.Invert()` is not same reference as `pair` | different objects |
| 10 | ToString returns PairCode | `new CurrencyPair(1, "EUR", "GEL").ToString()` | `"EUR/GEL"` |

---

## Exercise 3: Currency Rate — `hp.CurrencyRates` as a Class

**Concepts:** Multiple constructors, nullable properties, constructor chaining, computed property

### The HSP Table

```sql
-- Source: hp.CurrencyRates
CREATE TABLE hp.CurrencyRates (
    DT            DATETIME NOT NULL,
    SellCCY       VARCHAR(5) NOT NULL,
    BuyCCY        VARCHAR(5) NOT NULL,
    Rate          DECIMAL(38,8) NULL,      -- NBG rate
    ComercialRate DECIMAL(38,8) NULL       -- bank's commercial rate
    PRIMARY KEY (DT, SellCCY, BuyCCY)
);
```

### Your Task

```csharp
// CurrencyRate.cs
public class CurrencyRate
{
    // TODO: Properties
    // - Date (DateTime): get-only
    // - SellCurrency (string): get-only
    // - BuyCurrency (string): get-only
    // - Rate (decimal?): get/set, nullable (NBG official rate)
    // - CommercialRate (decimal?): get/set, nullable (bank's commercial rate)
    // - Spread (decimal?): COMPUTED — if both Rate and CommercialRate have values,
    //   return CommercialRate - Rate; otherwise null

    // TODO: Constructor(DateTime date, string sellCcy, string buyCcy)
    //   - Sets Date, SellCurrency (uppercase), BuyCurrency (uppercase)
    //   - Rate and CommercialRate default to null

    // TODO: Constructor(DateTime date, string sellCcy, string buyCcy, decimal rate)
    //   - Chains to the first constructor, then sets Rate

    // TODO: Constructor(DateTime date, string sellCcy, string buyCcy, decimal rate, decimal commercialRate)
    //   - Chains to the second constructor, then sets CommercialRate

    // TODO: Method — decimal Convert(decimal amount, bool useCommercialRate = false)
    //   - If useCommercialRate: uses CommercialRate (throw if null)
    //   - Otherwise: uses Rate (throw if null)
    //   - Returns amount * selectedRate

    // TODO: Override ToString()
    //   - Returns: "2025-01-15 USD/GEL: 2.75000000" format
}
```

### Test Cases — Implement in `CurrencyRateTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Basic constructor sets Date | `new CurrencyRate(dt, "USD", "GEL").Date` | the given date |
| 2 | Currencies stored uppercase | `new CurrencyRate(dt, "usd", "gel").SellCurrency` | `"USD"` |
| 3 | Rate defaults to null | `new CurrencyRate(dt, "USD", "GEL").Rate` | `null` |
| 4 | Constructor with rate sets Rate | `new CurrencyRate(dt, "USD", "GEL", 2.75m).Rate` | `2.75m` |
| 5 | Full constructor sets both | `new CurrencyRate(dt, "USD", "GEL", 2.75m, 2.80m)` | Rate=2.75, Commercial=2.80 |
| 6 | Spread computed correctly | Rate=2.75, CommercialRate=2.80 | Spread = `0.05m` |
| 7 | Spread null when Rate is null | no Rate set | Spread = `null` |
| 8 | Spread null when Commercial is null | only Rate set | Spread = `null` |
| 9 | Convert using official rate | Rate=2.75, Convert(100, false) | `275m` |
| 10 | Convert using commercial rate | CommercialRate=2.80, Convert(100, true) | `280m` |
| 11 | Convert throws when rate is null | no Rate set, Convert(100, false) | throws |
| 12 | ToString format | Rate=2.75m, date=2025-01-15 | contains `"USD/GEL"` and `"2.75"` |

---

## Exercise 4: Bank Account — `hw.Accounts` as a Class

**Concepts:** Encapsulation (`private set`), methods that modify state, validation, computed properties

### The HSP Table (Simplified)

```sql
-- Source: hw.Accounts (simplified)
CREATE TABLE hw.Accounts (
    AccountID       BIGINT IDENTITY PRIMARY KEY,
    CustomerID      INT,
    AccountNumber   VARCHAR(30),
    CCY             VARCHAR(5)      DEFAULT 'GEL',
    AccountState    BIGINT          DEFAULT 0,
    Balance         DECIMAL(28,6)   DEFAULT 0,
    BlockedAmount   DECIMAL(28,6)   DEFAULT 0,
    CreateTime      DATETIME2       DEFAULT SYSDATETIME()
);

-- The available balance function
CREATE FUNCTION hp.fnGetAccountAvailableBalanceByAccID (@AccountID BIGINT)
RETURNS DECIMAL AS BEGIN
    DECLARE @Amount DECIMAL;
    SELECT @Amount = a.Balance - a.BlockedAmount FROM hw.Accounts a WHERE a.AccountID = @AccountID;
    RETURN @Amount;
END;
```

### Your Task

This is the main exercise of the session. Build a `BankAccount` class with full encapsulation:

```csharp
// BankAccount.cs
public class BankAccount
{
    // TODO: Properties
    // - AccountId (long): get-only, set in constructor
    // - CustomerId (int): get-only, set in constructor
    // - AccountNumber (string): get-only, set in constructor
    // - CCY (string): get-only, default "GEL"
    // - Balance (decimal): get + PRIVATE set, default 0
    //   (external code can read but NOT write directly)
    // - BlockedAmount (decimal): get + PRIVATE set, default 0
    // - AccountState (long): get + PRIVATE set, default 0
    // - CreateTime (DateTime): get-only, set to DateTime.Now in constructor

    // TODO: Computed property
    // - AvailableBalance (decimal): Balance - BlockedAmount
    //   (mirrors hp.fnGetAccountAvailableBalanceByAccID)

    // TODO: Constructor(long accountId, int customerId, string accountNumber, string ccy = "GEL")
    // - Validates: accountNumber must not be null/empty
    // - Validates: ccy must be exactly 3 chars
    // - Sets all initial values, Balance = 0, BlockedAmount = 0

    // TODO: Methods
    //
    // public void Deposit(decimal amount)
    //   - amount must be > 0 (throw ArgumentException if not)
    //   - adds amount to Balance
    //
    // public bool Withdraw(decimal amount)
    //   - amount must be > 0 (throw ArgumentException if not)
    //   - if amount > AvailableBalance, return false (insufficient funds)
    //   - otherwise subtract from Balance and return true
    //
    // public void BlockAmount(decimal amount)
    //   - amount must be > 0
    //   - if amount > AvailableBalance, throw InvalidOperationException
    //   - adds amount to BlockedAmount
    //
    // public void UnblockAmount(decimal amount)
    //   - amount must be > 0
    //   - if amount > BlockedAmount, throw InvalidOperationException
    //   - subtracts amount from BlockedAmount
    //
    // public void Activate()   → sets AccountState = 1
    // public void Deactivate() → sets AccountState = 0
    // public bool IsActive     → computed: AccountState == 1

    // TODO: Override ToString()
    //   - Returns: "[AC01GEL00000000012345] 1,000.00 GEL (Available: 800.00)" format
}
```

### Test Cases — Implement in `BankAccountTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | New account has zero balance | `new BankAccount(1, 42, "AC01GEL00000000000001").Balance` | `0m` |
| 2 | New account has zero blocked | same account `.BlockedAmount` | `0m` |
| 3 | AvailableBalance = Balance - Blocked | deposit 1000, block 200 | AvailableBalance = `800m` |
| 4 | Deposit increases balance | deposit 500, deposit 300 | Balance = `800m` |
| 5 | Deposit with zero throws | `account.Deposit(0)` | throws ArgumentException |
| 6 | Deposit with negative throws | `account.Deposit(-100)` | throws ArgumentException |
| 7 | Withdraw succeeds when funds available | deposit 1000, withdraw 400 | returns `true`, Balance = `600m` |
| 8 | Withdraw fails when insufficient funds | deposit 100, withdraw 200 | returns `false`, Balance = `100m` (unchanged) |
| 9 | Withdraw respects blocked amount | deposit 1000, block 800, withdraw 300 | returns `false` (only 200 available) |
| 10 | Withdraw exactly available succeeds | deposit 1000, block 800, withdraw 200 | returns `true`, Balance = `800m` |
| 11 | BlockAmount increases BlockedAmount | deposit 1000, block 300 | BlockedAmount = `300m` |
| 12 | BlockAmount more than available throws | deposit 100, block 200 | throws InvalidOperationException |
| 13 | UnblockAmount decreases blocked | deposit 1000, block 300, unblock 100 | BlockedAmount = `200m` |
| 14 | UnblockAmount more than blocked throws | block 100, unblock 200 | throws InvalidOperationException |
| 15 | Activate sets state to 1 | `account.Activate()` | IsActive = `true` |
| 16 | Deactivate sets state to 0 | activate then deactivate | IsActive = `false` |
| 17 | Default CCY is GEL | `new BankAccount(1, 42, "ACC")` | CCY = `"GEL"` |
| 18 | Custom CCY is stored | `new BankAccount(1, 42, "ACC", "USD")` | CCY = `"USD"` |
| 19 | Empty account number throws | `new BankAccount(1, 42, "")` | throws ArgumentException |
| 20 | CreateTime is set automatically | new account | CreateTime is close to DateTime.Now |

---

## Exercise 5: Account Number Service — Refactor Session 3 Into a Static Class

**Concepts:** `static class`, organizing related methods, reusing Session 3 code

### Your Task

Take the `GenerateAccountNumber` and `ParseAccountNumber` methods from Session 3 and put them into a proper static class:

```csharp
// AccountNumberService.cs
public static class AccountNumberService
{
    private const int AccountNumberLength = 20;

    // TODO: Move GenerateAccountNumber from Session 3 here
    // public static string? Generate(
    //     int accountId,
    //     string ccy = "UPG",
    //     int type = 1,
    //     int usageType = 0,
    //     string accountInitial = "C",
    //     string schemaTypeChar = "A")

    // TODO: Move ParseAccountNumber from Session 3 here
    // public static (string SchemaTypeChar, int Type, string CCY, int UsageType, long AccountID)
    //     Parse(string accountNumber)

    // TODO: NEW — Add a Validate method
    // public static bool Validate(string accountNumber)
    //   Returns true if:
    //   - accountNumber is not null/empty
    //   - accountNumber.Length == 20
    //   - First char is 'A' or 'B' (valid schema types)
    //   - Parse(accountNumber) doesn't throw (the parts are valid)
    //
    //   Returns false otherwise (use try/catch around Parse)

    // TODO: Private constant
    // private const int AccountNumberLength = 20;
}
```

### Test Cases — Implement in `AccountNumberServiceTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Generate produces 20-char string | `AccountNumberService.Generate(1)` | length = 20 |
| 2 | Parse extracts AccountID correctly | Generate(12345), then Parse | AccountID = `12345` |
| 3 | Round-trip preserves all fields | Generate with custom params, then Parse | all fields match |
| 4 | Validate returns true for valid | Generate(1), then Validate | `true` |
| 5 | Validate returns false for null | `Validate(null)` | `false` |
| 6 | Validate returns false for empty | `Validate("")` | `false` |
| 7 | Validate returns false for wrong length | `Validate("SHORT")` | `false` |
| 8 | Validate returns false for invalid schema | `Validate("X" + "0".PadRight(19))` | `false` |
| 9 | Static class — methods called on class name | `AccountNumberService.Generate(...)` | compiles and works |
| 10 | Private constant not accessible | `AccountNumberService.AccountNumberLength` | should NOT compile (it's private) |

---

## Exercise 6 (Challenge): Currency Converter — `hp.fnCurrencyCalcAmount` as a Static Class

**Concepts:** `static class`, methods that use other classes (CurrencyRate), encapsulation of business logic

### The HSP Function

```sql
-- Source: hp.fnCurrencyCalcAmount (simplified logic)
-- When selling GEL: amount / rate
-- When selling foreign currency: amount * rate
-- When same currency: return amount as-is
CREATE FUNCTION hp.fnCurrencyCalcAmount(
    @CCY VARCHAR(100),
    @BuyCCY VARCHAR(100),
    @IsComercialRate INT,
    @Amount DECIMAL,
    @DateTime DATETIME
) RETURNS DECIMAL AS BEGIN
    IF @CCY = @BuyCCY RETURN @Amount;
    -- lookup rate from CCYRates table, then multiply or divide
    ...
END;
```

### Your Task

Since we don't have database access, we pass the rate explicitly:

```csharp
// CurrencyConverter.cs
public static class CurrencyConverter
{
    // TODO: public static decimal Convert(
    //     decimal amount,
    //     string fromCcy,
    //     string toCcy,
    //     CurrencyRate rate,
    //     bool useCommercialRate = false)
    //
    // Logic:
    //   1. If fromCcy == toCcy, return amount (no conversion needed)
    //   2. Pick the rate value: useCommercialRate ? rate.CommercialRate : rate.Rate
    //      (throw if the chosen rate is null)
    //   3. Determine direction:
    //      - If fromCcy == "GEL": result = amount / rateValue (selling GEL, buying foreign)
    //      - If toCcy == "GEL":   result = amount * rateValue (selling foreign, buying GEL)
    //      - Otherwise: throw NotSupportedException("Cross-currency conversion not supported")
    //   4. Round to 4 decimal places: Math.Round(result, 4)
    //   5. Return the result

    // TODO: private helper
    // private static decimal GetRateValue(CurrencyRate rate, bool useCommercial)
    //   Returns the appropriate rate or throws InvalidOperationException if null
}
```

### Test Cases — Implement in `CurrencyConverterTests.cs`

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Same currency returns amount | Convert(100, "GEL", "GEL", rate) | `100m` |
| 2 | USD to GEL (multiply) | amount=100, fromCcy="USD", rate=2.75 | `275m` |
| 3 | GEL to USD (divide) | amount=275, fromCcy="GEL", rate=2.75 | `100m` |
| 4 | EUR to GEL (multiply) | amount=50, fromCcy="EUR", rate=3.05 | `152.50m` |
| 5 | GEL to EUR (divide) | amount=305, fromCcy="GEL", toCcy="EUR", rate=3.05 | `100m` |
| 6 | Uses commercial rate when flag set | rate=2.75, commercial=2.80, useCommercial=true | uses 2.80 |
| 7 | Null rate throws | Rate=null, Convert(...) | throws |
| 8 | Cross-currency throws | Convert(100, "USD", "EUR", rate) | throws NotSupportedException |
| 9 | Result rounded to 4 decimals | Convert(100, "GEL", "USD", rate=2.73) | `Math.Round(100/2.73, 4)` |
| 10 | Zero amount returns zero | Convert(0, "USD", "GEL", rate=2.75) | `0m` |

---

## Bonus Exercise: Transfer Service — Composing Multiple Classes

**Concepts:** Using multiple classes together, method that orchestrates objects

### Your Task

Create a static class that transfers money between two `BankAccount` objects:

```csharp
// TransferService.cs
public static class TransferService
{
    // TODO: public static bool Transfer(
    //     BankAccount from,
    //     BankAccount to,
    //     decimal amount)
    //
    // Same-currency transfer:
    //   1. Validate: from and to must not be null
    //   2. Validate: from.CCY must equal to.CCY (for same-currency transfer)
    //   3. Try to withdraw from source (use from.Withdraw)
    //   4. If withdrawal fails, return false
    //   5. Deposit into destination (use to.Deposit)
    //   6. Return true

    // TODO: public static bool Transfer(
    //     BankAccount from,
    //     BankAccount to,
    //     decimal amount,
    //     CurrencyRate rate,
    //     bool useCommercialRate = false)
    //
    // Cross-currency transfer:
    //   1. Same null checks
    //   2. If same CCY, delegate to the simpler Transfer overload
    //   3. Convert amount using CurrencyConverter.Convert(amount, from.CCY, to.CCY, rate, useCommercialRate)
    //   4. Withdraw original amount from source
    //   5. Deposit converted amount into destination
    //   6. Return true (or false if withdrawal fails)
}
```

### Test Cases — Implement in `TransferServiceTests.cs`

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Same-currency transfer | from: 1000 GEL, to: 500 GEL, transfer 200 | from=800, to=700, returns true |
| 2 | Insufficient funds | from: 100 GEL, to: 500 GEL, transfer 200 | returns false, balances unchanged |
| 3 | Null source throws | from=null | throws |
| 4 | Null destination throws | to=null | throws |
| 5 | Different CCY without rate throws | from: GEL, to: USD, no rate overload | throws (CCY mismatch) |
| 6 | Cross-currency transfer | from: 1000 GEL, to: 0 USD, rate=2.75, transfer 275 GEL | from=725, to=100 USD |
| 7 | Transfer uses method overloading | same CCY calls simple, different calls complex | both work |

---

## Quick Reference: SQL Table → C# Class Mapping

| SQL Table Feature | C# Class Equivalent |
|---|---|
| `CREATE TABLE` | `public class ClassName { }` |
| Column definition | Property: `public Type Name { get; set; }` |
| `NOT NULL` | Required constructor parameter |
| `DEFAULT value` | Property initializer: `= value` |
| `IDENTITY` | `{ get; }` set only in constructor |
| Computed column (`AS expr`) | Computed property: `=> expression` |
| `CHECK` constraint | Validation in constructor or setter |
| `GRANT SELECT` / `DENY UPDATE` | `{ get; private set; }` |
| Stored procedure on the table | Method inside the class |
| Scalar function (standalone) | `static` method in a `static class` |
| Schema (`hp.`, `hw.`) | `namespace Banking.Core;` |
| `INSERT INTO ... VALUES (...)` | `new ClassName { Prop = value }` |
| A row in the table | An object (instance) of the class |
| `SELECT ... FROM table` → result set | `List<ClassName>` |
