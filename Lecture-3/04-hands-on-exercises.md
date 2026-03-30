# Hands-On Exercises — Session 3: Rewrite Real HSP Functions in C#

## The Goal

Rewrite **real production functions** from our HSP database as C# methods. These are actual functions running in our system today — same business logic, no SQL. Each exercise maps directly to concepts from this session.

---

## Functions Used in This Exercise

| # | Exercise | HSP Function | Difficulty 
|---|---|---|---|
| 1 | Error Code Lookup | `hp.fnGetErrorNumberConstant` | Warm-up |
| 2 | Action Result Classification | `hp.fnGetARCClassConstant` | Warm-up |
| 3 | Document Verification Status | `hp.fnGetVerificationEnumById` | Easy |
| 4 | IBAN Modulo 97 | `dbo.iban_mod97`  | Medium |
| 5 | IBAN Check Key Generator | `dbo.iban_set_key` | Medium |
| 6 | Generate Account Number | `hp.fnGenerateAccountNumber` |  Medium |
| 7 | Parse Account Number | `hp.fnParseAccountNumber` |  Medium |
| 8 | Split String | `hp.fnSplitString` | Medium+ |
| 9 | Extract Template Placeholders | `dbo.ExtractPlaceholders` |  Medium+ |
| 10 | Fee Calculation | `hp.fnCalcFeeAmount` | Challenge |
| Bonus | IBAN Generator | `dbo.generate_iban` | Challenge |

---

## How To Approach Each Exercise

1. **Read the original T-SQL** — understand what it does before writing anything
2. **Identify the C# equivalent** for each SQL construct (see the mapping table at the bottom)
3. **Write the C# method** — match the exact same behavior
4. **Write unit tests** — verify your implementation using the test cases table

---

## Project Setup

You will need **two projects** in a single solution:

### 1. Class Library (your implementations)

```bash
dotnet new classlib -n HspFunctions
```

Put all your methods in a **public static class** so the test project can access them:

```csharp
// HspFunctions/HspFunctions.cs
namespace HspFunctions;

public static class HspFunctions
{
    // All your exercise methods go here as public static methods
}
```

### 2. xUnit Test Project

```bash
dotnet new xunit -n HspFunctions.Tests
cd HspFunctions.Tests
dotnet add reference ../HspFunctions/HspFunctions.csproj
```

### xUnit Quick Reference

| What | How |
|---|---|
| Test method | `[Fact]` attribute on a public method |
| Parameterized test | `[Theory]` + `[InlineData(...)]` |
| Assert equal | `Assert.Equal(expected, actual)` |
| Assert null | `Assert.Null(actual)` |
| Assert not null | `Assert.NotNull(actual)` |
| Assert empty collection | `Assert.Empty(actual)` |
| Assert array contents | `Assert.Equal(new[] {"a", "b"}, actual)` |
| Assert string starts with | `Assert.StartsWith("GE", actual)` |
| Assert string ends with | `Assert.EndsWith("123", actual)` |
| Assert string contains | `Assert.Contains("USD", actual)` |

```csharp
// Example test file
using Xunit;
using static HspFunctions.HspFunctions;

public class ExampleTests
{
    [Fact]
    public void MyMethod_WhenCalledWithX_ReturnsY()
    {
        var result = MyMethod("X");
        Assert.Equal("Y", result);
    }

    [Theory]
    [InlineData("input1", "expected1")]
    [InlineData("input2", "expected2")]
    public void MyMethod_WithVariousInputs_ReturnsExpected(string input, string expected)
    {
        Assert.Equal(expected, MyMethod(input));
    }
}
```

Run tests with:
```bash
cd HspFunctions.Tests
dotnet test
```

---

## Exercise 1: Error Code Lookup — `hp.fnGetErrorNumberConstant`

**Concepts:** `switch` expression, string-to-int mapping

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnGetErrorNumberConstant.sql
CREATE FUNCTION hp.[fnGetErrorNumberConstant]
(
    @Type VARCHAR(50) = 'UserErrorClass60000'
)
RETURNS INT
AS
BEGIN
    RETURN (
        CASE
        WHEN @Type = 'UserErrorClass60000' THEN 60000
        WHEN @Type = 'SysErrorClass60001' THEN 60001
        WHEN @Type = 'ARC_GeneralError101' THEN 101
        ELSE 60000 END
    )
END
```

### Your Task

```csharp
static int GetErrorNumberConstant(string type = "UserErrorClass60000")
{
    // TODO: Use a switch expression to map type string to error code
    // Default case should return 60000
}
```

### Test Cases — Implement in `ErrorNumberConstantTests.cs`

| # | Test case | Input (`type`) | Expected |
|---|---|---|---|
| 1 | Known type: user error class | `"UserErrorClass60000"` | `60000` |
| 2 | Known type: system error class | `"SysErrorClass60001"` | `60001` |
| 3 | Known type: general error | `"ARC_GeneralError101"` | `101` |
| 4 | Unknown type falls back to default | `"SomethingElse"` | `60000` |
| 5 | Empty string falls back to default | `""` | `60000` |
| 6 | Another unknown type | `"unknown_type"` | `60000` |
| 7 | No argument uses default parameter | *(no argument)* | `60000` |

---

## Exercise 2: Action Result Classification — `hp.fnGetARCClassConstant`

**Concepts:** `switch` with range patterns, `if/else if` chain

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnGetARCClassConstant.sql
CREATE FUNCTION hp.[fnGetARCClassConstant]
(
    @ActionResultCode INT
)
RETURNS INT
AS
BEGIN
    RETURN (
        CASE
        WHEN @ActionResultCode BETWEEN 0 AND 99 THEN 0       -- Successful ARC Class
        WHEN @ActionResultCode BETWEEN 100 AND 399 THEN 100  -- General System ARC Class
        WHEN @ActionResultCode BETWEEN 400 AND 699 THEN 400  -- User/Customer ARC Class
        WHEN @ActionResultCode BETWEEN 700 AND 999 THEN 700  -- Reserved / N/A
        WHEN @ActionResultCode >= 1000 THEN 1000
        ELSE NULL END
    )
END
```

### Your Task

Write **two versions** of this method:

```csharp
// Version A: Use if/else if chain
static int? GetARCClassConstant_IfElse(int actionResultCode)
{
    // TODO: Translate the BETWEEN ranges to >= and <= checks
    // Return null for negative values (ELSE NULL)
}

// Version B: Use switch expression with range patterns
static int? GetARCClassConstant_Switch(int actionResultCode)
{
    // TODO: Use patterns like: >= 0 and <= 99 => 0
    // Use _ => null for default
}
```

**Hint:** C# `int?` (nullable int) is how you represent a value that can be `NULL`.

### Test Cases — Implement in `ARCClassConstantTests.cs`

Test **both versions** and verify they produce the same result for every input.

| # | Test case | Input (`actionResultCode`) | Expected |
|---|---|---|---|
| 1 | Lower boundary of success range | `0` | `0` |
| 2 | Mid success range | `50` | `0` |
| 3 | Upper boundary of success range | `99` | `0` |
| 4 | Lower boundary of system range | `100` | `100` |
| 5 | Mid system range | `250` | `100` |
| 6 | Upper boundary of system range | `399` | `100` |
| 7 | Lower boundary of customer range | `400` | `400` |
| 8 | Upper boundary of customer range | `699` | `400` |
| 9 | Lower boundary of reserved range | `700` | `700` |
| 10 | Upper boundary of reserved range | `999` | `700` |
| 11 | Lower boundary of overflow range | `1000` | `1000` |
| 12 | Well above 1000 | `1500` | `1000` |
| 13 | Very large value | `99999` | `1000` |
| 14 | Negative value returns null | `-1` | `null` |
| 15 | Another negative value | `-100` | `null` |
| 16 | Both versions agree for all values -10 to 1100 | loop -10..1100 | `IfElse == Switch` |

---

## Exercise 3: Document Verification Status — `hp.fnGetVerificationEnumById`

**Concepts:** `switch` with multiple case labels, `IN(...)` equivalent

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnGetVerificationEnumById.sql
CREATE FUNCTION [hp].[fnGetVerificationEnumById]
(
    @DocumentVerificationStatus INT
)
RETURNS VARCHAR(50)
AS
BEGIN
    DECLARE @VerificationEnum VARCHAR(50);

    SET @VerificationEnum = CASE
        WHEN @DocumentVerificationStatus = 1 THEN 'Enum_Verified'
        WHEN @DocumentVerificationStatus IN ( 4 ) THEN 'Enum_InReview'
        WHEN @DocumentVerificationStatus IN ( 0, 5, 2 ) THEN 'Enum_NotVerified'
        WHEN @DocumentVerificationStatus IN ( 6 ) THEN 'Enum_Transactions_Denied'
        ELSE 'Enum_None'
    END

    RETURN @VerificationEnum;
END;
```

### Your Task

```csharp
static string GetVerificationEnumById(int documentVerificationStatus)
{
    // TODO: Use a switch expression
    // SQL's IN(0, 5, 2) becomes multiple case labels: 0 or 5 or 2 =>
    // Return the matching enum string
}
```

**Hint:** In C# switch expressions, you can match multiple values with `or`:
```csharp
var result = value switch
{
    0 or 5 or 2 => "Enum_NotVerified",
    // ...
};
```

### Test Cases — Implement in `VerificationEnumTests.cs`

| # | Test case | Input (`status`) | Expected |
|---|---|---|---|
| 1 | Verified | `1` | `"Enum_Verified"` |
| 2 | In review | `4` | `"Enum_InReview"` |
| 3 | Not verified (status 0) | `0` | `"Enum_NotVerified"` |
| 4 | Not verified (status 2) | `2` | `"Enum_NotVerified"` |
| 5 | Not verified (status 5) | `5` | `"Enum_NotVerified"` |
| 6 | Transactions denied | `6` | `"Enum_Transactions_Denied"` |
| 7 | Unmapped status falls to default | `3` | `"Enum_None"` |
| 8 | Another unmapped status | `7` | `"Enum_None"` |
| 9 | Large unmapped status | `99` | `"Enum_None"` |
| 10 | Negative unmapped status | `-1` | `"Enum_None"` |

---

## Exercise 4: IBAN Modulo 97 — `dbo.iban_mod97`

**Concepts:** `while` loop, string character access, char-to-int conversion, modulo operator

### Original T-SQL

```sql
-- Source: HSP/dbo/Functions/iban_mod97.sql
CREATE FUNCTION dbo.iban_mod97 (@int varchar(50))
RETURNS int
AS
BEGIN
    DECLARE
        @mod_value int,
        @digit int,
        @i int;

    SET @mod_value = 0;
    SET @i = 1;

    WHILE @i <= LEN(@int)
    BEGIN
        SET @digit = ASCII(SUBSTRING(@int, @i, 1)) - ASCII('0');
        SET @mod_value = ((@mod_value * 10) + @digit) % 97;
        SET @i = @i + 1;
    END

    RETURN @mod_value;
END;
```

### Your Task

```csharp
static int IbanMod97(string numericString)
{
    // TODO:
    // Loop through each character in the string
    // Convert char to digit: numericString[i] - '0'  (same as ASCII(@ch) - ASCII('0'))
    // Accumulate: modValue = (modValue * 10 + digit) % 97
    // Return the final modValue
}
```

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `LEN(@int)` | `numericString.Length` |
| `SUBSTRING(@int, @i, 1)` | `numericString[i]` |
| `ASCII(ch) - ASCII('0')` | `ch - '0'` (char arithmetic) |
| `@i = 1; WHILE @i <= LEN(...)` | `for (int i = 0; i < str.Length; i++)` (0-based!) |

### Test Cases — Implement in `IbanMod97Tests.cs`

| # | Test case | Input (`numericString`) | Expected |
|---|---|---|---|
| 1 | Zero | `"0"` | `0` |
| 2 | 97 mod 97 = 0 | `"97"` | `0` |
| 3 | 194 mod 97 = 0 (97 * 2) | `"194"` | `0` |
| 4 | 100 mod 97 = 3 | `"100"` | `3` |
| 5 | 98 mod 97 = 1 | `"98"` | `1` |
| 6 | Single digit | `"1"` | `1` |
| 7 | 96 mod 97 = 96 | `"96"` | `96` |
| 8 | Large number (no int overflow) | `"161400182100011234561014"` | result is >= 0 and < 97 |

---

## Exercise 5: IBAN Check Key Generator — `dbo.iban_set_key`

**Concepts:** `while`/`for` loop, string manipulation, `if/else`, calling another method, early return

### Original T-SQL

```sql
-- Source: HSP/dbo/Functions/iban_set_key.sql
CREATE FUNCTION dbo.iban_set_key (@iban varchar(22))
RETURNS varchar(22)
AS
BEGIN
    DECLARE
        @iban2 varchar(50),
        @s varchar(50),
        @ch char,
        @i int

    SET @iban2 = UPPER(REPLACE(@iban, ' ', ''));

    IF LEN(@iban2) <> 22
        RETURN NULL

    IF LEFT(@iban2, 2) <> 'GE'
        RETURN NULL

    SET @iban2 = RIGHT(@iban2, LEN(@iban2) - 4) + '161400';
    SET @s = ''

    SET @i = 1

    WHILE @i <= LEN(@iban2)
    BEGIN
        SET @ch = SUBSTRING(@iban2, @i, 1);
        IF ASCII(@ch) BETWEEN ASCII('A') AND ASCII('Z')
            SET @s = @s + CONVERT(varchar(2), (ASCII(@ch) - ASCII('A') + 10))
        ELSE
            SET @s = @s + @ch

        SET @i = @i + 1
    END;

    DECLARE @key varchar(2)

    SET @key = CONVERT(varchar(2), 98 - dbo.iban_mod97(@s));
    IF LEN(@key) = 1
        SET @key = '0' + @key

    SET @iban = 'GE' + @key + LEFT(@iban2, 18);
    RETURN @iban
END;
```

### Your Task

```csharp
static string? IbanSetKey(string iban)
{
    // TODO:
    // 1. Uppercase and remove spaces
    // 2. Validate: length must be 22, must start with "GE" — return null if not
    // 3. Rearrange: take characters after first 4, append "161400"
    // 4. Convert letters to numbers: A=10, B=11, ..., Z=35
    //    - Loop through each char
    //    - If letter: append (ch - 'A' + 10).ToString()
    //    - If digit: append the char as-is
    // 5. Calculate check key: 98 - IbanMod97(s)
    // 6. Pad to 2 digits with leading zero if needed
    // 7. Return "GE" + key + first 18 chars of rearranged string
}
```

**Hint:** This method calls `IbanMod97` from Exercise 4 — method composition in action!

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `UPPER(REPLACE(@iban, ' ', ''))` | `iban.ToUpper().Replace(" ", "")` |
| `LEFT(@s, 2)` | `s[..2]` or `s.Substring(0, 2)` |
| `RIGHT(@s, LEN(@s) - 4)` | `s[4..]` or `s.Substring(4)` |
| `CONVERT(varchar, num)` | `num.ToString()` |
| `LEN(@key) = 1` → pad | `key.PadLeft(2, '0')` |

### Test Cases — Implement in `IbanSetKeyTests.cs`

| # | Test case | Input (`iban`) | Expected |
|---|---|---|---|
| 1 | Valid input returns 22-char string | `"GE00XX01123456789012"` | result is not null, length = 22 |
| 2 | Result starts with "GE" | `"GE00XX01123456789012"` | starts with `"GE"` |
| 3 | Body after check digits is preserved | `"GE00XX01123456789012"` | result[4..] = `"XX01123456789012"` |
| 4 | Too short returns null | `"SHORT"` | `null` |
| 5 | Empty string returns null | `""` | `null` |
| 6 | Almost valid (6 chars) returns null | `"GE00XX"` | `null` |
| 7 | Wrong country prefix returns null | `"US00XX01123456789012"` | `null` |
| 8 | Spaces are stripped before validation | `"GE00 XX01 1234 5678 9012"` | same result as without spaces |
| 9 | Lowercase is handled via ToUpper | `"ge00xx01123456789012"` | same result as uppercase |
| 10 | Deterministic (same input = same output) | call twice with `"GE00XX01123456789012"` | both results are equal |

---

## Exercise 6: Generate Account Number — `hp.fnGenerateAccountNumber`

**Concepts:** `if/else if`, `PadLeft`, string concatenation/interpolation, default parameters

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnGenerateAccountNumber.sql
CREATE FUNCTION hp.[fnGenerateAccountNumber]
(
    @AccountID          INT,
    @CCY                VARCHAR(10) = 'UPG',
    @Type               INT = 1,
    @UsageType          INT = 0,
    @AccountInitial     VARCHAR(10) = 'C',
    @AccountNumberSchemaTypeChar VARCHAR(10) = 'A'
)
RETURNS VARCHAR(50)
AS
BEGIN
    DECLARE @AccountNumber       VARCHAR(50),
            @AccountNumberLength INT = 20,
            @AccountNumberPrefix VARCHAR(50)

    IF @AccountNumberSchemaTypeChar = 'A' BEGIN

        SET @AccountNumberPrefix =
            CONVERT(varchar(10), @AccountNumberSchemaTypeChar) +
            CONVERT(varchar(10), @AccountInitial) +
            REPLICATE('0', 2 - LEN(@Type)) +
            CONVERT(NVARCHAR(10), @Type) +
            CONVERT(NVARCHAR(10), @CCY) +
            REPLICATE('0', 2 - LEN(@UsageType)) +
            CONVERT(NVARCHAR(10), @UsageType)

        SET @AccountNumber =
            @AccountNumberPrefix +
            REPLICATE('0', @AccountNumberLength - LEN(@AccountNumberPrefix) - LEN(@AccountID)) +
            CONVERT(NVARCHAR(100), @AccountID)

    END
    ELSE IF @AccountNumberSchemaTypeChar = 'B' BEGIN
        SET @AccountNumber = NULL
    END

    RETURN @AccountNumber
END
```

### Your Task

```csharp
static string? GenerateAccountNumber(
    int accountId,
    string ccy = "UPG",
    int type = 1,
    int usageType = 0,
    string accountInitial = "C",
    string schemaTypeChar = "A")
{
    // TODO:
    // Schema 'A':
    //   1. Build prefix: schemaTypeChar + accountInitial + type(padded to 2) + ccy + usageType(padded to 2)
    //   2. Build full number: prefix + zeros + accountId (total length = 20)
    // Schema 'B':
    //   Return null
    //
    // Hint: Use .ToString().PadLeft(2, '0') for zero-padding
    //       Use .PadLeft(totalLength, '0') or string concatenation for the ID part
}
```

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `REPLICATE('0', 2 - LEN(@Type)) + CONVERT(varchar, @Type)` | `type.ToString().PadLeft(2, '0')` |
| `REPLICATE('0', n - LEN(x)) + CONVERT(varchar, x)` | `x.ToString().PadLeft(n, '0')` |

### Test Cases — Implement in `GenerateAccountNumberTests.cs`

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Default params produce 20-char string | `accountId: 12345` | result length = 20 |
| 2 | Starts with schema + initial | `accountId: 12345` (defaults) | starts with `"AC"` |
| 3 | Contains currency code | `accountId: 12345` (defaults) | contains `"UPG"` |
| 4 | Ends with account ID | `accountId: 12345` (defaults) | ends with `"12345"` |
| 5 | Ends with account ID = 1 | `accountId: 1` (defaults) | ends with `"1"` |
| 6 | Ends with account ID = 99999 | `accountId: 99999` (defaults) | ends with `"99999"` |
| 7 | Type padded to 2 digits | `accountId: 1` (defaults, type=1) | result[2..4] = `"01"` |
| 8 | Custom params build correctly | `accountId: 1, ccy: "GEL", type: 5, usageType: 1, initial: "M"` | length = 20, starts with `"AM"`, contains `"GEL"` |
| 9 | Schema B returns null | `accountId: 1, schemaTypeChar: "B"` | `null` |
| 10 | Large account ID still fits 20 chars | `accountId: 999999999` | length = 20 |

---

## Exercise 7: Parse Account Number — `hp.fnParseAccountNumber`

**Concepts:** `Substring`, `int.Parse`, returning a tuple (replaces SQL table-valued return)

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnParseAccountNumber.sql
CREATE FUNCTION hp.[fnParseAccountNumber]
(
    @AccountNumber VARCHAR(50)
)
RETURNS @AccountSplit TABLE (
    AccountNumberSchemaTypeChar VARCHAR(10),
    Type INT,
    CCY VARCHAR(10),
    UsageType INT,
    AccountID BIGINT
)
AS
BEGIN
    SET @AccountNumberSchemaTypeChar = LEFT(@AccountNumber, 1)
    SET @Type = CAST(SUBSTRING(@AccountNumber, 2, 3) AS INT)
    SET @CCY =  SUBSTRING(@AccountNumber, 5, 3)
    SET @UsageType = CAST(SUBSTRING(@AccountNumber, 8, 2) AS INT)
    SET @AccountID = CAST(RIGHT(@AccountNumber, LEN(@AccountNumber) - 9) AS BIGINT)

    INSERT @AccountSplit VALUES(@AccountNumberSchemaTypeChar, @Type, @CCY, @UsageType, @AccountID)
    RETURN
END
```

### Your Task

```csharp
// Return a tuple instead of a table row
static (string SchemaTypeChar, int Type, string CCY, int UsageType, long AccountID)
    ParseAccountNumber(string accountNumber)
{
    // TODO:
    // Position 0: SchemaTypeChar (1 char)
    // Position 1-3: Type (3 chars → parse to int)
    // Position 4-6: CCY (3 chars)
    // Position 7-8: UsageType (2 chars → parse to int)
    // Position 9+: AccountID (rest → parse to long)
    //
    // Remember: C# strings are 0-based, T-SQL SUBSTRING is 1-based!
}
```

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `LEFT(@s, 1)` | `s[..1]` or `s.Substring(0, 1)` |
| `SUBSTRING(@s, 2, 3)` | `s.Substring(1, 3)` (0-based!) |
| `CAST(... AS INT)` | `int.Parse(...)` |
| `CAST(... AS BIGINT)` | `long.Parse(...)` |
| Returns a TABLE row | Returns a **tuple** |

### Test Cases — Implement in `ParseAccountNumberTests.cs`

These two functions are **inverses** of each other — the most important tests are round-trips.

| # | Test case | Input (`accountNumber`) | Expected |
|---|---|---|---|
| 1 | Extracts schema type (first char) | `"AC01UPG00000000012345"` | SchemaTypeChar = `"A"` |
| 2 | Extracts CCY | `"AC01UPG00000000012345"` | CCY = `"UPG"` |
| 3 | Extracts account ID | `"AC01UPG00000000012345"` | AccountID = `12345` |
| 4 | Round-trip: accountId = 1 | Generate(1) then Parse | AccountID = `1` |
| 5 | Round-trip: accountId = 100 | Generate(100) then Parse | AccountID = `100` |
| 6 | Round-trip: accountId = 54321 | Generate(54321) then Parse | AccountID = `54321` |
| 7 | Round-trip: accountId = 999999 | Generate(999999) then Parse | AccountID = `999999` |
| 8 | Round-trip preserves CCY = "UPG" | Generate(1, "UPG") then Parse | CCY = `"UPG"` |
| 9 | Round-trip preserves CCY = "GEL" | Generate(1, "GEL") then Parse | CCY = `"GEL"` |
| 10 | Round-trip preserves CCY = "USD" | Generate(1, "USD") then Parse | CCY = `"USD"` |
| 11 | Round-trip preserves CCY = "EUR" | Generate(1, "EUR") then Parse | CCY = `"EUR"` |
| 12 | Round-trip preserves all fields | Generate(42, "EUR", 5, 3, "M", "A") then Parse | SchemaTypeChar=`"A"`, CCY=`"EUR"`, AccountID=`42` |

> **Note:** Look carefully at the position offsets. The `Type` field in the original SQL starts at position 2 (1-based) which includes the AccountInitial character. Make sure your parsing aligns with how the number was generated.

---

## Exercise 8: Split String — `hp.fnSplitString`

**Concepts:** `while` loop, `IndexOf`, `Substring`, building a result collection (use `string[]`)

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnSplitString.sql
CREATE FUNCTION hp.[fnSplitString]
(
    @strin NVARCHAR(MAX),
    @seperator VARCHAR(4)
)
RETURNS @strintable TABLE ( Strval NVARCHAR(MAX) )
AS
BEGIN
    DECLARE
        @LocalSeperator VARCHAR(4) = '#',
        @CharIndex INT

    IF SUBSTRING(@strin, LEN(@strin), 1) <> @seperator
        SET @strin = @strin + @seperator

    SET @strin = REPLACE(@strin, @seperator, @LocalSeperator)

    SET @CharIndex = CHARINDEX(@LocalSeperator, @strin, 0)

    WHILE LEN(@strin) <> 0 BEGIN
        SET @CharIndex = CHARINDEX(@LocalSeperator, @strin, 0)

        INSERT @strintable
        VALUES ( LEFT(@strin, @CharIndex - 1) )

        SET @strin = RIGHT(@strin, LEN(@strin) - @CharIndex)
    END

    RETURN
END
```

### Your Task

```csharp
static string[] SplitString(string input, string separator)
{
    // TODO:
    // 1. If the string doesn't end with the separator, append it
    // 2. Use a while loop to find each separator position (IndexOf)
    // 3. Extract the substring before the separator
    // 4. Move past the separator, repeat
    // 5. Return all parts as a string array
    //
    // Use a List<string> to collect results, then .ToArray()
    // (List is like a growable array — we'll cover it properly in Session 5,
    //  but the pattern is: var list = new List<string>(); list.Add(item); list.ToArray())
}
```

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `CHARINDEX(@sep, @str, 0)` | `str.IndexOf(sep)` |
| `LEFT(@str, n)` | `str.Substring(0, n)` or `str[..n]` |
| `RIGHT(@str, LEN(@str) - n)` | `str.Substring(n)` or `str[n..]` |
| `LEN(@str) <> 0` | `str.Length > 0` |
| Returns TABLE rows | Returns `string[]` |

**After you're done:** Compare your implementation with C#'s built-in `"GEL,USD,EUR".Split(',')`. Yes, C# has this built-in! But writing it manually teaches you the loop + string manipulation pattern that the SQL codebase uses everywhere.

### Test Cases — Implement in `SplitStringTests.cs`

| # | Test case | Input (`input`, `separator`) | Expected |
|---|---|---|---|
| 1 | Comma-separated currencies | `"GEL,USD,EUR"`, `","` | `["GEL", "USD", "EUR"]` |
| 2 | Multi-char separator | `"Hello::World::Test"`, `"::"` | `["Hello", "World", "Test"]` |
| 3 | Single value (no separator found) | `"SingleValue"`, `","` | `["SingleValue"]` |
| 4 | Two elements | `"A,B"`, `","` | `["A", "B"]` |
| 5 | Trailing separator doesn't create empty | `"A,B,"`, `","` | `["A", "B"]` |
| 6 | Semicolon separator | `"one;two;three"`, `";"` | `["one", "two", "three"]` |
| 7 | Matches C#'s built-in Split | `"alpha,beta,gamma,delta"`, `","` | same as `"alpha,beta,gamma,delta".Split(',')` |

---

## Exercise 9: Extract Template Placeholders — `dbo.ExtractPlaceholders`

**Concepts:** `while(true)` + `break`, `IndexOf`, `Substring`, string parsing

### Original T-SQL

```sql
-- Source: HSP/dbo/Functions/ExtractPlaceholders.sql
CREATE FUNCTION dbo.ExtractPlaceholders
(
    @text NVARCHAR(MAX)
)
RETURNS @Results TABLE (Placeholder NVARCHAR(255))
AS
BEGIN
    DECLARE @start INT = 1
    DECLARE @open INT
    DECLARE @close INT
    DECLARE @var NVARCHAR(255)

    IF @text IS NOT NULL
        WHILE 1 = 1
        BEGIN
            SET @open = CHARINDEX('{$', @text, @start)
            IF @open = 0 BREAK

            SET @close = CHARINDEX('}', @text, @open)
            IF @close = 0 BREAK

            SET @var = SUBSTRING(@text, @open + 2, @close - @open - 2)

            INSERT INTO @Results(Placeholder) VALUES (@var)

            SET @start = @close + 1
        END

    RETURN
END
```

### Your Task

```csharp
static string[] ExtractPlaceholders(string? text)
{
    // TODO:
    // 1. If text is null, return empty array
    // 2. Use while(true) with break — same as SQL's WHILE 1=1 + BREAK
    // 3. Find "{$" using IndexOf (with startIndex)
    // 4. If not found, break
    // 5. Find "}" after the opening marker
    // 6. If not found, break
    // 7. Extract the placeholder name between {$ and }
    // 8. Move start position past the closing }
    // 9. Return all found placeholders
}
```

**Key Mappings:**
| T-SQL | C# |
|---|---|
| `WHILE 1 = 1 BEGIN ... END` | `while (true) { ... }` |
| `CHARINDEX('{$', @text, @start)` | `text.IndexOf("{$", start)` |
| `IF @open = 0 BREAK` | `if (open == -1) break;` (C# returns -1, not 0) |
| `SUBSTRING(@text, @open + 2, @close - @open - 2)` | `text.Substring(open + 2, close - open - 2)` |

### Test Cases — Implement in `ExtractPlaceholdersTests.cs`

| # | Test case | Input (`text`) | Expected |
|---|---|---|---|
| 1 | Two placeholders in text | `"Hello {$name}, balance is {$amount} GEL"` | `["name", "amount"]` |
| 2 | No placeholders | `"No placeholders here"` | empty array |
| 3 | Three placeholders in order | `"{$first} and {$second} and {$third}"` | `["first", "second", "third"]` |
| 4 | Null input | `null` | empty array |
| 5 | Empty string | `""` | empty array |
| 6 | Adjacent placeholders | `"{$a}{$b}"` | `["a", "b"]` |
| 7 | Incomplete marker (no closing brace) | `"Hello {$name"` | empty array |
| 8 | Dollar without opening brace is ignored | `"Hello $name, your {$balance}"` | `["balance"]` |

---

## Exercise 10 (Challenge): Fee Calculation — `hp.fnCalcFeeAmount`

**Concepts:** `if/else`, `Math.Min`/`Math.Max`, null-coalescing `??`, business logic

### Original T-SQL

```sql
-- Source: HSP/hp/Functions/fnCalcFeeAmount.sql
-- Simplified: the original reads fee config from a database table.
-- For this exercise, we pass the fee config as parameters directly.

-- Original logic (after the DB lookup):
--   IF @FeeFixedValue > 0
--       SET @AmountFee = @FeeFixedValue
--   ELSE
--       SET @AmountFee = @InAmount * @FeePercent / 100
--
--   IF @AmountFee < @FeeMinValue SET @AmountFee = @FeeMinValue
--   IF @AmountFee > @FeeMaxValue SET @AmountFee = @FeeMaxValue
```

### Your Task

In the real system, fee configuration comes from the `hp.TariffsByAmount` table. Since we don't have database access yet, we pass the tariff values as method parameters:

```csharp
static decimal CalcFeeAmount(
    decimal inAmount,
    decimal feeFixedValue,
    decimal feePercent,
    decimal feeMinValue,
    decimal feeMaxValue)
{
    // TODO:
    // 1. If feeFixedValue > 0, the fee IS the fixed value
    // 2. Otherwise, fee = inAmount * feePercent / 100
    // 3. Clamp: if fee < feeMinValue, use feeMinValue
    // 4. Clamp: if fee > feeMaxValue, use feeMaxValue
    // 5. Return the final fee
    //
    // Hint: Math.Max(feeMinValue, Math.Min(feeMaxValue, fee)) does both clamps in one line
}
```

### Test Cases — Implement in `CalcFeeAmountTests.cs`

| # | Test case | Input (`inAmount`, `fixedFee`, `percent`, `min`, `max`) | Expected |
|---|---|---|---|
| 1 | Fixed fee overrides percentage | `1000`, `5.00`, `1.0`, `0`, `100` | `5.00` |
| 2 | Percentage fee within range | `1000`, `0`, `2.0`, `1`, `50` | `20.00` |
| 3 | Percentage fee clamped to minimum | `10`, `0`, `2.0`, `1`, `50` | `1.00` |
| 4 | Percentage fee clamped to maximum | `5000`, `0`, `2.0`, `1`, `50` | `50.00` |
| 5 | Zero percent, zero min = zero fee | `1000`, `0`, `0`, `0`, `100` | `0.00` |
| 6 | Exactly at minimum boundary | `100`, `0`, `1.0`, `1`, `50` | `1.00` |
| 7 | Exactly at maximum boundary | `1000`, `0`, `5.0`, `1`, `50` | `50.00` |
| 8 | 1.5% of 100 | `100`, `0`, `1.5`, `0`, `1000` | `1.50` |
| 9 | 3% of 200 | `200`, `0`, `3.0`, `0`, `1000` | `6.00` |
| 10 | Fixed 10 overrides 2% of 500 | `500`, `10`, `2.0`, `0`, `1000` | `10.00` |

---

## Bonus Exercise: IBAN Generator — `dbo.generate_iban`

**Concepts:** Method composition (calls `IbanSetKey` which calls `IbanMod97`), `PadLeft`, string building

### Original T-SQL

```sql
-- Source: HSP/dbo/Functions/generate_iban.sql
-- Simplified: bank code comes from config table, we pass it as parameter
CREATE FUNCTION dbo.generate_iban (
    @account VARCHAR(20),
    @dept_no INT,
    @bal_acc_alt DECIMAL(12,6),
    @iban_code VARCHAR(2) = 'XX'  -- normally from hp.AppConfigs
)
RETURNS varchar(22)
AS
BEGIN
    DECLARE @iban varchar(22), @acc14 varchar(14), @dept varchar(2)

    IF LTRIM(ISNULL(@iban_code,'')) = ''
        SET @iban_code = 'XX'

    SET @acc14 = CONVERT(varchar(14), @account)
    SET @acc14 = @acc14 + CONVERT(VARCHAR(4), FLOOR(@bal_acc_alt))

    SET @acc14 = REPLICATE('0', 14 - LEN(@acc14)) + @acc14

    SET @dept = CONVERT(varchar(2), @dept_no % 100)
    SET @dept = REPLICATE('0', 2 - LEN(@dept)) + @dept

    SET @iban = 'GE00' + @iban_code + @dept + @acc14

    SET @iban = dbo.iban_set_key(@iban)

    RETURN @iban
END;
```

### Your Task

```csharp
static string? GenerateIban(
    string account,
    int deptNo,
    decimal balAccAlt,
    string ibanCode = "XX")
{
    // TODO:
    // 1. Default ibanCode to "XX" if null or whitespace
    // 2. Build acc14: account + Floor(balAccAlt) as string, padded to 14 chars with leading zeros
    // 3. Build dept: deptNo % 100 as string, padded to 2 chars with leading zeros
    // 4. Assemble: "GE00" + ibanCode + dept + acc14
    // 5. Call IbanSetKey() to calculate the correct check digits
    // 6. Return the final IBAN
    //
    // This exercise chains: GenerateIban → IbanSetKey → IbanMod97
    // Three levels of method composition!
}
```

**Hint:** `Math.Floor(balAccAlt)` gives you the integer part. Convert to int first, then to string.

### Test Cases — Implement in `GenerateIbanTests.cs`

| # | Test case | Input (`account`, `deptNo`, `balAccAlt`, `ibanCode`) | Expected |
|---|---|---|---|
| 1 | Valid input returns 22-char string | `"12345678"`, `1`, `1234.56`, `"TB"` | result length = 22 |
| 2 | Result starts with "GE" | `"12345678"`, `1`, `1234.56`, `"TB"` | starts with `"GE"` |
| 3 | Bank code at positions 4-5 | `"12345678"`, `1`, `1234.56`, `"TB"` | result[4..6] = `"TB"` |
| 4 | Default bank code is "XX" | `"12345678"`, `1`, `1234.56` *(no ibanCode)* | result[4..6] = `"XX"` |
| 5 | Empty bank code falls back to "XX" | `"12345678"`, `1`, `1234.56`, `""` | result[4..6] = `"XX"` |
| 6 | Dept modulo: 1 and 101 produce same dept | `"12345678"`, `1` vs `101`, same others | result[6..8] are equal |
| 7 | Deterministic (same input = same output) | call twice with same args | both results are equal |
| 8 | Check digits are valid (self-validation) | call GenerateIban, then IbanSetKey on result | IbanSetKey returns the same IBAN |

---

## Quick Reference: T-SQL → C# Mapping

| T-SQL | C# Equivalent |
|---|---|
| `CASE WHEN ... THEN ... END` | `switch` expression |
| `BETWEEN x AND y` | `>= x and <= y` (pattern) |
| `IN (1, 2, 3)` | `1 or 2 or 3` (switch pattern) |
| `ISNULL(@x, default)` | `x ?? defaultValue` |
| `IIF(cond, a, b)` | `cond ? a : b` |
| `WHILE ... BEGIN ... END` | `while (...) { ... }` / `for (...)` |
| `WHILE 1=1 BEGIN ... BREAK ... END` | `while (true) { ... break; }` |
| `LEN(@s)` | `s.Length` |
| `SUBSTRING(@s, start, len)` | `s.Substring(start - 1, len)` (0-based!) |
| `LEFT(@s, n)` | `s[..n]` or `s.Substring(0, n)` |
| `RIGHT(@s, n)` | `s[^n..]` or `s.Substring(s.Length - n)` |
| `CHARINDEX(@find, @s, @start)` | `s.IndexOf(find, start)` (returns -1, not 0) |
| `REPLACE(@s, old, new)` | `s.Replace(old, new)` |
| `UPPER(@s)` | `s.ToUpper()` |
| `REPLICATE('0', n) + @s` | `s.PadLeft(n + s.Length, '0')` or `s.PadLeft(totalLen, '0')` |
| `CONVERT(varchar, @num)` | `num.ToString()` |
| `CAST(@s AS INT)` | `int.Parse(s)` |
| `ASCII(@ch) - ASCII('0')` | `ch - '0'` |
| `ROUND(@x, n)` | `Math.Round(x, n)` |
| `FLOOR(@x)` | `Math.Floor(x)` |
| Returns TABLE row | Returns a **tuple** `(T1, T2, ...)` |
| `RETURN @value` | `return value;` |
