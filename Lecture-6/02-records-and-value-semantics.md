# Records and Value Semantics in C#

## Value Semantics vs Reference Semantics

This distinction determines how equality and copying work — and it matters a lot when you design data transfer objects.

| Behavior | Reference Semantics (class) | Value Semantics (struct / record) |
|---|---|---|
| Equality | Are they the **same object in memory**? | Do they have the **same data**? |
| `==` | `true` only if same reference | `true` if all fields are equal |
| Copy | Copies the reference (pointer) | Copies the data |
| Mutation through copy | Both variables see the change | Changes to copy don't affect original |

```csharp
// Reference semantics — class
var a = new BankAccount { Balance = 1000m };
var b = a;               // b points to the same object
b.Balance = 999m;
Console.WriteLine(a.Balance);  // 999 — a and b are the same object!

// Value semantics — struct
var x = new Money(1000m, "GEL");
var y = x;               // y is a COPY
// y = new Money(999m, "GEL");  // would create new struct; x unchanged
Console.WriteLine(x.Amount);  // 1000 — unchanged
```

> **SQL analogy:** Reference semantics is like an alias for a table — `FROM hw.Accounts a` and `FROM hw.Accounts b` in the same query both point to the same table. Value semantics is like a result set — once you `SELECT INTO #temp`, you have your own independent copy.

---

## The Problem Records Solve

Imagine you need a **data transfer object** (DTO) to carry account data from a query result to a business layer. You want:
- **Immutability** — once created, nobody changes it
- **Value equality** — two DTOs with the same data should be "equal"
- **Easy creation** — minimal boilerplate

With a **class**, achieving this requires a lot of code:

```csharp
// Class-based DTO — lots of boilerplate for immutability + equality
public class AccountDto
{
    public long AccountId { get; }
    public string AccountNumber { get; }
    public decimal Balance { get; }
    public string CCY { get; }

    public AccountDto(long accountId, string accountNumber, decimal balance, string ccy)
    {
        AccountId = accountId;
        AccountNumber = accountNumber;
        Balance = balance;
        CCY = ccy;
    }

    // Must manually implement equality...
    public override bool Equals(object? obj)
        => obj is AccountDto other
           && AccountId == other.AccountId
           && AccountNumber == other.AccountNumber
           && Balance == other.Balance
           && CCY == other.CCY;

    public override int GetHashCode()
        => HashCode.Combine(AccountId, AccountNumber, Balance, CCY);
}
```

With a **record**, all of this is one line:

```csharp
// Record — immutable DTO with built-in value equality
public record AccountDto(long AccountId, string AccountNumber, decimal Balance, string CCY);
```

---

## Records: Immutable DTOs Made Simple

A `record` is a reference type (like a class) that uses **value-based equality** and generates immutable properties from its constructor parameters automatically.

```csharp
// Positional record — the compiler generates all properties, constructor,
// Equals, GetHashCode, and ToString automatically
public record AccountDto(long AccountId, string AccountNumber, decimal Balance, string CCY);
```

```csharp
var dto1 = new AccountDto(1, "ACC001", 5000m, "GEL");
var dto2 = new AccountDto(1, "ACC001", 5000m, "GEL");
var dto3 = new AccountDto(2, "ACC002", 3000m, "USD");

// Value equality — same data means equal
Console.WriteLine(dto1 == dto2);    // True
Console.WriteLine(dto1 == dto3);    // False

// Free ToString() — "AccountDto { AccountId = 1, AccountNumber = ACC001, ... }"
Console.WriteLine(dto1);

// Immutable — this does NOT compile:
// dto1.Balance = 9999m;   // ERROR: init-only
```

> **SQL analogy:** A record is like the result set of a `SELECT` query — it's a snapshot of data at a point in time, and two result sets with identical rows are "equal" in the sense that they contain the same data. Nobody expects you to modify a result set in-place.

---

## The `with` Expression — Non-Destructive Mutation

Records are immutable, but you often need a "copy with one field changed". The `with` expression creates a **new record** based on an existing one:

```csharp
public record AccountDto(long AccountId, string AccountNumber, decimal Balance, string CCY);

var original = new AccountDto(1, "ACC001", 5000m, "GEL");

// Create new record with Balance changed — original is untouched
var updated = original with { Balance = 5500m };

Console.WriteLine(original.Balance);  // 5000
Console.WriteLine(updated.Balance);   // 5500
Console.WriteLine(original == updated); // False — different data
```

```csharp
// More changes at once
var transferred = original with { Balance = 0m, AccountNumber = "ACC099" };
```

> **SQL analogy:** `with` is like writing `SELECT AccountId, AccountNumber, 5500 AS Balance, CCY FROM ...` — you get a new result with one column changed; the source table is unmodified.

---

## Records with Custom Members

Records aren't limited to positional syntax — you can add computed properties, methods, and validation:

```csharp
public record TransactionDto(
    long TransactionId,
    decimal Amount,
    string CCY,
    TransactionStatus Status,
    DateTime Timestamp)
{
    // Computed property
    public bool IsCompleted => Status == TransactionStatus.Completed;

    // Custom ToString override
    public override string ToString()
        => $"Tx#{TransactionId}: {Amount:N2} {CCY} [{Status}]";

    // Validation in a factory method
    public static TransactionDto Create(long id, decimal amount, string ccy)
    {
        if (amount <= 0) throw new ArgumentException("Amount must be positive.");
        if (ccy.Length != 3) throw new ArgumentException("CCY must be 3 chars.");
        return new TransactionDto(id, amount, ccy, TransactionStatus.Pending, DateTime.Now);
    }
}
```

---

## Record Inheritance

Records support inheritance. Useful when you have a base DTO and specific variants:

```csharp
// Base record
public record FinancialProductDto(long Id, int CustomerId, string CCY, decimal Balance);

// Derived records
public record DepositDto(long Id, int CustomerId, string CCY, decimal Balance,
                         decimal InterestRate, DateTime MaturityDate)
    : FinancialProductDto(Id, CustomerId, CCY, Balance);

public record LoanDto(long Id, int CustomerId, string CCY, decimal Balance,
                      decimal InterestRate, decimal RemainingDebt)
    : FinancialProductDto(Id, CustomerId, CCY, Balance);
```

```csharp
FinancialProductDto product = new DepositDto(1, 42, "GEL", 5000m, 0.12m, new DateTime(2026, 12, 31));

if (product is DepositDto deposit)
    Console.WriteLine($"Matures: {deposit.MaturityDate:d}");
```

---

## `record struct` — Value-Type Records

A `record struct` combines the immutability/equality features of records with the value-type semantics of structs. Use it for very small, frequently copied data:

```csharp
// record struct — value semantics + auto equality + auto ToString
public readonly record struct Money(decimal Amount, string Currency);

var a = new Money(1000m, "GEL");
var b = new Money(1000m, "GEL");

Console.WriteLine(a == b);   // True — value equality
Console.WriteLine(a);        // "Money { Amount = 1000, Currency = GEL }"

var c = a with { Amount = 500m };  // with works on record structs too
Console.WriteLine(a.Amount); // 1000 — unchanged
Console.WriteLine(c.Amount); // 500
```

---

## Choosing the Right Type

| Need | Type to Use |
|---|---|
| Mutable object with identity (a bank account that changes state) | `class` |
| Immutable DTO passed between layers | `record` |
| Small value-like data (Money, DateRange) with value equality | `readonly record struct` |
| Small custom value type, performance-critical, no auto-equality needed | `readonly struct` |
| Named set of integer constants (statuses, types) | `enum` |

```
                    Is it small (< 16 bytes)?
                    /                       \
                 Yes                         No
                  |                           |
    Does it need value equality?        Does it need value equality?
           /           \                      /           \
         Yes            No                  Yes            No
          |              |                   |              |
   record struct      struct              record           class
```

---

## Records vs Classes vs Structs — Quick Comparison

| Feature | `class` | `struct` | `record` | `record struct` |
|---|---|---|---|---|
| Type | Reference | Value | Reference | Value |
| Default equality | Reference | Value (field-by-field) | Value (auto-generated) | Value (auto-generated) |
| Immutable by default | No | No | Yes (init-only props) | Yes (`readonly`) |
| `with` expression | No | No | Yes | Yes |
| Auto `ToString` | No | Yes (basic) | Yes (formatted) | Yes (formatted) |
| Inheritance | Yes | No | Yes (record to record) | No |
| Can be null | Yes | No (use `T?`) | Yes | No (use `T?`) |

---

## Practical Pattern: Layered DTOs in Banking

```csharp
// Enums for type safety
public enum TransactionStatus { Pending, Completed, Failed, Cancelled }
public enum TransactionType   { Deposit, Withdrawal, Transfer, Fee }

// Value object — readonly record struct
public readonly record struct Money(decimal Amount, string Currency)
{
    public override string ToString() => $"{Amount:N2} {Currency}";
}

// DTO — record (reference type with value equality)
public record TransactionDto(
    long TransactionId,
    TransactionType Type,
    Money Amount,
    TransactionStatus Status,
    DateTime Timestamp);

// Usage
var tx = new TransactionDto(
    TransactionId: 1001,
    Type: TransactionType.Deposit,
    Amount: new Money(1000m, "GEL"),
    Status: TransactionStatus.Completed,
    Timestamp: DateTime.Now);

Console.WriteLine(tx);
// TransactionDto { TransactionId = 1001, Type = Deposit, Amount = 1,000.00 GEL, ... }

// "Modify" without mutating — create new record
var failed = tx with { Status = TransactionStatus.Failed };
```

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| Two result sets with identical rows | Two records with `==` returning `true` |
| Immutable result set (snapshot in time) | `record` — immutable by default |
| `SELECT ... AS NewColumn` (derived result) | `with` expression on a record |
| Small composite domain value | `readonly record struct` |
| DTO going between layers | `record` |
| Mutable entity representing a live DB row | `class` |
