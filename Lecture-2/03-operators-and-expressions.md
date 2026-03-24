# Operators and Expressions

## Most Operators Are Identical to T-SQL

Good news: if you can write a `WHERE` clause, you already know most C# operators.

## Arithmetic Operators

| Operator | C# | T-SQL | Notes |
|---|---|---|---|
| Addition | `a + b` | `@a + @b` | Identical |
| Subtraction | `a - b` | `@a - @b` | Identical |
| Multiplication | `a * b` | `@a * @b` | Identical |
| Division | `a / b` | `@a / @b` | **Watch out:** integer division truncates (see below) |
| Modulo | `a % b` | `@a % @b` | Identical |

### Integer Division Warning

```csharp
int a = 7;
int b = 2;
Console.WriteLine(a / b);       // 3  (not 3.5! Integer division truncates)
Console.WriteLine(a / (decimal)b); // 3.5 (cast one operand to decimal)
```

This is exactly the same as SQL:
```sql
SELECT 7 / 2          -- 3 (integer division)
SELECT 7 / CAST(2 AS DECIMAL(18,2))  -- 3.5
```

### Compound Assignment Operators (No SQL Equivalent)

```csharp
int count = 10;
count += 5;    // count = count + 5  → 15
count -= 3;    // count = count - 3  → 12
count *= 2;    // count = count * 2  → 24
count /= 4;    // count = count / 4  → 6
count %= 4;    // count = count % 4  → 2

// Increment / Decrement
count++;       // count = count + 1
count--;       // count = count - 1
```

> **No SQL equivalent:** In T-SQL you always write `SET @count = @count + 1`. C# gives you shortcuts.

## Comparison Operators

| Operation | C# | T-SQL | Notes |
|---|---|---|---|
| Equal | `a == b` | `@a = @b` | **Two** equal signs in C#! |
| Not equal | `a != b` | `@a <> @b` or `@a != @b` | Both work in T-SQL; only `!=` in C# |
| Greater than | `a > b` | `@a > @b` | Identical |
| Less than | `a < b` | `@a < @b` | Identical |
| Greater or equal | `a >= b` | `@a >= @b` | Identical |
| Less or equal | `a <= b` | `@a <= @b` | Identical |

> **Critical difference:** In C#, `=` is **assignment** and `==` is **comparison**. In T-SQL, `=` does both depending on context.

```csharp
int x = 5;         // Assignment: x now holds 5
if (x == 5)        // Comparison: is x equal to 5?
{
    Console.WriteLine("x is five");
}

// Common mistake:
if (x = 5)  // COMPILE ERROR in C# -- this is an assignment, not a comparison
```

## Logical Operators

| Operation | C# | T-SQL | Notes |
|---|---|---|---|
| AND | `&&` | `AND` | Short-circuit: if left is false, right isn't evaluated |
| OR | `\|\|` | `OR` | Short-circuit: if left is true, right isn't evaluated |
| NOT | `!` | `NOT` | Prefix operator |

```csharp
bool isActive = true;
bool hasBalance = true;

// C#:
if (isActive && hasBalance)
{
    Console.WriteLine("Can transact");
}

// Equivalent T-SQL:
// IF @isActive = 1 AND @hasBalance = 1
```

### Short-Circuit Evaluation

```csharp
// Safe: if account is null, the second condition is never evaluated
if (account != null && account.Balance > 0)
{
    // Process...
}
```

In T-SQL, SQL Server may evaluate both sides of an `AND` in any order. In C#, `&&` guarantees left-to-right, and stops early if the left side is `false`. This is very useful for null checks.

## Null Operators -- Your New Superpowers

### Null-Coalescing `??` -- Like ISNULL / COALESCE

```csharp
string? name = null;
string displayName = name ?? "Unknown";  // "Unknown" (because name is null)
```

T-SQL equivalent:
```sql
DECLARE @name NVARCHAR(50) = NULL
SELECT ISNULL(@name, 'Unknown')      -- 'Unknown'
-- or
SELECT COALESCE(@name, 'Unknown')    -- 'Unknown'
```

### Chaining `??` -- Like COALESCE with Multiple Arguments

```csharp
string? first = null;
string? second = null;
string? third = "Found it";
string result = first ?? second ?? third ?? "Default";  // "Found it"
```

T-SQL equivalent:
```sql
SELECT COALESCE(@first, @second, @third, 'Default')
```

### Null-Coalescing Assignment `??=`

```csharp
string? name = null;
name ??= "Default";  // Assigns "Default" only if name is null
// name is now "Default"

name ??= "Other";    // Does nothing because name is no longer null
// name is still "Default"
```

No direct T-SQL equivalent -- you'd have to write:
```sql
IF @name IS NULL SET @name = 'Default'
```

### Null-Conditional `?.` -- Safe Navigation

This is the most powerful null operator. It checks for null before accessing a member:

```csharp
string? name = null;

// WITHOUT null-conditional -- throws NullReferenceException!
// int length = name.Length;  // CRASH!

// WITH null-conditional -- returns null instead of crashing
int? length = name?.Length;  // null (no crash)

// Combine with ?? for a default value
int safeLength = name?.Length ?? 0;  // 0
```

No direct T-SQL equivalent, but the concept is similar to:
```sql
-- You never need this in SQL because NULL propagates naturally:
SELECT LEN(@name)  -- Returns NULL if @name is NULL (no error)
```

### Chaining Null-Conditional

```csharp
// If customer is null, OR customer.Address is null, return "N/A"
string city = customer?.Address?.City ?? "N/A";
```

Without the `?.` operator, you'd need:
```csharp
string city;
if (customer != null && customer.Address != null)
    city = customer.Address.City;
else
    city = "N/A";
```

## Type Conversion (Casting)

### SQL: CAST and TRY_CAST

```sql
SELECT CAST('123' AS INT)          -- 123 (crashes if invalid)
SELECT TRY_CAST('abc' AS INT)      -- NULL (safe, returns NULL if invalid)
```

### C#: Multiple Options

```csharp
// 1. Direct cast -- throws exception if types are incompatible
int fromDouble = (int)3.14;           // 3 (truncates decimal part)

// 2. Convert class -- throws on invalid input
int fromString = Convert.ToInt32("123");  // 123
// Convert.ToInt32("abc") -- throws FormatException

// 3. Parse -- throws on invalid input (like CAST)
int parsed = int.Parse("123");         // 123
// int.Parse("abc") -- throws FormatException

// 4. TryParse -- returns bool, no exception (like TRY_CAST)
bool success = int.TryParse("123", out int result);  // success = true, result = 123
bool failed = int.TryParse("abc", out int bad);      // failed = false, bad = 0
```

### When to Use Which

| Method | Use When | SQL Equivalent |
|---|---|---|
| `(int)value` | You're sure the types are compatible | Implicit conversion |
| `Convert.ToInt32()` | Converting between different types | `CAST(@x AS INT)` |
| `int.Parse()` | Parsing strings you trust | `CAST(@str AS INT)` |
| `int.TryParse()` | Parsing user input (might be invalid) | `TRY_CAST(@str AS INT)` |

> **Best practice for user input:** Always use `TryParse`. Never trust that user input is valid.

```csharp
Console.Write("Enter amount: ");
string? input = Console.ReadLine();

if (decimal.TryParse(input, out decimal amount))
{
    Console.WriteLine($"You entered: {amount:N2}");
}
else
{
    Console.WriteLine("Invalid amount. Please enter a number.");
}
```

## String Operations

### Common String Methods

| C# | T-SQL | Example |
|---|---|---|
| `str.Length` | `LEN(@str)` | `"hello".Length` → `5` |
| `str.ToUpper()` | `UPPER(@str)` | `"hello".ToUpper()` → `"HELLO"` |
| `str.ToLower()` | `LOWER(@str)` | `"HELLO".ToLower()` → `"hello"` |
| `str.Trim()` | `TRIM(@str)` | `"  hi  ".Trim()` → `"hi"` |
| `str.TrimStart()` | `LTRIM(@str)` | `"  hi".TrimStart()` → `"hi"` |
| `str.TrimEnd()` | `RTRIM(@str)` | `"hi  ".TrimEnd()` → `"hi"` |
| `str.Substring(start, len)` | `SUBSTRING(@str, start, len)` | `"hello".Substring(1, 3)` → `"ell"` |
| `str.Contains("x")` | `CHARINDEX('x', @str) > 0` | `"hello".Contains("ell")` → `true` |
| `str.StartsWith("x")` | `@str LIKE 'x%'` | `"hello".StartsWith("he")` → `true` |
| `str.EndsWith("x")` | `@str LIKE '%x'` | `"hello".EndsWith("lo")` → `true` |
| `str.Replace("a", "b")` | `REPLACE(@str, 'a', 'b')` | `"hello".Replace("l", "r")` → `"herro"` |
| `str.Split(',')` | `STRING_SPLIT(@str, ',')` | `"a,b,c".Split(',')` → `["a","b","c"]` |
| `str.IndexOf("x")` | `CHARINDEX('x', @str) - 1` | `"hello".IndexOf("l")` → `2` (0-based!) |
| `string.Join(",", arr)` | `STRING_AGG(@col, ',')` | `string.Join(",", new[]{"a","b"})` → `"a,b"` |

> **Important difference:** C# uses **0-based indexing**. The first character is at position 0, not 1 like in SQL Server's `SUBSTRING`.

```csharp
string text = "Hello, Banking!";

// 0-based indexing
Console.WriteLine(text[0]);           // 'H' (first character)
Console.WriteLine(text.Substring(7)); // "Banking!"
Console.WriteLine(text.Length);       // 15
```

### String Immutability

Strings in C# are **immutable** -- they cannot be changed after creation. Every "modification" creates a new string:

```csharp
string name = "hello";
string upper = name.ToUpper();  // Creates a new string "HELLO"
// name is still "hello" -- it wasn't modified
```

This is similar to how SQL works -- `UPPER(@name)` returns a new value, it doesn't change `@name`.

## Summary

| Concept | Key Takeaway |
|---|---|
| **Arithmetic** | Same as T-SQL. Watch integer division truncation. |
| **Comparison** | `==` for equality (not `=`). `!=` for not-equal. |
| **Logical** | `&&` (AND), `\|\|` (OR), `!` (NOT). Short-circuit evaluation. |
| **`??`** | Null-coalescing = ISNULL/COALESCE |
| **`?.`** | Null-conditional = safe navigation (no SQL equivalent) |
| **`??=`** | Assign only if null |
| **Casting** | Use `TryParse` for user input, `Parse` for trusted strings |
| **Strings** | 0-based indexing. Immutable. Rich set of methods. |

---

*Previous: [Value Types vs Reference Types](02-value-vs-reference-types.md) | Next: [Hands-on Exercises](04-hands-on-exercises.md)*
