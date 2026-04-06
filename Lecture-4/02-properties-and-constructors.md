# Properties and Constructors — Columns With Rules

## Fields vs Properties

In SQL, a column is just a column. In C#, there's a distinction between **fields** (raw data storage) and **properties** (controlled access to data).

### Fields — Raw Storage (Like a Column With No Constraints)

```csharp
public class CurrencyRate
{
    public DateTime Date;          // field — anyone can read/write directly
    public string SellCurrency;    // field — no validation, no control
    public string BuyCurrency;
    public decimal Rate;
}

// Anyone can set anything
var rate = new CurrencyRate();
rate.Rate = -999m;                // no validation — bad data gets in!
rate.SellCurrency = null;         // no protection
```

Fields are like columns with **no constraints** — raw, unprotected data.

### Properties — Controlled Access (Like Columns With Constraints)

```csharp
public class CurrencyRate
{
    public DateTime Date { get; set; }         // auto-property
    public string SellCurrency { get; set; }   // auto-property
    public string BuyCurrency { get; set; }
    public decimal Rate { get; set; }
}
```

This looks similar, but properties use `{ get; set; }` and give you the ability to add control later. **Always use properties, not fields, for data that other code accesses.**

> **SQL analogy:** A field is a column with no constraints. A property is a column with `CHECK`, `DEFAULT`, and computed column capabilities built in.

---

## Auto-Properties — The Standard

Auto-properties are the most common form. The compiler generates a hidden backing field:

```csharp
public class CurrencyCode
{
    // Auto-properties — the compiler generates the backing field for you
    public string Code { get; set; }        // "GEL", "USD", "EUR"
    public string NumericCode { get; set; } // "981", "840", "978"
    public int Scale { get; set; }          // usually 2 (cents)
}
```

This is based on the real `hw.CurrencyCodes3` table:

```sql
CREATE TABLE hw.CurrencyCodes3 (
    CurrencyCode   VARCHAR(5),
    CurrencyNumber VARCHAR(5),
    CurrencyScale  INT
);
```

---

## Properties With Default Values

```csharp
public class BankAccount
{
    public long AccountId { get; set; }
    public string CCY { get; set; } = "GEL";              // default value
    public decimal Balance { get; set; } = 0m;             // default value
    public decimal BlockedAmount { get; set; } = 0m;       // default value
    public long AccountState { get; set; } = 0;            // default value
    public DateTime CreateTime { get; set; } = DateTime.Now; // default value
}
```

This maps directly to SQL `DEFAULT` constraints:

```sql
CCY           VARCHAR(5)    DEFAULT 'GEL',
Balance       DECIMAL(28,6) DEFAULT 0,
BlockedAmount DECIMAL(28,6) DEFAULT 0,
AccountState  BIGINT        DEFAULT 0,
CreateTime    DATETIME2     DEFAULT SYSDATETIME()
```

---

## Read-Only Properties — Computed Columns

### `{ get; }` — Set Once, Read Forever

```csharp
public class CurrencyPair
{
    public int PairId { get; }             // can only be set in constructor
    public string BaseCurrency { get; }    // can only be set in constructor
    public string QuoteCurrency { get; }   // can only be set in constructor

    public CurrencyPair(int pairId, string baseCurrency, string quoteCurrency)
    {
        PairId = pairId;
        BaseCurrency = baseCurrency;
        QuoteCurrency = quoteCurrency;
    }
}
```

### `{ get; init; }` — Set During Initialization Only (C# 9+)

```csharp
public class CurrencyPair
{
    public int PairId { get; init; }
    public string BaseCurrency { get; init; }
    public string QuoteCurrency { get; init; }
}

// Can set during initialization
var pair = new CurrencyPair
{
    PairId = 1,
    BaseCurrency = "USD",
    QuoteCurrency = "GEL"
};

// Cannot change after
pair.BaseCurrency = "EUR";  // COMPILER ERROR! init-only property
```

> **SQL analogy:** `{ get; init; }` is like an `IDENTITY` column or a column on a table with `INSERT` allowed but `UPDATE` denied.

### Computed Properties — Like Computed Columns

The `hp.CurrencyPairs` table has a computed column:

```sql
-- SQL computed column
PairCode AS (BaseCurrency + '/' + QuoteCurrency),
NormalizedKey AS (UPPER(BaseCurrency) + '/' + UPPER(QuoteCurrency))
```

In C#, this becomes a **computed property** (get-only with a body):

```csharp
public class CurrencyPair
{
    public string BaseCurrency { get; set; }
    public string QuoteCurrency { get; set; }

    // Computed property — recalculated every time you access it
    public string PairCode => $"{BaseCurrency}/{QuoteCurrency}";
    public string NormalizedKey => $"{BaseCurrency.ToUpper()}/{QuoteCurrency.ToUpper()}";
}

var pair = new CurrencyPair { BaseCurrency = "usd", QuoteCurrency = "gel" };
Console.WriteLine(pair.PairCode);       // "usd/gel"
Console.WriteLine(pair.NormalizedKey);  // "USD/GEL"
```

---

## Properties With Validation — Like CHECK Constraints

You can add logic to `get` and `set` using a **backing field**:

```csharp
public class BankAccount
{
    private decimal _balance;    // private backing field

    public decimal Balance
    {
        get => _balance;
        set
        {
            if (value < 0)
                throw new ArgumentException("Balance cannot be negative.");
            _balance = value;
        }
    }
}
```

This is like a `CHECK` constraint:

```sql
ALTER TABLE hw.Accounts ADD CONSTRAINT CK_Balance CHECK (Balance >= 0);
```

The difference: in SQL, the constraint is on the table. In C#, the validation lives inside the class itself.

---

## Mixed Access — `private set`

Sometimes you want everyone to **read** but only the class itself to **write**:

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; } = 0m;  // read: public, write: private

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Deposit must be positive.");
        Balance += amount;      // the class itself can modify
    }
}

var account = new BankAccount();
account.Deposit(500m);           // OK — goes through the method
Console.WriteLine(account.Balance); // 500m — can read
account.Balance = 1000m;         // COMPILER ERROR — cannot set from outside
```

> **SQL analogy:** This is like a column you can `SELECT` but can only `UPDATE` through a stored procedure — controlled modification.

---

## Constructors — The INSERT Rules

A **constructor** is the code that runs when you create a new object. It's like the rules that apply during an `INSERT`:

### Parameterless Constructor (Default)

```csharp
public class CurrencyCode
{
    public string Code { get; set; } = "";
    public string NumericCode { get; set; } = "";
    public int Scale { get; set; } = 2;

    // This is the default constructor — it's implicit if you don't write any constructor
}

var gel = new CurrencyCode();  // Code = "", NumericCode = "", Scale = 2
```

### Parameterized Constructor — Enforcing Required Data

```csharp
public class CurrencyCode
{
    public string Code { get; }
    public string NumericCode { get; }
    public int Scale { get; }

    // Constructor — like a required INSERT with NOT NULL columns
    public CurrencyCode(string code, string numericCode, int scale)
    {
        Code = code;
        NumericCode = numericCode;
        Scale = scale;
    }
}

var gel = new CurrencyCode("GEL", "981", 2);   // must provide all three
var usd = new CurrencyCode("USD", "840", 2);
// new CurrencyCode();                           // ERROR — no parameterless constructor
```

> **SQL analogy:** A constructor with required parameters is like having `NOT NULL` on columns with no `DEFAULT` — you must provide values during INSERT.

### Constructor With Defaults

```csharp
public class BankAccount
{
    public long AccountId { get; }
    public string CCY { get; }
    public decimal Balance { get; private set; }

    // Mix of required and optional parameters
    public BankAccount(long accountId, string ccy = "GEL", decimal initialBalance = 0m)
    {
        AccountId = accountId;
        CCY = ccy;
        Balance = initialBalance;
    }
}

var a = new BankAccount(1);                    // CCY = "GEL", Balance = 0
var b = new BankAccount(2, "USD");             // Balance = 0
var c = new BankAccount(3, "EUR", 1000m);      // all specified
```

### Constructor Validation

```csharp
public class CurrencyPair
{
    public string BaseCurrency { get; }
    public string QuoteCurrency { get; }
    public bool IsActive { get; set; }
    public DateTime CreateDate { get; }

    public CurrencyPair(string baseCurrency, string quoteCurrency)
    {
        // Validation — like CHECK constraints at INSERT time
        if (string.IsNullOrWhiteSpace(baseCurrency))
            throw new ArgumentException("Base currency is required.");
        if (string.IsNullOrWhiteSpace(quoteCurrency))
            throw new ArgumentException("Quote currency is required.");
        if (baseCurrency.Length != 3 || quoteCurrency.Length != 3)
            throw new ArgumentException("Currency codes must be 3 characters.");
        if (baseCurrency == quoteCurrency)
            throw new ArgumentException("Base and quote currency must differ.");

        BaseCurrency = baseCurrency.ToUpper();
        QuoteCurrency = quoteCurrency.ToUpper();
        IsActive = true;                    // default
        CreateDate = DateTime.Now;          // like DEFAULT GETDATE()
    }

    public string PairCode => $"{BaseCurrency}/{QuoteCurrency}";
}
```

---

## Constructor Chaining — One Constructor Calls Another

When you have multiple constructors, they can call each other with `: this(...)`:

```csharp
public class BankAccount
{
    public long AccountId { get; }
    public int? CustomerId { get; }
    public string CCY { get; }
    public decimal Balance { get; private set; }

    // Full constructor
    public BankAccount(long accountId, int customerId, string ccy, decimal initialBalance)
    {
        AccountId = accountId;
        CustomerId = customerId;
        CCY = ccy;
        Balance = initialBalance;
    }

    // Simplified: default balance
    public BankAccount(long accountId, int customerId, string ccy)
        : this(accountId, customerId, ccy, 0m)   // chains to the full constructor
    {
    }

    // Minimal: default CCY and balance
    public BankAccount(long accountId, int customerId)
        : this(accountId, customerId, "GEL", 0m)
    {
    }
}
```

This is like having multiple `INSERT` patterns — one with all columns specified, others with defaults:

```sql
-- Full insert
INSERT INTO hw.Accounts (AccountID, CustomerID, CCY, Balance) VALUES (1, 42, 'USD', 1000);
-- Minimal insert (CCY and Balance use defaults)
INSERT INTO hw.Accounts (AccountID, CustomerID) VALUES (2, 42);
```

---

## Summary

| SQL Concept | C# Equivalent |
|---|---|
| Column | Property (`{ get; set; }`) |
| Column with `DEFAULT` | Property with initializer (`= value`) |
| Computed column | Computed property (`=> expression`) |
| `NOT NULL` constraint | Required constructor parameter |
| `CHECK` constraint | Validation in setter or constructor |
| `IDENTITY` column | `{ get; }` or `{ get; init; }` |
| `INSERT` with defaults | Constructor with optional parameters |
| Column with `GRANT SELECT` but `DENY UPDATE` | `{ get; private set; }` |
