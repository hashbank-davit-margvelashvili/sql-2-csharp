# Access Modifiers, Static Members, and Namespaces

## Access Modifiers — The GRANT/DENY of C#

In SQL, you control who can access what with `GRANT` and `DENY`. In C#, you control visibility with **access modifiers** on classes, properties, methods, and fields.

### The Four Main Access Modifiers

```csharp
public class BankAccount
{
    public  decimal Balance { get; private set; }   // anyone can read, only this class writes
    private decimal _overdraftLimit = 0m;            // only this class can see this
    protected string InternalCode { get; set; }     // this class + subclasses
    internal int AuditFlag { get; set; }            // only this project/assembly
}
```

| C# Modifier | SQL Analogy | Who Can Access |
|---|---|---|
| `public` | `GRANT EXECUTE TO PUBLIC` | Any code, anywhere |
| `private` | No `GRANT` at all (owner only) | Only inside the same class |
| `protected` | `GRANT` to schema + sub-schemas | The class itself + classes that inherit from it |
| `internal` | `GRANT` to database users only | Any code in the same project/assembly |

### Why This Matters — Encapsulation

**Encapsulation** means hiding internal details and exposing only what's necessary. Think of it like a stored procedure that hides the complex SQL behind a simple interface:

```sql
-- SQL: the caller doesn't see the internal logic
EXEC hp.spUMGetAccountBalance @AccountID = 42;
-- You don't know (or care) that internally it:
--   1. Joins hw.Accounts with hw.AccountStates
--   2. Checks blocked amounts
--   3. Applies currency conversion
```

In C#, the same idea:

```csharp
public class BankAccount
{
    // PUBLIC: the interface other code uses
    public decimal Balance { get; private set; }
    public decimal BlockedAmount { get; private set; }

    public decimal GetAvailableBalance() => Balance - BlockedAmount;

    public bool Withdraw(decimal amount)
    {
        if (!CanWithdraw(amount))      // calls private method
            return false;

        Balance -= amount;
        return true;
    }

    // PRIVATE: internal implementation — hidden from callers
    private bool CanWithdraw(decimal amount)
    {
        return amount > 0 && amount <= GetAvailableBalance();
    }
}

// From outside:
var account = new BankAccount(1, 42, "GEL", 1000m);
bool ok = account.Withdraw(200m);              // public — accessible
decimal avail = account.GetAvailableBalance(); // public — accessible
// bool check = account.CanWithdraw(500m);     // COMPILER ERROR — private!
```

The caller only sees `Withdraw()` and `GetAvailableBalance()`. The internal validation logic (`CanWithdraw`) is hidden — just like the implementation inside a stored procedure.

---

## Designing With Access Modifiers — The Practical Rule

```
                        ┌─────────────────────────┐
                        │  Start with PRIVATE     │
                        │  (hide everything)      │
                        └────────────┬────────────┘
                                     │
                            Need external access?
                                     │
                              ┌──────┴──────┐
                              │ YES         │ NO
                              ▼             ▼
                        Make PUBLIC     Keep PRIVATE
```

**Default to `private`**. Only make things `public` when another class genuinely needs to use them. This is the opposite of SQL where everything in a schema is visible by default.

```csharp
public class CurrencyConverter
{
    // PUBLIC — the method other code will call
    public decimal Convert(decimal amount, string fromCcy, string toCcy, decimal rate)
    {
        if (fromCcy == toCcy) return amount;
        return RoundToScale(amount * rate);
    }

    // PRIVATE — helper method, no one else needs to call this directly
    private decimal RoundToScale(decimal amount)
    {
        return Math.Floor(amount * 10000m) / 10000m;
    }
}
```

---

## Static Members — Functions That Don't Need a Row

In SQL, some functions don't operate on a specific row — they're utilities:

```sql
-- These don't need a table row, they're standalone
SELECT GETDATE();
SELECT dbo.iban_mod97('123456');
SELECT hp.fnCalcFeeAmount(1000, 5.00, 1.0, 0, 100);
```

In C#, the equivalent is **`static`** — members that belong to the class itself, not to any specific object:

### Static Methods

```csharp
public class IbanValidator
{
    // Static method — no object needed to call it
    public static int Mod97(string numericString)
    {
        int modValue = 0;
        foreach (char ch in numericString)
        {
            int digit = ch - '0';
            modValue = (modValue * 10 + digit) % 97;
        }
        return modValue;
    }
}

// Call directly on the class, no object needed:
int result = IbanValidator.Mod97("161400182100011234561014");

// You DON'T do this:
// var validator = new IbanValidator();
// validator.Mod97("...");  // wrong for static methods
```

> **SQL analogy:** A static method is like a scalar function — `dbo.iban_mod97(...)`. You call it by name, not through a table row.

### Static vs Instance — When to Use Each

| Static (class-level) | Instance (object-level) |
|---|---|
| `IbanValidator.Mod97(str)` | `account.GetAvailableBalance()` |
| Doesn't need object data | Needs the object's data (Balance, BlockedAmount) |
| Like a scalar function: `dbo.iban_mod97()` | Like a method on a row: `SELECT Balance - BlockedAmount FROM...` |
| Called on the **class name** | Called on an **object** |

### Static Properties

```csharp
public class AppConfig
{
    // Static property — one value shared by all code
    public static string DefaultCurrency { get; set; } = "GEL";
    public static string BankCode { get; set; } = "TB";
}

// Access without creating an object
string ccy = AppConfig.DefaultCurrency;  // "GEL"
```

> **SQL analogy:** Static properties are like values in a configuration table — `hp.AppConfigs` — global settings not tied to a specific row.

---

## Static Classes — Utility Containers

A **static class** cannot be instantiated. It's purely a container for static methods — like a schema that holds only functions:

```csharp
// Static class — can only contain static members
public static class FeeCalculator
{
    public static decimal CalcFeeAmount(
        decimal amount, decimal fixedFee, decimal percent, decimal min, decimal max)
    {
        decimal fee = fixedFee > 0 ? fixedFee : amount * percent / 100m;
        return Math.Max(min, Math.Min(max, fee));
    }

    public static decimal CalcVat(decimal amount) => amount * 0.18m;
}

// Usage — always through the class name
decimal fee = FeeCalculator.CalcFeeAmount(1000m, 0m, 2.0m, 1m, 50m);
decimal vat = FeeCalculator.CalcVat(fee);
```

> **SQL analogy:** A static class is like a schema full of scalar functions. `FeeCalculator.CalcFeeAmount(...)` is like `hp.fnCalcFeeAmount(...)`. The schema (`hp`) is the class; the function is the static method.

### When to use static classes vs regular classes

| Use a **static class** when | Use a **regular class** when |
|---|---|
| Pure calculations (no state) | The object holds data (properties) |
| Utility/helper methods | Behavior depends on the object's state |
| Constants and configuration | You need multiple instances |
| Example: `FeeCalculator`, `IbanValidator` | Example: `BankAccount`, `CurrencyPair` |

---

## Namespaces — Schema Qualification

In SQL, you organize objects into schemas:

```sql
hp.fnGenerateAccountNumber    -- hp schema
hw.Accounts                   -- hw schema
dbo.iban_mod97                -- dbo schema
```

In C#, you organize classes into **namespaces**:

```csharp
namespace Banking.Core
{
    public class BankAccount { ... }
}

namespace Banking.Currency
{
    public class CurrencyPair { ... }
    public class CurrencyRate { ... }
    public static class CurrencyConverter { ... }
}

namespace Banking.Validation
{
    public static class IbanValidator { ... }
}
```

### Using Namespaces

Without `using`, you need the full name (like schema-qualifying in SQL):

```csharp
// Without using — fully qualified (like hw.Accounts)
Banking.Core.BankAccount account = new Banking.Core.BankAccount();
Banking.Currency.CurrencyPair pair = new Banking.Currency.CurrencyPair("USD", "GEL");
```

With `using` — like setting a default schema:

```csharp
using Banking.Core;
using Banking.Currency;

// Now you can use short names
BankAccount account = new BankAccount();
CurrencyPair pair = new CurrencyPair("USD", "GEL");
```

### File-Scoped Namespaces (C# 10+)

Modern C# lets you declare the namespace for the entire file without nesting:

```csharp
// Old style (extra nesting)
namespace Banking.Core
{
    public class BankAccount
    {
        // ...
    }
}

// Modern style (file-scoped — no extra nesting)
namespace Banking.Core;

public class BankAccount
{
    // ...
}
```

---

## Partial Classes (Awareness)

A `partial` class lets you split a single class across multiple files:

```csharp
// File: BankAccount.cs
public partial class BankAccount
{
    public long AccountId { get; set; }
    public decimal Balance { get; private set; }
}

// File: BankAccount.Methods.cs
public partial class BankAccount
{
    public void Deposit(decimal amount) { Balance += amount; }
    public bool Withdraw(decimal amount) { ... }
}
```

The compiler merges them into one class. This is mainly used by code generators (like EF Core, WinForms). You'll encounter it but rarely write partial classes yourself.

---

## Putting It All Together

Here's a realistic example combining everything from this session — modeled on real HSP entities:

```csharp
namespace Banking.Core;

public class BankAccount
{
    // Properties with controlled access
    public long AccountId { get; }
    public int CustomerId { get; }
    public string AccountNumber { get; }
    public string CCY { get; }
    public decimal Balance { get; private set; }
    public decimal BlockedAmount { get; private set; }
    public DateTime CreateTime { get; }

    // Computed property
    public decimal AvailableBalance => Balance - BlockedAmount;

    // Constructor with validation
    public BankAccount(long accountId, int customerId, string accountNumber, string ccy = "GEL")
    {
        if (string.IsNullOrWhiteSpace(accountNumber))
            throw new ArgumentException("Account number is required.");

        AccountId = accountId;
        CustomerId = customerId;
        AccountNumber = accountNumber;
        CCY = ccy;
        Balance = 0m;
        BlockedAmount = 0m;
        CreateTime = DateTime.Now;
    }

    // Public methods — the interface
    public void Deposit(decimal amount)
    {
        ValidatePositiveAmount(amount);
        Balance += amount;
    }

    public bool Withdraw(decimal amount)
    {
        ValidatePositiveAmount(amount);
        if (amount > AvailableBalance)
            return false;

        Balance -= amount;
        return true;
    }

    public void BlockAmount(decimal amount)
    {
        ValidatePositiveAmount(amount);
        if (amount > AvailableBalance)
            throw new InvalidOperationException("Cannot block more than available balance.");
        BlockedAmount += amount;
    }

    // Private helper — hidden implementation detail
    private void ValidatePositiveAmount(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive.");
    }

    // Override ToString for readable output
    public override string ToString()
        => $"[{AccountNumber}] {Balance:N2} {CCY} (Available: {AvailableBalance:N2})";
}
```

---

## Summary

| SQL Concept | C# Equivalent |
|---|---|
| `GRANT EXECUTE TO PUBLIC` | `public` |
| No GRANT (owner only) | `private` |
| `GRANT` to schema family | `protected` |
| `GRANT` to database only | `internal` |
| Scalar function (standalone) | `static` method |
| Schema of utility functions | `static class` |
| Schema name (`hp.`, `dbo.`) | Namespace (`Banking.Core.`) |
| `USE` / default schema | `using` statement |
| Split a procedure across files | `partial class` |
