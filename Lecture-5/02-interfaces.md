# Interfaces in C# — Contracts for Behavior

## What Is an Interface?

An **interface** defines a **contract** — a set of methods and properties that a class **promises** to implement. The interface says **what** must exist, but not **how** it works.

| SQL Concept | C# Concept |
|---|---|
| A stored procedure signature (name, parameters, return type) | An interface method signature |
| Multiple procedures with the same signature but different logic | Multiple classes implementing the same interface |
| A standard API contract ("every payment procedure must accept these params") | An interface definition |

> **Key analogy:** In the HSP system, imagine a rule that says "every payment processing procedure must accept `@Amount DECIMAL, @CCY VARCHAR(5), @AccountID BIGINT` and return `BIT` for success/failure." That's an interface — a contract that multiple implementations must follow.

---

## Defining an Interface

Interfaces are defined with the `interface` keyword. By convention, interface names start with `I`:

```csharp
// Interface — the contract
public interface ITransactionProcessor
{
    bool Process(decimal amount, string ccy, long accountId);
    string ProcessorName { get; }
}
```

This says: "Any class that implements `ITransactionProcessor` **must** have a `Process` method and a `ProcessorName` property."

---

## Implementing an Interface

A class implements an interface by listing it after `:` and providing all the required members:

```csharp
public class CardPaymentProcessor : ITransactionProcessor
{
    public string ProcessorName => "Card Payment";

    public bool Process(decimal amount, string ccy, long accountId)
    {
        // Card-specific logic: validate card, check limits, authorize
        Console.WriteLine($"Processing card payment of {amount:N2} {ccy}");
        return true;
    }
}

public class WireTransferProcessor : ITransactionProcessor
{
    public string ProcessorName => "Wire Transfer";

    public bool Process(decimal amount, string ccy, long accountId)
    {
        // Wire-specific logic: validate IBAN, check SWIFT, submit to clearing
        Console.WriteLine($"Processing wire transfer of {amount:N2} {ccy}");
        return true;
    }
}

public class CashProcessor : ITransactionProcessor
{
    public string ProcessorName => "Cash";

    public bool Process(decimal amount, string ccy, long accountId)
    {
        // Cash-specific logic: check denominations, update cash register
        Console.WriteLine($"Processing cash operation of {amount:N2} {ccy}");
        return true;
    }
}
```

Three different classes, all following the same contract. The calling code doesn't need to know **which** processor it's using:

```csharp
ITransactionProcessor processor = GetProcessor(transactionType);
bool success = processor.Process(1000m, "GEL", 42);
// Works regardless of whether it's Card, Wire, or Cash!
```

---

## Why Use Interfaces?

### The HSP Problem

In the HSP system, fee calculation works differently depending on the fee type:

```sql
-- HSP has different fee calculation logic scattered across procedures
CREATE PROCEDURE hp.spCalcFixedFee       @Amount DECIMAL, @FeeConfig INT, ...
CREATE PROCEDURE hp.spCalcPercentFee     @Amount DECIMAL, @FeeConfig INT, ...
CREATE PROCEDURE hp.spCalcTieredFee      @Amount DECIMAL, @FeeConfig INT, ...

-- The calling code has to know which one to call:
IF @FeeType = 1 EXEC hp.spCalcFixedFee ...
ELSE IF @FeeType = 2 EXEC hp.spCalcPercentFee ...
ELSE IF @FeeType = 3 EXEC hp.spCalcTieredFee ...
```

This creates long `IF/ELSE` chains whenever a new fee type is added.

### The C# Solution — Interface

```csharp
// The contract
public interface IFeeCalculator
{
    decimal Calculate(decimal amount);
    string FeeType { get; }
}

// Fixed fee
public class FixedFeeCalculator : IFeeCalculator
{
    private readonly decimal _fixedAmount;

    public FixedFeeCalculator(decimal fixedAmount) => _fixedAmount = fixedAmount;
    public string FeeType => "Fixed";
    public decimal Calculate(decimal amount) => _fixedAmount;
}

// Percentage fee
public class PercentFeeCalculator : IFeeCalculator
{
    private readonly decimal _percent;
    private readonly decimal _min;
    private readonly decimal _max;

    public PercentFeeCalculator(decimal percent, decimal min, decimal max)
    {
        _percent = percent;
        _min = min;
        _max = max;
    }

    public string FeeType => "Percent";

    public decimal Calculate(decimal amount)
    {
        decimal fee = amount * _percent / 100m;
        return Math.Max(_min, Math.Min(_max, fee));
    }
}

// Tiered fee
public class TieredFeeCalculator : IFeeCalculator
{
    public string FeeType => "Tiered";

    public decimal Calculate(decimal amount)
    {
        if (amount <= 1000m) return amount * 0.01m;
        if (amount <= 10000m) return 10m + (amount - 1000m) * 0.005m;
        return 55m + (amount - 10000m) * 0.002m;
    }
}
```

Now the calling code doesn't need `IF/ELSE`:

```csharp
// The code just uses the interface — it doesn't care which implementation
IFeeCalculator calculator = GetFeeCalculator(feeConfigId);
decimal fee = calculator.Calculate(transactionAmount);
```

When you add a new fee type, you create a new class. **No existing code changes.**

---

## Multiple Interface Implementation

Unlike inheritance (one parent only), a class can implement **multiple interfaces**:

```csharp
public interface ILoggable
{
    string ToLogString();
}

public interface IAuditable
{
    string AuditTrail { get; }
    DateTime LastModified { get; }
}

// One class, multiple contracts
public class BankAccount : ILoggable, IAuditable
{
    public long AccountId { get; }
    public decimal Balance { get; private set; }
    public DateTime LastModified { get; private set; }

    public BankAccount(long id)
    {
        AccountId = id;
        LastModified = DateTime.Now;
    }

    // ILoggable implementation
    public string ToLogString() => $"Account {AccountId}: Balance={Balance:N2}";

    // IAuditable implementation
    public string AuditTrail => $"Account {AccountId} last modified {LastModified:G}";
}
```

> **SQL analogy:** This is like a stored procedure that must conform to multiple standards — for example, it must follow the logging standard (write to the audit table) AND the transaction standard (accept specific parameters). Each standard is an interface.

---

## Interface as a Property Type

Interfaces can be used as types for properties, parameters, and collections:

```csharp
public class TransactionService
{
    // The service doesn't know which validator it will use
    private readonly ITransactionValidator _validator;
    private readonly IFeeCalculator _feeCalculator;

    // Dependencies injected through the constructor
    public TransactionService(ITransactionValidator validator, IFeeCalculator feeCalculator)
    {
        _validator = validator;
        _feeCalculator = feeCalculator;
    }

    public bool ProcessTransaction(decimal amount, BankAccount account)
    {
        // Uses the interface — works with ANY implementation
        if (!_validator.Validate(amount, account))
            return false;

        decimal fee = _feeCalculator.Calculate(amount);
        account.Withdraw(amount + fee);
        return true;
    }
}
```

This is **Dependency Injection** — a pattern you'll use extensively in ASP.NET Core (covered in later sessions). The key idea: code depends on **interfaces** (contracts), not **concrete classes**.

---

## Default Interface Methods (C# 8+)

Interfaces can provide a default implementation. Classes can override it or use the default:

```csharp
public interface ITransactionValidator
{
    bool Validate(decimal amount, BankAccount account);

    // Default implementation — classes get this for free
    string ValidationDescription => "Standard transaction validation";
}

public class BasicValidator : ITransactionValidator
{
    public bool Validate(decimal amount, BankAccount account)
    {
        return amount > 0 && account.AvailableBalance >= amount;
    }
    // Uses the default ValidationDescription
}

public class StrictValidator : ITransactionValidator
{
    public bool Validate(decimal amount, BankAccount account)
    {
        return amount > 0
            && amount <= 50000m   // daily limit
            && account.AvailableBalance >= amount;
    }

    // Overrides the default
    public string ValidationDescription => "Strict validation with daily limit";
}
```

---

## Interface vs Abstract Class — When to Use Each

| Feature | Interface | Abstract Class |
|---|---|---|
| Multiple inheritance | A class can implement **many** interfaces | A class can extend **one** base class only |
| Fields/state | Cannot have instance fields | Can have fields and state |
| Constructors | No constructors | Can have constructors |
| Access modifiers on members | All public by default | Can have private, protected |
| Default implementation | Since C# 8 (limited) | Full implementations allowed |
| Use when... | Defining a **capability** ("can do") | Defining a **category** ("is a") |

### Rules of Thumb

```
"Is it a kind of...?"           → Use inheritance (abstract class)
  - DepositTransaction IS A Transaction
  - LoanAccount IS A FinancialProduct

"Can it do...?"                 → Use an interface
  - BankAccount CAN BE logged (ILoggable)
  - Transaction CAN BE validated (ITransactionValidator)
  - Account CAN BE audited (IAuditable)
```

### Combining Both

In practice, you often combine them:

```csharp
// Abstract base class for shared state and behavior
public abstract class Transaction
{
    public long Id { get; }
    public decimal Amount { get; }
    public DateTime Timestamp { get; }
    // ...shared constructor, shared methods...
}

// Interfaces for capabilities
public interface IReversible
{
    bool Reverse();
}

public interface ILoggable
{
    string ToLogString();
}

// Concrete class: IS A Transaction, CAN BE reversed, CAN BE logged
public class WireTransfer : Transaction, IReversible, ILoggable
{
    public bool Reverse() { /* reversal logic */ return true; }
    public string ToLogString() => $"Wire #{Id}: {Amount:N2}";
    // ...
}

// This class IS A Transaction but CANNOT be reversed
public class CashDeposit : Transaction, ILoggable
{
    public string ToLogString() => $"Cash #{Id}: {Amount:N2}";
    // No IReversible — cash deposits can't be reversed
}
```

---

## Summary

| SQL Thinking | C# Thinking |
|---|---|
| "Every payment procedure must accept these parameters" | Interface defines method signatures |
| Different procedures, same parameter contract | Different classes, same interface |
| `IF @Type = 1 EXEC proc1 ELSE EXEC proc2` | Use interface — no IF/ELSE needed |
| A procedure can follow multiple standards | A class can implement multiple interfaces |
| Standard API contract for a family of procedures | `interface IPaymentProcessor { ... }` |
| Choosing between inheritance and interfaces | "Is a" = inheritance, "Can do" = interface |
