# Control Flow in C# — A SQL Developer's Guide

## You Already Know This (Sort Of)

T-SQL has control flow: `IF...ELSE`, `WHILE`, `CASE`. C# has the same concepts but with more power and flexibility. Every T-SQL construct maps directly to a C# equivalent — and C# gives you additional tools T-SQL doesn't have.

---

## IF / ELSE — The Same Logic, Different Syntax

### T-SQL
```sql
IF @amount > 1000
BEGIN
    SET @fee = @amount * 0.01;
END
ELSE IF @amount > 100
BEGIN
    SET @fee = @amount * 0.02;
END
ELSE
BEGIN
    SET @fee = 5.00;
END
```

### C#
```csharp
if (amount > 1000)
{
    fee = amount * 0.01m;
}
else if (amount > 100)
{
    fee = amount * 0.02m;
}
else
{
    fee = 5.00m;
}
```

### Key Differences

| T-SQL | C# |
|---|---|
| `IF @amount > 1000` | `if (amount > 1000)` |
| `BEGIN...END` | `{ }` (curly braces) |
| `ELSE IF` | `else if` |
| No `@` prefix on variables | No `@` prefix |
| `SET @fee = value` | `fee = value;` |

> **Rule:** In C#, the condition must be in parentheses `()`. The body goes in curly braces `{}`. If the body is a single statement, you *can* omit the braces — but **always use braces** in production code for clarity and safety.

---

## The Ternary Operator — Inline IIF()

In T-SQL you might use `IIF()` for a single condition:

```sql
-- T-SQL
SET @status = IIF(@balance >= 0, 'Active', 'Overdrawn');
```

C# has the **ternary operator** `condition ? valueIfTrue : valueIfFalse`:

```csharp
// C#
string status = balance >= 0 ? "Active" : "Overdrawn";
```

Use the ternary for simple, one-line conditions. Use `if/else` for anything more complex.

```csharp
// Good: simple assignment
decimal fee = isVip ? 0m : 5.00m;

// Bad: complex logic in ternary (use if/else instead)
decimal fee = isVip ? (amount > 1000 ? 0m : 2.50m) : (amount > 1000 ? 5m : 10m); // Too nested!
```

---

## The Null-Coalescing Operator — C#'s ISNULL/COALESCE

```sql
-- T-SQL: return @discount if not null, otherwise 0
SET @fee = @fee - ISNULL(@discount, 0);
SET @name = COALESCE(@firstName, @lastName, 'Unknown');
```

```csharp
// C#: ?? is ISNULL/COALESCE for two values
decimal actualFee = fee - (discount ?? 0m);
string displayName = firstName ?? lastName ?? "Unknown";

// ??= assigns only if the variable is null
discount ??= 0m;  // same as: if (discount == null) discount = 0m;
```

---

## SWITCH — Pattern Matching Power-Up

### T-SQL CASE WHEN

```sql
SET @description = CASE @transactionType
    WHEN 'DEP' THEN 'Deposit'
    WHEN 'WIT' THEN 'Withdrawal'
    WHEN 'TRF' THEN 'Transfer'
    ELSE 'Unknown'
END;
```

### C# switch statement (classic)

```csharp
string description;
switch (transactionType)
{
    case "DEP":
        description = "Deposit";
        break;
    case "WIT":
        description = "Withdrawal";
        break;
    case "TRF":
        description = "Transfer";
        break;
    default:
        description = "Unknown";
        break;
}
```

> **Note the `break`**: Unlike SQL's `CASE`, C# `switch` requires explicit `break` at the end of each case. Without it, you get a compiler error (C# doesn't allow accidental fall-through).

### C# switch expression (modern — C# 8+)

This is the preferred modern syntax — more concise, and it's an *expression* (has a value):

```csharp
string description = transactionType switch
{
    "DEP" => "Deposit",
    "WIT" => "Withdrawal",
    "TRF" => "Transfer",
    _     => "Unknown"   // _ is the default case
};
```

This is equivalent to SQL's `CASE WHEN ... ELSE ... END`. It's an expression, so you can use it inline.

### Pattern matching in switch expressions

C# switch expressions go far beyond SQL's `CASE` — you can match on types, conditions, and ranges:

```csharp
// Match on value ranges (no SQL equivalent)
string riskLevel = balance switch
{
    < 0          => "Negative",
    >= 0 and < 1000   => "Low",
    >= 1000 and < 50000 => "Medium",
    >= 50000     => "High"
};

// Match on type (no SQL equivalent)
string describe = transaction switch
{
    DepositTransaction d  => $"Deposit of {d.Amount:C}",
    WithdrawalTransaction w => $"Withdrawal of {w.Amount:C}",
    null => "No transaction",
    _    => "Unknown transaction type"
};
```

---

## Logical Operators

| T-SQL | C# | Meaning |
|---|---|---|
| `AND` | `&&` | Both must be true |
| `OR` | `\|\|` | At least one must be true |
| `NOT` | `!` | Negate |
| `=` | `==` | Equal comparison |
| `<>` | `!=` | Not equal |
| `IS NULL` | `== null` | Null check |
| `IS NOT NULL` | `!= null` or `is not null` | Non-null check |

```csharp
// T-SQL: IF @amount > 0 AND @accountStatus = 'Active' AND @balance IS NOT NULL
if (amount > 0 && accountStatus == "Active" && balance != null)
{
    // process
}

// C# also supports: is null / is not null (pattern matching)
if (balance is not null && balance > 1000m)
{
    // balance is guaranteed non-null here
}
```

---

## Summary

| SQL Concept | C# Equivalent |
|---|---|
| `IF...ELSE IF...ELSE` | `if...else if...else` |
| `CASE WHEN...THEN...ELSE END` | `switch` statement or switch expression |
| `IIF(cond, a, b)` | `cond ? a : b` |
| `ISNULL(x, default)` | `x ?? default` |
| `COALESCE(a, b, c)` | `a ?? b ?? c` |
| `AND` / `OR` / `NOT` | `&&` / `\|\|` / `!` |
| `IS NULL` | `== null` |
