# The C# Type System -- A SQL Developer's Map

## Every SQL Type Has a C# Equivalent

As a SQL Server developer, you already think in types every day. You choose `INT` vs `BIGINT`, `DECIMAL(18,2)` vs `MONEY`, `VARCHAR` vs `NVARCHAR`. C# has the same decisions, with the same tradeoffs -- just different names.

## The Complete Type Mapping

### Numeric Types

| SQL Server | C# | Size | Range | When to Use |
|---|---|---|---|---|
| `TINYINT` | `byte` | 1 byte | 0 to 255 | Flags, small counters |
| `SMALLINT` | `short` | 2 bytes | -32,768 to 32,767 | Rarely used |
| `INT` | `int` | 4 bytes | -2.1B to 2.1B | General-purpose integers |
| `BIGINT` | `long` | 8 bytes | -9.2 quintillion to 9.2 quintillion | Large IDs, timestamps |
| `REAL` | `float` | 4 bytes | ~7 significant digits | Scientific data (avoid for money!) |
| `FLOAT` | `double` | 8 bytes | ~15 significant digits | Scientific data (avoid for money!) |
| `DECIMAL(p,s)` | `decimal` | 16 bytes | 28-29 significant digits | **Money, financial calculations** |
| `MONEY` | `decimal` | 8 bytes | -- | No direct equivalent; use `decimal` |

> **Critical for banking:** Always use `decimal` for financial amounts. Never use `float` or `double` -- they have floating-point precision errors. `0.1 + 0.2` might equal `0.30000000000000004` with `double`, but `0.3` with `decimal`.

```csharp
// WRONG -- floating point error
double wrongAmount = 0.1 + 0.2;        // 0.30000000000000004

// CORRECT -- exact decimal arithmetic
decimal correctAmount = 0.1m + 0.2m;   // 0.3
```

### Text Types

| SQL Server | C# | Notes |
|---|---|---|
| `CHAR(n)` / `VARCHAR(n)` | `string` | C# strings are always Unicode, always variable-length |
| `NCHAR(n)` / `NVARCHAR(n)` | `string` | Same as above -- no separate "N" variant needed |
| `VARCHAR(MAX)` / `NVARCHAR(MAX)` | `string` | Strings have no fixed size limit in C# |
| `TEXT` / `NTEXT` | `string` | Deprecated in SQL; just `string` in C# |

> **Simplification:** In SQL Server you choose between `VARCHAR` and `NVARCHAR`, fixed vs variable, with a size limit. In C#, there's just `string` -- it's always Unicode, always variable-length, and has no declared size limit.

```csharp
string name = "Davit";                    // Like NVARCHAR
string longText = "This can be any length, no MAX needed";
string empty = "";                         // Empty string (not NULL)
string? nullable = null;                   // Nullable string (like NULL in SQL)
```

### Boolean

| SQL Server | C# | Notes |
|---|---|---|
| `BIT` | `bool` | `true`/`false` instead of `1`/`0` |

```csharp
bool isActive = true;    // SQL: DECLARE @isActive BIT = 1
bool isFrozen = false;   // SQL: DECLARE @isFrozen BIT = 0

// In SQL: IF @isActive = 1
// In C#:
if (isActive)  // No need to compare to true -- it IS a boolean
{
    Console.WriteLine("Account is active");
}
```

### Date and Time

| SQL Server | C# | Notes |
|---|---|---|
| `DATE` | `DateOnly` | Date without time (C# 10+) |
| `TIME` | `TimeOnly` | Time without date (C# 10+) |
| `DATETIME` | `DateTime` | Date + time combined |
| `DATETIME2` | `DateTime` | Same C# type, higher precision |
| `DATETIMEOFFSET` | `DateTimeOffset` | Date + time + timezone offset |
| -- | `TimeSpan` | Duration (like DATEDIFF result) |

```csharp
DateTime now = DateTime.Now;                       // Current date and time
DateOnly today = DateOnly.FromDateTime(DateTime.Now); // Just the date
TimeOnly currentTime = TimeOnly.FromDateTime(DateTime.Now); // Just the time
DateTimeOffset withOffset = DateTimeOffset.Now;    // With timezone info

TimeSpan duration = TimeSpan.FromHours(2.5);       // 2 hours 30 minutes
DateTime future = now + duration;                  // Add duration to a date
```

### Special Types

| SQL Server | C# | Notes |
|---|---|---|
| `UNIQUEIDENTIFIER` | `Guid` | 128-bit unique identifier |
| `VARBINARY(n)` | `byte[]` | Binary data as byte array |
| `XML` | `string` or `XDocument` | Typically handled as string |
| `JSON` (2022+) | `string` or deserialized object | Use `System.Text.Json` |
| `NULL` | `null` | Same concept, different handling |

```csharp
Guid transactionId = Guid.NewGuid();  // Like NEWID() in SQL
Console.WriteLine(transactionId);      // e.g., "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

## Summary: The Quick Reference Card

| SQL Server | C# | Notes |
|---|---|---|
| INT | `int` | |
| BIGINT | `long` | |
| DECIMAL | `decimal` | Use `m` suffix: `100.50m` |
| BIT | `bool` | `true`/`false`, not `1`/`0` |
| VARCHAR | `string` | Always Unicode, no size limit |
| NVARCHAR | `string` | |
| DATETIME | `DateTime` | |
| DATE | `DateOnly` | |
| UNIQUEIDENTIFIER | `Guid` | |
| NULL | `null` | |

---

*Next: [Value Types vs Reference Types](02-value-vs-reference-types.md)*
