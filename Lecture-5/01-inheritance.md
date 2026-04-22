# Inheritance in C# — A SQL Developer's Guide

## The Big Idea

In Session 4 you learned that a class is like a table definition. **Inheritance** lets you create a new class that **extends** an existing one — it gets all the parent's properties and methods, and can add its own.

| SQL Concept | C# Concept |
|---|---|
| A **view** that extends a base table with extra columns/logic | A **derived class** that extends a base class |
| A base table shared by multiple views | A **base class** shared by multiple derived classes |
| Common columns across similar tables | Common properties/methods in a base class |
| Table-specific columns | Properties/methods unique to the derived class |

> **Key analogy:** Think of inheritance like creating views on top of a base table. The view inherits all columns from the table and can add computed columns or filter logic. A derived class inherits all members from the base class and can add new ones or change behavior.

---

## Why Inheritance?

In the HSP system, look at these tables — they share many common columns:

```sql
-- These tables share a LOT of common structure
CREATE TABLE hw.Accounts (
    AccountID     BIGINT IDENTITY PRIMARY KEY,
    CustomerID    INT,
    CCY           VARCHAR(5),
    Balance       DECIMAL(28,6),
    AccountState  BIGINT,
    CreateTime    DATETIME2 DEFAULT SYSDATETIME()
);

CREATE TABLE hw.Deposits (
    DepositID     BIGINT IDENTITY PRIMARY KEY,
    CustomerID    INT,
    CCY           VARCHAR(5),
    Balance       DECIMAL(28,6),
    AccountState  BIGINT,
    CreateTime    DATETIME2 DEFAULT SYSDATETIME(),
    -- Deposit-specific columns
    InterestRate  DECIMAL(10,4),
    MaturityDate  DATETIME2
);

CREATE TABLE hw.Loans (
    LoanID        BIGINT IDENTITY PRIMARY KEY,
    CustomerID    INT,
    CCY           VARCHAR(5),
    Balance       DECIMAL(28,6),
    AccountState  BIGINT,
    CreateTime    DATETIME2 DEFAULT SYSDATETIME(),
    -- Loan-specific columns
    InterestRate  DECIMAL(10,4),
    RemainingDebt DECIMAL(28,6)
);
```

In SQL, you can't avoid this repetition. In C#, **inheritance** eliminates it:

```csharp
// Base class — the shared structure
public class FinancialProduct
{
    public long Id { get; }
    public int CustomerId { get; }
    public string CCY { get; }
    public decimal Balance { get; protected set; }
    public long AccountState { get; protected set; }
    public DateTime CreateTime { get; }

    public FinancialProduct(long id, int customerId, string ccy)
    {
        Id = id;
        CustomerId = customerId;
        CCY = ccy;
        Balance = 0m;
        AccountState = 0;
        CreateTime = DateTime.Now;
    }
}

// Derived class — inherits everything + adds its own
public class DepositAccount : FinancialProduct
{
    public decimal InterestRate { get; }
    public DateTime MaturityDate { get; }

    public DepositAccount(long id, int customerId, string ccy,
                          decimal interestRate, DateTime maturityDate)
        : base(id, customerId, ccy)   // calls the parent constructor
    {
        InterestRate = interestRate;
        MaturityDate = maturityDate;
    }
}

// Another derived class
public class LoanAccount : FinancialProduct
{
    public decimal InterestRate { get; }
    public decimal RemainingDebt { get; private set; }

    public LoanAccount(long id, int customerId, string ccy,
                       decimal interestRate, decimal debt)
        : base(id, customerId, ccy)
    {
        InterestRate = interestRate;
        RemainingDebt = debt;
    }
}
```

`DepositAccount` and `LoanAccount` **inherit** `Id`, `CustomerId`, `CCY`, `Balance`, `AccountState`, and `CreateTime` from `FinancialProduct`. They don't need to repeat those properties.

---

## The `: base(...)` Constructor Call

When a derived class is created, the **parent constructor must run first**. You call it with `: base(...)`:

```csharp
public class LoanAccount : FinancialProduct
{
    public decimal InterestRate { get; }

    public LoanAccount(long id, int customerId, string ccy, decimal rate)
        : base(id, customerId, ccy)   // runs FinancialProduct constructor first
    {
        InterestRate = rate;           // then sets LoanAccount-specific data
    }
}
```

> **SQL analogy:** This is like an `INSTEAD OF INSERT` trigger where you first insert into the base table, then insert into the extension table. The parent must be set up before the child adds its own data.

---

## `virtual` and `override` — Changing Inherited Behavior

A base class can mark methods as `virtual`, meaning derived classes **can replace** the behavior with `override`:

```csharp
public class FinancialProduct
{
    public decimal Balance { get; protected set; }

    // virtual = "derived classes CAN override this"
    public virtual string GetStatement()
    {
        return $"Balance: {Balance:N2} {CCY}";
    }
}

public class DepositAccount : FinancialProduct
{
    public decimal InterestRate { get; }
    public DateTime MaturityDate { get; }

    // override = "I'm replacing the parent's version"
    public override string GetStatement()
    {
        return $"Deposit: {Balance:N2} {CCY} @ {InterestRate:P2}, Matures: {MaturityDate:d}";
    }
}

public class LoanAccount : FinancialProduct
{
    public decimal RemainingDebt { get; private set; }

    public override string GetStatement()
    {
        return $"Loan: {Balance:N2} {CCY}, Remaining: {RemainingDebt:N2}";
    }
}
```

```csharp
FinancialProduct product = new DepositAccount(1, 42, "GEL", 0.12m, new DateTime(2026, 12, 31));
Console.WriteLine(product.GetStatement());
// Output: "Deposit: 0.00 GEL @ 12.00%, Matures: 12/31/2026"
// Even though the variable type is FinancialProduct, the DepositAccount version runs!
```

> **SQL analogy:** `virtual`/`override` is like having a base stored procedure that different schemas can override. Think of `hp.spProcessTransaction` being overridden by `hp.spProcessDepositTransaction` — same name, different behavior depending on the type.

---

## `base` — Calling the Parent's Version

Sometimes you want to **extend** the parent's behavior rather than replace it entirely. Use `base.MethodName()`:

```csharp
public class FinancialProduct
{
    public virtual string GetStatement()
    {
        return $"[{Id}] {Balance:N2} {CCY}";
    }
}

public class DepositAccount : FinancialProduct
{
    public override string GetStatement()
    {
        // Call the parent's version first, then add deposit-specific info
        string baseStatement = base.GetStatement();
        return $"{baseStatement} | Rate: {InterestRate:P2}, Maturity: {MaturityDate:d}";
    }
}
```

```csharp
var deposit = new DepositAccount(1, 42, "GEL", 0.10m, new DateTime(2026, 6, 30));
deposit.Balance = 5000m; // assume we set this somehow
Console.WriteLine(deposit.GetStatement());
// "[1] 5,000.00 GEL | Rate: 10.00%, Maturity: 6/30/2026"
```

---

## `abstract` — Forcing Derived Classes to Implement

An `abstract` method has **no body** — derived classes **must** provide the implementation:

```csharp
// abstract class — CANNOT be instantiated directly
public abstract class Transaction
{
    public long TransactionId { get; }
    public DateTime Timestamp { get; }
    public decimal Amount { get; }
    public string CCY { get; }

    protected Transaction(long id, decimal amount, string ccy)
    {
        TransactionId = id;
        Amount = amount;
        CCY = ccy;
        Timestamp = DateTime.Now;
    }

    // abstract = "every derived class MUST implement this"
    public abstract bool Execute();

    // abstract property
    public abstract string TransactionType { get; }

    // Regular method — inherited as-is
    public string GetSummary()
        => $"[{TransactionType}] {Amount:N2} {CCY} at {Timestamp:HH:mm:ss}";
}
```

```csharp
// You CANNOT do this:
// var t = new Transaction(1, 100m, "GEL");   // COMPILER ERROR — abstract!

// You MUST create a concrete derived class:
public class DepositTransaction : Transaction
{
    public BankAccount TargetAccount { get; }

    public DepositTransaction(long id, decimal amount, string ccy, BankAccount target)
        : base(id, amount, ccy)
    {
        TargetAccount = target;
    }

    public override string TransactionType => "DEPOSIT";

    public override bool Execute()
    {
        TargetAccount.Deposit(Amount);
        return true;
    }
}
```

> **SQL analogy:** An `abstract` class is like a table type definition that can never hold data itself — it only defines the structure. Or think of it as a template stored procedure that says "every implementation must have these parameters and return this type" but doesn't provide the body.

---

## `sealed` — Preventing Further Inheritance

If you want to stop a class from being inherited further, mark it `sealed`:

```csharp
public sealed class WireTransfer : Transaction
{
    // No class can inherit from WireTransfer
    public override string TransactionType => "WIRE";
    public override bool Execute() { /* ... */ return true; }
}

// This would NOT compile:
// public class InternationalWireTransfer : WireTransfer { }  // ERROR — sealed!
```

> **SQL analogy:** `sealed` is like a view marked `WITH SCHEMABINDING` — it's locked down, no further modifications to its structure.

---

## The `is` and `as` Keywords — Type Checking

When working with inheritance, you sometimes need to check what type an object actually is:

```csharp
FinancialProduct product = GetProduct(accountId);  // could be any derived type

// 'is' — checks the type (like checking a discriminator column)
if (product is DepositAccount deposit)
{
    Console.WriteLine($"Interest rate: {deposit.InterestRate:P2}");
}
else if (product is LoanAccount loan)
{
    Console.WriteLine($"Remaining debt: {loan.RemainingDebt:N2}");
}

// 'as' — attempts a cast, returns null if it fails
var maybeDeposit = product as DepositAccount;
if (maybeDeposit != null)
{
    Console.WriteLine(maybeDeposit.MaturityDate);
}
```

> **SQL analogy:** This is like checking a `Type` or `Category` column to determine which kind of record you're dealing with:
> ```sql
> IF @ProductType = 'DEPOSIT'
>     SELECT InterestRate, MaturityDate FROM hw.Deposits WHERE ID = @ID
> ELSE IF @ProductType = 'LOAN'
>     SELECT InterestRate, RemainingDebt FROM hw.Loans WHERE ID = @ID
> ```

---

## When to Use Inheritance

| Use Inheritance When | Don't Use Inheritance When |
|---|---|
| There's a clear "is a" relationship (a Deposit **is a** FinancialProduct) | The relationship is "has a" (an Account **has a** Currency, not "is a" Currency) |
| Derived classes share significant common behavior | You just want to reuse one or two methods |
| You want polymorphism (treating different types uniformly) | The types don't form a natural hierarchy |
| The base class represents a real abstraction | You're doing it just to avoid typing |

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| Repeated columns across similar tables | Shared properties in a base class |
| View that extends a table | Derived class that extends a base class |
| Calling a base procedure from another | `base.MethodName()` |
| Template procedure (no body) | `abstract` method |
| Overriding behavior per type | `virtual` / `override` |
| Preventing further extension | `sealed` |
| Checking a type/category column | `is` pattern matching |
