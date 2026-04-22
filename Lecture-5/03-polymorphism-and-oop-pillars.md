# Polymorphism and the Four OOP Pillars

## What Is Polymorphism?

**Polymorphism** means "many forms." In C#, it means you can treat different objects through a **common interface or base class**, and each object responds with its own behavior.

> **SQL analogy:** Imagine you have a procedure `hp.spExecuteTransaction @TransactionID BIGINT`. Internally, it checks the transaction type and runs completely different logic for deposits vs. withdrawals vs. transfers. Polymorphism lets you do this **without the IF/ELSE chain** — the correct logic runs automatically based on the object's type.

---

## Polymorphism in Action

### The HSP Way (IF/ELSE chains)

```sql
-- HSP: determine behavior based on type column
CREATE PROCEDURE hp.spExecuteTransaction @TransactionID BIGINT
AS BEGIN
    DECLARE @Type VARCHAR(20);
    SELECT @Type = TransactionType FROM hp.Transactions WHERE ID = @TransactionID;

    IF @Type = 'DEPOSIT'
        EXEC hp.spExecuteDeposit @TransactionID;
    ELSE IF @Type = 'WITHDRAWAL'
        EXEC hp.spExecuteWithdrawal @TransactionID;
    ELSE IF @Type = 'TRANSFER'
        EXEC hp.spExecuteTransfer @TransactionID;
    ELSE IF @Type = 'FEE'
        EXEC hp.spExecuteFeeCharge @TransactionID;
    -- Every new type = another ELSE IF
END;
```

### The C# Way (Polymorphism)

```csharp
// Base class or interface defines the contract
public abstract class Transaction
{
    public long Id { get; }
    public decimal Amount { get; }
    public string CCY { get; }

    protected Transaction(long id, decimal amount, string ccy)
    {
        Id = id;
        Amount = amount;
        CCY = ccy;
    }

    // Each derived class implements its own version
    public abstract bool Execute();
    public abstract string TransactionType { get; }
}

public class DepositTransaction : Transaction
{
    private readonly BankAccount _account;

    public DepositTransaction(long id, decimal amount, string ccy, BankAccount account)
        : base(id, amount, ccy) => _account = account;

    public override string TransactionType => "DEPOSIT";

    public override bool Execute()
    {
        _account.Deposit(Amount);
        return true;
    }
}

public class WithdrawalTransaction : Transaction
{
    private readonly BankAccount _account;

    public WithdrawalTransaction(long id, decimal amount, string ccy, BankAccount account)
        : base(id, amount, ccy) => _account = account;

    public override string TransactionType => "WITHDRAWAL";

    public override bool Execute()
    {
        return _account.Withdraw(Amount);
    }
}

public class TransferTransaction : Transaction
{
    private readonly BankAccount _from;
    private readonly BankAccount _to;

    public TransferTransaction(long id, decimal amount, string ccy,
                               BankAccount from, BankAccount to)
        : base(id, amount, ccy)
    {
        _from = from;
        _to = to;
    }

    public override string TransactionType => "TRANSFER";

    public override bool Execute()
    {
        if (!_from.Withdraw(Amount))
            return false;
        _to.Deposit(Amount);
        return true;
    }
}
```

Now the "execute transaction" code is trivially simple:

```csharp
// No IF/ELSE — polymorphism handles it
Transaction transaction = GetTransaction(transactionId);
bool success = transaction.Execute();   // correct version runs automatically!
```

You can even process a batch:

```csharp
List<Transaction> batch = GetPendingTransactions();

foreach (Transaction tx in batch)
{
    bool ok = tx.Execute();   // each transaction knows how to execute itself
    Console.WriteLine($"{tx.TransactionType} #{tx.Id}: {(ok ? "OK" : "FAILED")}");
}
```

Whether the list contains deposits, withdrawals, or transfers — the loop is the same. Each object does the right thing. **That's polymorphism.**

---

## Polymorphism Through Interfaces

Polymorphism also works through interfaces:

```csharp
public interface INotificationSender
{
    bool Send(string recipient, string message);
    string Channel { get; }
}

public class SmsNotifier : INotificationSender
{
    public string Channel => "SMS";
    public bool Send(string recipient, string message)
    {
        Console.WriteLine($"SMS to {recipient}: {message}");
        return true;
    }
}

public class EmailNotifier : INotificationSender
{
    public string Channel => "Email";
    public bool Send(string recipient, string message)
    {
        Console.WriteLine($"Email to {recipient}: {message}");
        return true;
    }
}

public class PushNotifier : INotificationSender
{
    public string Channel => "Push";
    public bool Send(string recipient, string message)
    {
        Console.WriteLine($"Push to {recipient}: {message}");
        return true;
    }
}
```

```csharp
// Send notification through ALL channels — polymorphism in action
List<INotificationSender> senders = new()
{
    new SmsNotifier(),
    new EmailNotifier(),
    new PushNotifier()
};

foreach (INotificationSender sender in senders)
{
    sender.Send("customer@bank.ge", "Your transaction is complete.");
}
// Each sender uses its own implementation of Send()
```

---

## The Four OOP Pillars

Object-Oriented Programming is built on four fundamental principles. You've now seen all of them:

### Pillar 1: Encapsulation (Session 4)

**Hiding internal details, exposing a controlled interface.**

```csharp
public class BankAccount
{
    public decimal Balance { get; private set; }    // can't modify from outside
    public void Deposit(decimal amount) { ... }     // controlled access
    private bool CanWithdraw(decimal amount) { ... } // hidden implementation
}
```

> **SQL analogy:** A stored procedure hides the complex SQL behind a simple `EXEC` call. The caller doesn't see the joins, temp tables, or cursors inside.

---

### Pillar 2: Abstraction

**Showing only the essential features, hiding the complexity.**

```csharp
// The caller sees a simple interface
public interface IPaymentProcessor
{
    bool Process(decimal amount, string ccy);
}

// The complexity is hidden inside each implementation
public class CardPaymentProcessor : IPaymentProcessor
{
    public bool Process(decimal amount, string ccy)
    {
        // Internally: validate card, check fraud, authorize with bank,
        // handle 3DS, log transaction, update limits, send notification...
        // But the caller just sees: Process(amount, ccy) → bool
        return true;
    }
}
```

> **SQL analogy:** When you call `EXEC hp.spProcessPayment @Amount, @CCY`, you don't need to know it internally runs 15 sub-procedures, creates temp tables, and sends emails. The abstraction is the simple procedure signature.

---

### Pillar 3: Inheritance (This Session)

**Creating new classes by extending existing ones — reuse and specialize.**

```csharp
public class FinancialProduct { /* shared data and behavior */ }
public class DepositAccount : FinancialProduct { /* adds interest, maturity */ }
public class LoanAccount : FinancialProduct { /* adds debt, repayment */ }
```

> **SQL analogy:** Like views that extend a base table. Each view inherits all columns and adds its own computed columns or filters.

---

### Pillar 4: Polymorphism (This Session)

**Different objects responding to the same message in their own way.**

```csharp
// One method call, three different behaviors
List<Transaction> transactions = GetTransactions();
foreach (var tx in transactions)
{
    tx.Execute();   // deposit deposits, withdrawal withdraws, transfer transfers
}
```

> **SQL analogy:** Like having `EXEC @DynamicProcName @Params` where the procedure that runs depends on the type — but in C# it's type-safe and checked at compile time, not a string-based dynamic call.

---

## How the Pillars Work Together

Here's a complete example that uses all four pillars:

```csharp
// ABSTRACTION: simple interface hides complexity
public interface ITransactionValidator
{
    bool Validate(Transaction transaction, BankAccount account);
}

// ENCAPSULATION: internal rules are hidden
public class StandardValidator : ITransactionValidator
{
    private const decimal DailyLimit = 100_000m;   // private — hidden detail

    public bool Validate(Transaction transaction, BankAccount account)
    {
        return transaction.Amount > 0
            && transaction.Amount <= DailyLimit
            && account.AvailableBalance >= transaction.Amount;
    }
}

// INHERITANCE: shared structure
public abstract class Transaction
{
    public decimal Amount { get; }
    public abstract bool Execute();
    protected Transaction(decimal amount) => Amount = amount;
}

// POLYMORPHISM: each type executes differently
public class DepositTransaction : Transaction
{
    public override bool Execute() { /* deposit logic */ return true; }
    public DepositTransaction(decimal amount) : base(amount) { }
}

public class WithdrawalTransaction : Transaction
{
    public override bool Execute() { /* withdrawal logic */ return true; }
    public WithdrawalTransaction(decimal amount) : base(amount) { }
}
```

```csharp
// Using it all together
ITransactionValidator validator = new StandardValidator();  // ABSTRACTION
List<Transaction> batch = GetPendingTransactions();          // INHERITANCE

foreach (var tx in batch)
{
    if (validator.Validate(tx, account))                     // ABSTRACTION
    {
        tx.Execute();                                        // POLYMORPHISM
    }
    // account.Balance is private set → ENCAPSULATION
}
```

---

## Real-World Pattern: Strategy Pattern

The combination of interfaces and polymorphism creates the **Strategy Pattern** — one of the most useful design patterns in banking software:

```csharp
public class TransactionProcessor
{
    private readonly IFeeCalculator _feeCalculator;
    private readonly ITransactionValidator _validator;
    private readonly INotificationSender _notifier;

    // Strategies are injected — can be swapped without changing this class
    public TransactionProcessor(
        IFeeCalculator feeCalculator,
        ITransactionValidator validator,
        INotificationSender notifier)
    {
        _feeCalculator = feeCalculator;
        _validator = validator;
        _notifier = notifier;
    }

    public bool Process(Transaction tx, BankAccount account)
    {
        if (!_validator.Validate(tx, account))
            return false;

        decimal fee = _feeCalculator.Calculate(tx.Amount);
        bool success = tx.Execute();

        if (success)
            _notifier.Send(account.CustomerId.ToString(), $"Transaction {tx.Id} completed.");

        return success;
    }
}
```

Different configurations for different scenarios:

```csharp
// Standard retail banking
var retailProcessor = new TransactionProcessor(
    new PercentFeeCalculator(0.5m, 0.10m, 50m),
    new StandardValidator(),
    new SmsNotifier()
);

// VIP banking — no fees, higher limits, email
var vipProcessor = new TransactionProcessor(
    new FixedFeeCalculator(0m),
    new VipValidator(),        // higher limits
    new EmailNotifier()
);

// Same TransactionProcessor class, completely different behavior!
```

---

## Summary

| Concept | What It Does | SQL Equivalent |
|---|---|---|
| Encapsulation | Hides internals, exposes interface | Stored procedure hiding complex SQL |
| Abstraction | Simplifies complex systems | Simple EXEC call for complex operations |
| Inheritance | Reuses and extends existing classes | Views extending base tables |
| Polymorphism | Same call, different behavior per type | Dynamic procedure dispatch (but type-safe) |
| Strategy Pattern | Swap behavior via interfaces | Configurable stored procedure parameters |
