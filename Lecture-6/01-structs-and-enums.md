# Structs and Enums in C# — A SQL Developer's Guide

## Structs: Value-Type Alternatives to Classes

In Session 2 you learned about value types vs reference types. `int`, `decimal`, `bool`, `DateTime` are all **value types** — they live on the stack, are copied when assigned, and have no identity of their own.

A **struct** lets you create your own value type — perfect for small, immutable pieces of data that represent a single coherent value.

| SQL Concept | C# Struct |
|---|---|
| A scalar value like `DECIMAL(28,6)` | A struct wrapping related scalars |
| `DECIMAL + VARCHAR` for a monetary amount | `Money(decimal Amount, string Currency)` |
| Inline table-valued function returning 1 row | A method returning a struct |
| `DEFAULT` constraint with no nulls allowed | Value types can't be null by default |

> **Key analogy:** Think of a `Money` struct the way you think of a composite domain value in SQL — like `DECIMAL(28,6)` for `Balance` paired with `VARCHAR(5)` for `CCY`. They belong together, they represent one concept, and you always want them copied — never shared by reference.

---

## Defining a Struct

```csharp
// A Money value type — copied on assignment, never null, small
public struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new ArgumentException("Amount cannot be negative.", nameof(amount));
        if (string.IsNullOrEmpty(currency) || currency.Length != 3)
            throw new ArgumentException("Currency must be a 3-character code.", nameof(currency));

        Amount = amount;
        Currency = currency.ToUpper();
    }

    public override string ToString() => $"{Amount:N2} {Currency}";
}
```

```csharp
Money a = new Money(1000m, "GEL");
Money b = a;             // b is a COPY of a — they are independent
b = new Money(500m, "GEL");

Console.WriteLine(a);   // 1,000.00 GEL — unchanged
Console.WriteLine(b);   // 500.00 GEL
```

Compare this with a class:

```csharp
// If Money were a class, b and a would point to the same object
Money a = new Money(1000m, "GEL");
Money b = a;             // b REFERENCES the same object as a
```

---

## Struct vs Class — The Key Differences

| Feature | `struct` | `class` |
|---|---|---|
| Memory | Stack (usually) | Heap |
| Assignment | Copies the value | Copies the reference |
| Default value | `default` (zeroed fields) | `null` |
| Can be null? | No (unless `Nullable<T>` / `T?`) | Yes |
| Inheritance | Cannot inherit | Can inherit |
| Best for | Small, immutable, value-like data | Objects with identity and state |

---

## When to Use a Struct

Use a struct when the type:

1. **Represents a single value** — `Money`, `DateRange`, `Coordinates`, `AccountNumber`
2. **Is small** — rule of thumb: 16 bytes or less (2–4 fields of primitive types)
3. **Should be copied, not shared** — each holder gets its own copy
4. **Is immutable** — no one should change it after creation

**Don't** use a struct when:
- The type has identity (two accounts with the same balance are *not* the same account)
- The type is large (copying gets expensive)
- You need inheritance

```csharp
// Good struct candidates in banking:
public struct Money { decimal Amount; string Currency; }
public struct DateRange { DateTime From; DateTime To; }
public struct AccountNumber { string Value; }

// Bad struct candidates:
// BankAccount — has identity, has state, has many fields
// Transaction — has complex behavior, needs inheritance
```

---

## Readonly Structs

Mark a struct `readonly` to guarantee immutability — the compiler prevents any method from modifying fields:

```csharp
public readonly struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency.ToUpper();
    }

    // Arithmetic operations return NEW Money values — immutable!
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies.");
        return new Money(Amount + other.Amount, Currency);
    }

    public Money Subtract(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot subtract different currencies.");
        return new Money(Amount - other.Amount, Currency);
    }

    public override string ToString() => $"{Amount:N2} {Currency}";
}
```

```csharp
var balance = new Money(1000m, "GEL");
var fee     = new Money(5m, "GEL");
var net     = balance.Subtract(fee);   // new Money(995m, "GEL")

Console.WriteLine(balance);  // 1,000.00 GEL — unchanged
Console.WriteLine(net);      // 995.00 GEL
```

---

## Enums: Lookup Tables as a Type

An **enum** (enumeration) is a named set of integer constants. In SQL you probably have a lookup/reference table or a hardcoded `INT` column where 1 means Pending, 2 means Completed, etc. Enums bring that concept into the type system.

| SQL Pattern | C# Enum |
|---|---|
| `AccountState BIGINT` with values 0=Active, 1=Frozen, 2=Closed | `enum AccountState { Active, Frozen, Closed }` |
| Hardcoded integers: `WHERE TransactionStatus = 1` | Named constants: `TransactionStatus.Completed` |
| Lookup table `dbo.TransactionTypes` | `enum TransactionType { Deposit, Withdrawal, Transfer, Fee }` |
| `CASE WHEN Status = 0 THEN 'Pending'` | `status.ToString()` → `"Pending"` |

> **Key analogy:** If you have a column like `hw.Accounts.AccountState BIGINT` where 0 = Active, 1 = Frozen, 2 = Closed, using a raw `long` in C# is dangerous — magic numbers everywhere. An enum gives those numbers names, so the compiler catches typos and your code is self-documenting.

---

## Defining an Enum

```csharp
// Simple enum — values default to 0, 1, 2, ...
public enum TransactionStatus
{
    Pending,     // 0
    Completed,   // 1
    Failed,      // 2
    Cancelled    // 3
}
```

```csharp
// Explicit underlying values (matching the DB integers)
public enum AccountState : long
{
    Active   = 0,
    Frozen   = 1,
    Closed   = 2,
    Blocked  = 3
}
```

```csharp
// Usage
TransactionStatus status = TransactionStatus.Pending;
Console.WriteLine(status);          // "Pending"
Console.WriteLine((int)status);     // 0

// Compare
if (status == TransactionStatus.Pending)
    Console.WriteLine("Transaction is awaiting processing.");

// Convert from int (e.g., value read from database)
int dbValue = 2;
TransactionStatus fromDb = (TransactionStatus)dbValue;
Console.WriteLine(fromDb);          // "Failed"
```

---

## Enums in Switch Expressions

Enums and `switch` are a natural pair — like a `CASE` expression in SQL:

```csharp
// SQL equivalent:
// CASE AccountState
//   WHEN 0 THEN 'Account is active'
//   WHEN 1 THEN 'Account is frozen — no withdrawals allowed'
//   WHEN 2 THEN 'Account is closed'
//   ELSE 'Unknown state'
// END

string GetStateDescription(AccountState state) => state switch
{
    AccountState.Active  => "Account is active",
    AccountState.Frozen  => "Account is frozen — no withdrawals allowed",
    AccountState.Closed  => "Account is closed",
    AccountState.Blocked => "Account is blocked by compliance",
    _                    => "Unknown state"
};
```

---

## Enum Parsing and Conversion

```csharp
// Convert enum to its underlying int (for storing in DB)
AccountState state = AccountState.Frozen;
long dbValue = (long)state;            // 1L

// Convert int back to enum (reading from DB)
long fromDb = 2L;
AccountState restored = (AccountState)fromDb;   // AccountState.Closed

// Parse from string (e.g., from config or API)
bool ok = Enum.TryParse<AccountState>("Frozen", out AccountState parsed);
// ok = true, parsed = AccountState.Frozen

// All values of an enum
foreach (AccountState s in Enum.GetValues<AccountState>())
    Console.WriteLine($"{(long)s}: {s}");
// 0: Active
// 1: Frozen
// 2: Closed
// 3: Blocked
```

---

## Flags Enums — Bitwise Combinations

When a column stores a **bitmask** (multiple flags OR'd together), use `[Flags]`:

```csharp
// SQL: AccountPermissions INT -- 1=CanDeposit, 2=CanWithdraw, 4=CanTransfer
// Bit 1+2 set = can deposit AND withdraw

[Flags]
public enum AccountPermissions
{
    None        = 0,
    CanDeposit  = 1,      // bit 0
    CanWithdraw = 2,      // bit 1
    CanTransfer = 4,      // bit 2
    CanAll      = CanDeposit | CanWithdraw | CanTransfer  // all bits
}
```

```csharp
AccountPermissions permissions = AccountPermissions.CanDeposit | AccountPermissions.CanWithdraw;
Console.WriteLine(permissions);         // "CanDeposit, CanWithdraw"
Console.WriteLine((int)permissions);    // 3

// Check if a specific flag is set
bool canWithdraw = permissions.HasFlag(AccountPermissions.CanWithdraw);  // true
bool canTransfer = permissions.HasFlag(AccountPermissions.CanTransfer);  // false
```

---

## Replacing Magic Numbers with Enums

This is the single most impactful use of enums in real banking code:

```csharp
// Before — magic numbers, fragile
public class Transaction
{
    public int Status { get; private set; } = 0;   // what is 0?

    public void MarkCompleted() => Status = 1;      // what is 1?
    public void MarkFailed()    => Status = 2;      // what is 2?

    public bool IsPending => Status == 0;           // same magic number in two places
}

// After — enum, self-documenting and type-safe
public class Transaction
{
    public TransactionStatus Status { get; private set; } = TransactionStatus.Pending;

    public void MarkCompleted() => Status = TransactionStatus.Completed;
    public void MarkFailed()    => Status = TransactionStatus.Failed;

    public bool IsPending => Status == TransactionStatus.Pending;
}
```

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| Composite domain value (amount + currency) | `readonly struct Money` |
| Columns that always travel together, copied into results | Struct (value type, copied) |
| Lookup table / hardcoded integer column | `enum` |
| `CASE WHEN Status = 0 THEN ...` | `switch` expression on enum |
| Integer bitmask permissions column | `[Flags] enum` |
| Converting stored integer back to meaning | `(MyEnum)dbValue` |
