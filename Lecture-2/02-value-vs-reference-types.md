# Value Types vs Reference Types

## A Concept SQL Doesn't Have

In SQL Server, every variable holds its data directly. `DECLARE @x INT = 5` stores the number 5 in `@x`. Period. There's no distinction about *how* the variable holds that data.

C# has a fundamental distinction that affects how variables behave: **value types** and **reference types**.

## The Key Difference

### Value Types -- The Variable IS the Data

A value type variable contains the actual data directly. When you copy it, you get an independent copy.

```csharp
int a = 42;
int b = a;      // b gets a COPY of the value 42
b = 100;        // Changing b does NOT affect a

Console.WriteLine(a);  // 42 -- unchanged
Console.WriteLine(b);  // 100
```

Think of it like this: `a` is a box containing the number 42. `b = a` creates a *new box* with its own copy of 42.

```
Before:  a: [42]    b: [42]     (two independent boxes)
After:   a: [42]    b: [100]    (changing b doesn't touch a)
```

### Reference Types -- The Variable Points to the Data

A reference type variable doesn't contain the data -- it contains a *reference* (address) to where the data lives in memory. When you copy it, both variables point to the *same data*.

```csharp
int[] arrayA = new int[] { 1, 2, 3 };
int[] arrayB = arrayA;      // arrayB points to the SAME array
arrayB[0] = 999;            // Changing through arrayB...

Console.WriteLine(arrayA[0]);  // 999 -- arrayA is affected too!
```

Think of it like this: `arrayA` is a piece of paper with an address written on it. `arrayB = arrayA` writes the same address on a second piece of paper. Both pieces of paper lead to the same house.

```
Before:  arrayA: [→ memory location 0x1234]  →  [1, 2, 3]
         arrayB: [→ memory location 0x1234]  →  (same array)

After:   arrayA: [→ memory location 0x1234]  →  [999, 2, 3]
         arrayB: [→ memory location 0x1234]  →  (same array, modified)
```

## SQL Analogy

The closest SQL analogy is the difference between **scalar variables** and **table variables / temp tables**:

```sql
-- Scalar: value is self-contained (like a value type)
DECLARE @amount DECIMAL(18,2) = 100.00

-- Table variable: holds a reference to a data structure
DECLARE @results TABLE (Id INT, Name NVARCHAR(50))
```

But in SQL, you can't assign one table variable to another, so the reference behavior never surfaces. In C#, it happens constantly.

## Which Types Are Which?

### Value Types (stored directly)

| Type | Example |
|---|---|
| `int`, `long`, `short`, `byte` | `int count = 5;` |
| `decimal`, `float`, `double` | `decimal amount = 100m;` |
| `bool` | `bool isActive = true;` |
| `char` | `char grade = 'A';` |
| `DateTime`, `DateOnly`, `TimeOnly` | `DateTime now = DateTime.Now;` |
| `TimeSpan` | `TimeSpan duration = TimeSpan.FromHours(2);` |
| `Guid` | `Guid id = Guid.NewGuid();` |
| `struct` (custom value types) | We'll cover these in Session 6 |
| `enum` (enumerated types) | We'll cover these in Session 6 |

### Reference Types (stored as a reference)

| Type | Example |
|---|---|
| `string` | `string name = "Davit";` |
| `object` | `object anything = 42;` |
| `int[]`, `string[]` (arrays) | `int[] numbers = { 1, 2, 3 };` |
| `List<T>`, `Dictionary<K,V>` | We'll cover these in Session 7 |
| Any `class` you create | We'll cover these in Session 4 |

> **Note:** `string` is technically a reference type, but it behaves like a value type because strings are **immutable** -- once created, a string can never be changed. Any "modification" creates a new string.

## Why This Matters

### Passing to Methods

```csharp
// Value type: the method gets a COPY
void DoubleIt(int x)
{
    x = x * 2;  // Only changes the local copy
}

int myNumber = 10;
DoubleIt(myNumber);
Console.WriteLine(myNumber);  // Still 10 -- the original wasn't changed
```

In SQL terms: this is like an INPUT parameter -- the procedure gets a copy, not the original.

```csharp
// Reference type: the method gets the SAME reference
void AddItem(List<int> list)
{
    list.Add(999);  // Modifies the ACTUAL list
}

List<int> myList = new List<int> { 1, 2, 3 };
AddItem(myList);
Console.WriteLine(myList.Count);  // 4 -- the original list was modified!
```

### Default Values

Value types always have a default value (they can't be "nothing"):

| Type | Default Value | SQL Equivalent |
|---|---|---|
| `int` | `0` | -- |
| `decimal` | `0m` | -- |
| `bool` | `false` | -- |
| `DateTime` | `0001-01-01 00:00:00` | -- |

Reference types default to `null` (nothing exists):

| Type | Default Value | SQL Equivalent |
|---|---|---|
| `string` | `null` | `NULL` |
| `object` | `null` | `NULL` |
| Arrays, Lists | `null` | `NULL` |

## The `var` Keyword

Instead of writing the type explicitly, you can use `var` and let the compiler figure it out:

```csharp
var count = 42;                    // Compiler knows this is int
var name = "Davit";                // Compiler knows this is string
var balance = 15000.50m;           // Compiler knows this is decimal
var today = DateTime.Now;          // Compiler knows this is DateTime
var items = new List<int>();       // Compiler knows this is List<int>
```

**Important:** `var` is NOT dynamic typing. The type is determined at compile time and is just as strict as writing it explicitly. It's a convenience, not a weakness.

```csharp
var x = 42;
x = "hello";  // COMPILE ERROR -- x is int, you can't assign a string
```

### When to Use `var`

- **Use `var`** when the type is obvious from the right side: `var list = new List<string>()`
- **Use explicit type** when it adds clarity: `decimal amount = GetAmount()` (otherwise you'd have to check what `GetAmount` returns)

## Constants

```csharp
// const -- compile-time constant, must be assigned immediately, cannot change
const decimal MAX_TRANSFER = 100_000m;
const string CURRENCY = "GEL";

// readonly -- runtime constant, can be assigned in constructor, cannot change after
readonly DateTime createdAt = DateTime.Now;  // Evaluated at runtime
```

| C# | SQL Equivalent | Notes |
|---|---|---|
| `const` | Hard-coded value | Must know the value at compile time |
| `readonly` | No direct equivalent | Set once, then immutable |

## Summary

| Concept | Key Point |
|---|---|
| **Value types** | Variable contains the data directly. Copy = independent clone. (`int`, `decimal`, `bool`, `DateTime`) |
| **Reference types** | Variable contains a reference to the data. Copy = both point to the same data. (`string`, arrays, classes) |
| **`var`** | Compiler infers the type. Still strictly typed -- just shorthand. |
| **`const`** | Compile-time constant. Value cannot change. |
| **`readonly`** | Runtime constant. Set once, then frozen. |

---

*Previous: [The C# Type System](01-type-system-overview.md) | Next: [Operators and Expressions](03-operators-and-expressions.md)*
