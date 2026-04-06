# Classes and Objects in C# — A SQL Developer's Guide

## The Big Mental Shift

In SQL you think in **tables and rows**. A table defines what columns exist; a row holds actual data. In C#, a **class** defines what data and behavior exist; an **object** is an actual instance with real values.

| SQL Concept | C# Concept |
|---|---|
| `CREATE TABLE` definition | `class` definition |
| A row in the table | An **object** (instance of the class) |
| Columns | **Properties** (or fields) |
| Computed columns | **Read-only properties** with logic |
| Constraints (NOT NULL, DEFAULT, CHECK) | **Constructors** + **validation logic** |
| Stored procedures that operate on the table | **Methods** inside the class |
| Schema (`dbo`, `hp`, `hw`) | **Namespace** |

The critical difference: **in SQL, data and behavior are separate** (tables hold data, procedures operate on data). **In C#, data and behavior live together** inside a class.

---

## From Table to Class

### The SQL Table

Here's a simplified version of the real `hw.Accounts` table in our HSP database:

```sql
CREATE TABLE hw.Accounts (
    AccountID       BIGINT IDENTITY(1,1) PRIMARY KEY,
    CustomerID      INT,
    AccountNumber   VARCHAR(30),
    CCY             VARCHAR(5)    DEFAULT 'GEL',
    AccountState    BIGINT        DEFAULT 0,
    Balance         DECIMAL(28,6) DEFAULT 0,
    BlockedAmount   DECIMAL(28,6) DEFAULT 0,
    AccountName     NVARCHAR(2048),
    CreateTime      DATETIME2     DEFAULT SYSDATETIME()
);
```

### The C# Class

```csharp
public class BankAccount
{
    // Properties (like columns)
    public long AccountId { get; set; }
    public int? CustomerId { get; set; }
    public string? AccountNumber { get; set; }
    public string CCY { get; set; } = "GEL";           // default value
    public long AccountState { get; set; } = 0;         // default value
    public decimal Balance { get; set; } = 0m;           // default value
    public decimal BlockedAmount { get; set; } = 0m;     // default value
    public string? AccountName { get; set; }
    public DateTime CreateTime { get; set; } = DateTime.Now;
}
```

### Creating Objects (Like Inserting Rows)

```sql
-- SQL: INSERT a row
INSERT INTO hw.Accounts (CustomerID, AccountNumber, CCY, AccountName)
VALUES (42, 'AC01GEL00000000012345', 'GEL', 'Main Account');
```

```csharp
// C#: Create an object
BankAccount account = new BankAccount
{
    CustomerId = 42,
    AccountNumber = "AC01GEL00000000012345",
    CCY = "GEL",
    AccountName = "Main Account"
};
// AccountId, Balance, BlockedAmount, AccountState get their default values
// CreateTime defaults to DateTime.Now
```

---

## But a Class Can Also Have Methods

This is where classes go beyond tables. A table is just data. A class is **data + behavior**:

```csharp
public class BankAccount
{
    public long AccountId { get; set; }
    public string CCY { get; set; } = "GEL";
    public decimal Balance { get; set; } = 0m;
    public decimal BlockedAmount { get; set; } = 0m;

    // Method — behavior that belongs to this account
    public decimal GetAvailableBalance()
    {
        return Balance - BlockedAmount;
    }
}
```

Compare with the HSP function that does the same thing separately:

```sql
-- SQL: behavior is SEPARATE from the table
CREATE FUNCTION hp.fnGetAccountAvailableBalanceByAccID (@AccountID BIGINT)
RETURNS DECIMAL
AS BEGIN
    DECLARE @Amount DECIMAL;
    SELECT @Amount = a.Balance - a.BlockedAmount
    FROM hw.Accounts a
    WHERE a.AccountID = @AccountID;
    RETURN @Amount;
END;
```

In C#, the method lives inside the class, so it can directly access the object's own data:

```csharp
BankAccount account = new BankAccount { Balance = 1000m, BlockedAmount = 200m };
decimal available = account.GetAvailableBalance();  // 800m
```

> **Key insight:** In SQL, `hp.fnGetAccountAvailableBalanceByAccID(@AccountID)` needs to query the table to find the row. In C#, `account.GetAvailableBalance()` already has the data — it **is** the row.

---

## Object Initializer Syntax

C# gives you several ways to set up an object. The **object initializer** is the most common:

```csharp
// Object initializer — like a named INSERT with specific columns
var account = new BankAccount
{
    CustomerId = 42,
    CCY = "USD",
    Balance = 5000m,
    AccountName = "USD Savings"
};
// Properties not listed keep their default values
```

This is similar to SQL's `INSERT` where you only specify some columns and the rest get defaults:

```sql
INSERT INTO hw.Accounts (CustomerID, CCY, Balance, AccountName)
VALUES (42, 'USD', 5000, 'USD Savings');
-- AccountState, BlockedAmount, etc. get DEFAULT values
```

---

## Multiple Objects = Like Multiple Rows

```csharp
// Create a list of accounts — like a result set
List<BankAccount> accounts = new List<BankAccount>
{
    new BankAccount { AccountId = 1, CCY = "GEL", Balance = 1000m },
    new BankAccount { AccountId = 2, CCY = "USD", Balance = 5000m },
    new BankAccount { AccountId = 3, CCY = "EUR", Balance = 2500m }
};

// Iterate like processing a result set
foreach (var acct in accounts)
{
    Console.WriteLine($"Account {acct.AccountId}: {acct.Balance:N2} {acct.CCY}");
}
```

---

## Reference Types — Objects Live on the Heap

This is a concept that has no SQL equivalent and is important to understand.

When you create an object, the variable holds a **reference** (address) to the object, not the object itself:

```csharp
BankAccount a = new BankAccount { Balance = 1000m };
BankAccount b = a;       // b points to the SAME object, not a copy!

b.Balance = 2000m;
Console.WriteLine(a.Balance);  // 2000m — both a and b are the same object!
```

This is like two variables pointing to the same row. If you change it through one, the other sees the change.

To create an independent copy, you need to create a new object:

```csharp
BankAccount a = new BankAccount { Balance = 1000m };
BankAccount b = new BankAccount { Balance = a.Balance };  // separate object

b.Balance = 2000m;
Console.WriteLine(a.Balance);  // 1000m — a is unchanged
```

---

## The `new` Keyword

Every object is created with `new`. This allocates memory and runs the constructor:

```csharp
// Full syntax
BankAccount account = new BankAccount();

// With type inference (var)
var account = new BankAccount();

// Target-typed new (C# 9+) — when the type is obvious from the left side
BankAccount account = new();

// With object initializer
var account = new BankAccount { Balance = 1000m };
```

> **SQL analogy:** `new` is like `INSERT INTO` — it creates a new instance. Without `new`, you have `null` — like a variable that points to no row.

---

## null — The Absent Row

```csharp
BankAccount? account = null;     // no object exists — like an empty result set

if (account == null)
{
    Console.WriteLine("No account found");  // this prints
}

// Null-conditional — safe access
decimal? balance = account?.Balance;   // null, doesn't throw an exception

// Create the object
account = new BankAccount { Balance = 500m };
balance = account?.Balance;            // 500m
```

This maps directly to SQL patterns you already know:

```sql
DECLARE @Balance DECIMAL;
SELECT @Balance = Balance FROM hw.Accounts WHERE AccountID = 999;
-- If no row found, @Balance is NULL

IF @Balance IS NULL
    PRINT 'No account found';
```

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| Define a table with columns | Define a class with properties |
| INSERT a row | Create an object with `new` |
| Columns have DEFAULT values | Properties have default values |
| Stored procedures operate on tables | Methods live inside the class |
| Functions query data from tables | Methods directly access the object's data |
| A row has data only | An object has data AND behavior |
| NULL result set | `null` reference |
