# Tuples, Nullable Types, and Anonymous Types in C#

## Tuples: Returning Multiple Values

In SQL, a query naturally returns multiple columns. In C#, a method can only return one thing — but with **tuples**, that one thing can bundle several values together without creating a named type.

| SQL Pattern | C# Tuple |
|---|---|
| `SELECT AccountId, Balance, CCY FROM ...` | `(long AccountId, decimal Balance, string CCY)` |
| `EXEC sp_GetSummary @Id` returning 3 OUTPUT params | Method returning `(bool Success, decimal Total, int Count)` |
| Quick ad-hoc multi-column result | Named tuple — no need to create a class |

> **Key analogy:** A tuple is like a `SELECT` statement that returns a single row with multiple columns — you're bundling values together for a specific purpose without committing to a full type definition (no `CREATE TABLE`, no `CREATE TYPE`).

---

## Basic Tuple Syntax

```csharp
// Return a tuple from a method
public (bool Success, decimal Fee, string Message) ProcessTransaction(decimal amount)
{
    if (amount <= 0)
        return (false, 0m, "Amount must be positive.");

    decimal fee = amount * 0.01m;
    return (true, fee, $"Processed {amount:N2}, fee: {fee:N2}");
}
```

```csharp
// Call it and destructure
var result = ProcessTransaction(1000m);
Console.WriteLine(result.Success);   // True
Console.WriteLine(result.Fee);       // 10.00
Console.WriteLine(result.Message);   // "Processed 1,000.00, fee: 10.00"

// Destructure directly into separate variables
var (success, fee, message) = ProcessTransaction(500m);
Console.WriteLine($"Success: {success}, Fee: {fee}");

// Discard values you don't need with _
var (ok, _, msg) = ProcessTransaction(250m);
```

---

## Named vs Unnamed Tuples

```csharp
// Named tuple — preferred, self-documenting
(long AccountId, decimal Balance, string CCY) GetBalance(string accountNumber)
    => (1L, 5000m, "GEL");

// Unnamed (positional) tuple — less readable
(long, decimal, string) GetBalanceUnnamed(string accountNumber)
    => (1L, 5000m, "GEL");
```

```csharp
var named   = GetBalance("ACC001");
Console.WriteLine(named.AccountId);  // 1
Console.WriteLine(named.Balance);    // 5000
Console.WriteLine(named.CCY);        // GEL

var unnamed = GetBalanceUnnamed("ACC001");
Console.WriteLine(unnamed.Item1);    // 1  — less readable
Console.WriteLine(unnamed.Item2);    // 5000
```

Always use **named tuples** when the tuple leaves the method scope. Reserve unnamed tuples for local, throwaway unpacking.

---

## Tuples in LINQ and Local Variables

Tuples shine for local grouping and quick multi-value assignments:

```csharp
// Quick swap (no temp variable needed)
(int a, int b) = (10, 20);
(a, b) = (b, a);          // swap
Console.WriteLine($"a={a}, b={b}");  // a=20, b=10
```

```csharp
// Grouping related values locally without creating a class
var accounts = new[]
{
    ("ACC001", 5000m, "GEL"),
    ("ACC002", 3000m, "USD"),
    ("ACC003", 1500m, "EUR"),
};

foreach (var (number, balance, ccy) in accounts)
    Console.WriteLine($"{number}: {balance:N2} {ccy}");
```

---

## When Tuples vs Records vs Classes

| Use Case | Best Choice |
|---|---|
| Internal method — quick multi-value return | Tuple `(bool Success, decimal Fee)` |
| DTO crossing a layer boundary (API, service) | `record` |
| Frequently instantiated, value-like | `readonly record struct` |
| Complex object with behavior | `class` |

A tuple is **private** by design — use it inside a method or as a return type for private/internal methods. If the tuple starts appearing in many places, extract it into a `record`.

---

## Nullable Value Types: `int?`

Value types (`int`, `decimal`, `bool`, `DateTime`, `struct`) can't normally be `null`. But database columns can be NULL. The **nullable value type** `T?` bridges that gap.

| SQL | C# |
|---|---|
| `INT NULL` | `int?` |
| `DECIMAL(28,6) NULL` | `decimal?` |
| `DATETIME2 NULL` | `DateTime?` |
| `IS NULL` check | `value == null` or `value.HasValue == false` |
| `ISNULL(col, 0)` | `value ?? 0` |
| `COALESCE(a, b, c)` | `a ?? b ?? c` |

```csharp
// Database column: MaturityDate DATETIME2 NULL
DateTime? maturityDate = null;         // no maturity date set yet
DateTime? maturityDate2 = new DateTime(2026, 12, 31);   // set

// Check for null
if (maturityDate.HasValue)
    Console.WriteLine(maturityDate.Value);  // safe access via .Value
else
    Console.WriteLine("No maturity date.");

// Shorter null check
if (maturityDate != null)
    Console.WriteLine(maturityDate.Value);

// Null-coalescing: provide a default if null (like ISNULL / COALESCE)
DateTime effectiveDate = maturityDate ?? DateTime.MaxValue;
Console.WriteLine(effectiveDate);   // DateTime.MaxValue
```

---

## The `??` and `?.` Operators

These are your null-safety tools — equivalent to `ISNULL`/`COALESCE` and conditional access:

```csharp
// ?? — null-coalescing: "use left side unless it's null, then use right side"
decimal? fee = null;
decimal actualFee = fee ?? 0m;           // 0m (like ISNULL(fee, 0))

string? name = null;
string displayName = name ?? "Unknown";  // "Unknown"

// Chaining ?? (like COALESCE)
string? first  = null;
string? second = null;
string  third  = "fallback";
string result = first ?? second ?? third;   // "fallback"
```

```csharp
// ?. — null-conditional: "access member only if not null"
BankAccount? account = GetAccount(999);   // might return null

// Without ?. — you'd need an if-null check
string? number = account?.AccountNumber;  // null if account is null

// Chaining ?.
decimal? balance = account?.Balance;

// Combining with ??
decimal safeBalance = account?.Balance ?? 0m;
Console.WriteLine($"Balance: {safeBalance:N2}");
```

> **SQL analogy:** `??` is `ISNULL(expression, default_value)`. `?.` is like wrapping a column reference in `CASE WHEN Account IS NULL THEN NULL ELSE Account.Balance END` — you only access the inner value when the outer is not null.

---

## Nullable Reference Types: `string?` and `#nullable enable`

In C#, reference types (`string`, `BankAccount`, `List<T>`) can normally be assigned `null` without any warning. Since C# 8, you can enable **nullable reference type** checking to get **compile-time null safety**:

```csharp
#nullable enable   // turn on null analysis for this file (or project-wide in .csproj)

// Without ? — the compiler assumes this is NEVER null
string accountNumber = "ACC001";
accountNumber = null;   // WARNING: cannot assign null to non-nullable string

// With ? — explicitly says "this might be null"
string? optionalNote = null;   // OK — we said it can be null

// The compiler now forces you to check before using
void PrintNote(string? note)
{
    // Console.WriteLine(note.Length);  // WARNING: possible null dereference
    if (note != null)
        Console.WriteLine(note.Length);   // safe
    Console.WriteLine(note?.Length ?? 0); // also safe
}
```

```csharp
// Method that might return null — annotate with ?
BankAccount? FindAccount(string number)
{
    // Returns null if not found
    return null;
}

// Caller knows it can be null
BankAccount? account = FindAccount("ACC999");
decimal balance = account?.Balance ?? 0m;   // safe
```

---

## Enabling Nullable in Your Project

Add this to your `.csproj` to enable nullable analysis project-wide:

```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

Or add `#nullable enable` at the top of individual files.

Once enabled, the compiler gives warnings when you:
- Dereference a nullable reference without a null check
- Assign `null` to a non-nullable reference
- Return `null` from a method that isn't annotated `?`

This is one of the best ways to prevent `NullReferenceException` at compile time instead of runtime.

---

## Null Patterns — Reference Guide

```csharp
// Pattern: check and use
if (account != null)
    Console.WriteLine(account.Balance);

// Pattern: pattern matching (modern style)
if (account is not null)
    Console.WriteLine(account.Balance);

// Pattern: null-conditional for optional access
decimal? bal = account?.Balance;

// Pattern: null-coalescing for defaults
decimal balance = account?.Balance ?? 0m;

// Pattern: throw if null (guard clause)
ArgumentNullException.ThrowIfNull(account);  // throws if null, continues if not

// Pattern: null-coalescing assignment (assign if currently null)
account ??= new BankAccount();   // only assigns if account is null
```

---

## Anonymous Types: Ad-Hoc Projections

An **anonymous type** lets you create a quick object with named properties without defining a class. You've seen this pattern in SQL's SELECT — you project into a shape that exists only for this query.

```csharp
// SQL:
// SELECT AccountNumber, Balance, CCY FROM hw.Accounts WHERE CustomerID = 42

// C# anonymous type — shape defined inline
var summary = new { AccountNumber = "ACC001", Balance = 5000m, CCY = "GEL" };

Console.WriteLine(summary.AccountNumber);  // "ACC001"
Console.WriteLine(summary.Balance);        // 5000
Console.WriteLine(summary);
// { AccountNumber = ACC001, Balance = 5000, CCY = GEL }
```

> **SQL analogy:** An anonymous type is like a `SELECT` without `INTO` — you're projecting data into a temporary shape that only exists in the context of the query. You didn't create a permanent table; you just needed that shape for this moment.

---

## Anonymous Types in LINQ

Anonymous types are most useful in LINQ projections:

```csharp
var accounts = new List<BankAccount> { /* ... */ };

// Project to an anonymous type — like a SELECT with specific columns
var summaries = accounts
    .Where(a => a.Balance > 0)
    .Select(a => new { a.AccountNumber, a.Balance, a.CCY });

foreach (var s in summaries)
    Console.WriteLine($"{s.AccountNumber}: {s.Balance:N2} {s.CCY}");
```

**Limitation:** Anonymous types can't be returned from methods or stored in typed fields — they live within the method where they're created. When you need to cross a method boundary, use a `record` or `class` instead.

---

## Anonymous Types vs Records vs Tuples

| Feature | Anonymous Type | Tuple | Record |
|---|---|---|---|
| Defines named properties | Yes | Yes (if named) | Yes |
| Can cross method boundary | No | Yes | Yes |
| Value equality | Yes | Yes | Yes |
| `ToString()` | Yes (formatted) | Yes | Yes |
| Can be used as generic type arg | No | Yes | Yes |
| Best for | Local LINQ projections | Quick multi-return | DTOs between layers |

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| Multi-column single-row result | Named tuple `(long Id, decimal Balance, string CCY)` |
| `INT NULL` column | `int?` — nullable value type |
| `ISNULL(col, default)` | `col ?? default` |
| `COALESCE(a, b, c)` | `a ?? b ?? c` |
| `WHERE col IS NOT NULL` guard | `ArgumentNullException.ThrowIfNull(col)` |
| `SELECT col1, col2` without INTO | Anonymous type `new { col1, col2 }` |
| Compile-time NULL contract (`NOT NULL` column) | `#nullable enable` — `string` can't be null |
| Optional column (`NULL` allowed) | `string?` — explicitly nullable |
