# Hands-On Exercises — Session 5: Inheritance, Interfaces, and Polymorphism

## The Goal

Build a **transaction processing system** using inheritance, interfaces, and polymorphism. You'll create a `Transaction` base class, derive specific transaction types, and define interfaces for validation and fee calculation — all based on real HSP banking patterns.

---

## Project Setup

Continue using the same solution from Session 4, or create a new project:

### 1. Class Library (your implementations)

```bash
dotnet new classlib -n HspTransactions
```

Organize your classes — one class per file:

```
HspTransactions/
  Transaction.cs              (abstract base class)
  DepositTransaction.cs       (derived class)
  WithdrawalTransaction.cs    (derived class)
  TransferTransaction.cs      (derived class)
  FeeChargeTransaction.cs     (derived class)
  ITransactionValidator.cs    (interface)
  BasicValidator.cs
  StrictValidator.cs
  IFeeCalculator.cs           (interface)
  FixedFeeCalculator.cs
  PercentFeeCalculator.cs
  TieredFeeCalculator.cs
  TransactionProcessor.cs     (combines everything)
```

Add a reference to your Session 4 `HspModels` project (you'll need `BankAccount`):

```bash
cd HspTransactions
dotnet add reference ../HspModels/HspModels.csproj
```

### 2. xUnit Test Project

```bash
dotnet new xunit -n HspTransactions.Tests
cd HspTransactions.Tests
dotnet add reference ../HspTransactions/HspTransactions.csproj
dotnet add reference ../HspModels/HspModels.csproj
```

Run tests with:
```bash
cd HspTransactions.Tests
dotnet test
```

---

## Exercise 1: Transaction Base Class — Abstract Foundation

**Concepts:** Abstract class, abstract methods, protected constructor, computed properties

### The HSP Table

```sql
-- Source: hp.Transactions (simplified)
CREATE TABLE hp.Transactions (
    TransactionID   BIGINT IDENTITY PRIMARY KEY,
    TransactionType VARCHAR(20) NOT NULL,      -- 'DEPOSIT', 'WITHDRAWAL', 'TRANSFER', 'FEE'
    Amount          DECIMAL(28,6) NOT NULL,
    CCY             VARCHAR(5) NOT NULL,
    Status          VARCHAR(10) DEFAULT 'PENDING', -- 'PENDING', 'COMPLETED', 'FAILED'
    CreateTime      DATETIME2 DEFAULT SYSDATETIME(),
    Description     NVARCHAR(500)
);
```

### Your Task

```csharp
// Transaction.cs
public abstract class Transaction
{
    // TODO: Properties
    // - TransactionId (long): get-only, set in constructor
    // - Amount (decimal): get-only, set in constructor
    // - CCY (string): get-only, set in constructor
    // - Status (string): get + protected set, default "PENDING"
    // - CreateTime (DateTime): get-only, defaults to DateTime.Now
    // - Description (string?): get/set, nullable

    // TODO: Abstract members — derived classes MUST implement
    // - TransactionType (string): abstract get-only property
    // - Execute(): abstract method returning bool

    // TODO: Protected constructor(long id, decimal amount, string ccy)
    //   - Validates: amount must be > 0
    //   - Validates: ccy must not be null/empty and must be 3 chars
    //   - Sets all properties

    // TODO: Concrete methods (inherited by all derived classes)
    // - public void MarkCompleted() → sets Status = "COMPLETED"
    // - public void MarkFailed() → sets Status = "FAILED"
    // - public bool IsPending => Status == "PENDING"

    // TODO: GetSummary() — virtual method (can be overridden)
    //   Returns: "[DEPOSIT] 1,000.00 GEL - PENDING" format
    //   Uses TransactionType, Amount, CCY, Status

    // TODO: Override ToString()
    //   Returns: "Transaction #123: [DEPOSIT] 1,000.00 GEL"
}
```

### Test Cases — Implement in `TransactionTests.cs`

Since `Transaction` is abstract, you can't instantiate it directly. Test through derived classes (Exercise 2), or create a simple test helper:

```csharp
// Test helper — minimal concrete implementation
public class TestTransaction : Transaction
{
    public override string TransactionType => "TEST";
    public override bool Execute() => true;

    public TestTransaction(long id, decimal amount, string ccy)
        : base(id, amount, ccy) { }
}
```

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Cannot instantiate directly | `new Transaction(...)` | Should NOT compile |
| 2 | Properties set correctly | `new TestTransaction(1, 100m, "GEL")` | Id=1, Amount=100, CCY="GEL" |
| 3 | Default status is PENDING | new TestTransaction | Status = `"PENDING"` |
| 4 | IsPending returns true initially | new TestTransaction | IsPending = `true` |
| 5 | MarkCompleted changes status | call MarkCompleted() | Status = `"COMPLETED"`, IsPending = `false` |
| 6 | MarkFailed changes status | call MarkFailed() | Status = `"FAILED"` |
| 7 | Zero amount throws | `new TestTransaction(1, 0m, "GEL")` | throws ArgumentException |
| 8 | Negative amount throws | `new TestTransaction(1, -100m, "GEL")` | throws ArgumentException |
| 9 | Invalid CCY throws | `new TestTransaction(1, 100m, "AB")` | throws ArgumentException |
| 10 | GetSummary format | TestTransaction(1, 500m, "GEL").GetSummary() | contains `"[TEST]"`, `"500"`, `"GEL"` |
| 11 | CreateTime is set | new TestTransaction | CreateTime close to DateTime.Now |

---

## Exercise 2: Derived Transaction Types — Inheritance in Practice

**Concepts:** Inheritance, `: base(...)`, `override`, type-specific behavior

### Your Tasks

#### DepositTransaction

```csharp
// DepositTransaction.cs
public class DepositTransaction : Transaction
{
    // TODO: Property
    // - TargetAccount (BankAccount): get-only, the account receiving the deposit

    // TODO: Constructor(long id, decimal amount, string ccy, BankAccount targetAccount)
    //   - Calls base(id, amount, ccy)
    //   - Validates: targetAccount must not be null
    //   - Validates: ccy must match targetAccount.CCY

    // TODO: Override TransactionType => "DEPOSIT"

    // TODO: Override Execute()
    //   - Calls TargetAccount.Deposit(Amount)
    //   - Calls MarkCompleted()
    //   - Returns true
}
```

#### WithdrawalTransaction

```csharp
// WithdrawalTransaction.cs
public class WithdrawalTransaction : Transaction
{
    // TODO: Property
    // - SourceAccount (BankAccount): get-only

    // TODO: Constructor(long id, decimal amount, string ccy, BankAccount sourceAccount)
    //   - Calls base(id, amount, ccy)
    //   - Validates: sourceAccount must not be null
    //   - Validates: ccy must match sourceAccount.CCY

    // TODO: Override TransactionType => "WITHDRAWAL"

    // TODO: Override Execute()
    //   - Calls SourceAccount.Withdraw(Amount)
    //   - If withdrawal succeeds: MarkCompleted(), return true
    //   - If withdrawal fails (insufficient funds): MarkFailed(), return false
}
```

#### TransferTransaction

```csharp
// TransferTransaction.cs
public class TransferTransaction : Transaction
{
    // TODO: Properties
    // - SourceAccount (BankAccount): get-only
    // - DestinationAccount (BankAccount): get-only

    // TODO: Constructor(long id, decimal amount, string ccy,
    //                   BankAccount source, BankAccount destination)
    //   - Calls base(id, amount, ccy)
    //   - Validates: both accounts must not be null
    //   - Validates: source and destination must not be the same account
    //   - Validates: ccy must match source account CCY
    //     (for simplicity, only same-currency transfers)

    // TODO: Override TransactionType => "TRANSFER"

    // TODO: Override Execute()
    //   - Withdraw from source
    //   - If withdrawal fails: MarkFailed(), return false
    //   - Deposit to destination
    //   - MarkCompleted(), return true

    // TODO: Override GetSummary()
    //   - Returns: "[TRANSFER] 1,000.00 GEL from ACC001 to ACC002 - COMPLETED"
}
```

### Test Cases — Implement in `DepositTransactionTests.cs`, etc.

**DepositTransaction Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | TransactionType is DEPOSIT | new DepositTransaction | TransactionType = `"DEPOSIT"` |
| 2 | Execute deposits to account | account balance=0, deposit 500 | account.Balance = `500m`, returns `true` |
| 3 | Execute marks completed | after Execute() | Status = `"COMPLETED"` |
| 4 | Null account throws | targetAccount = null | throws ArgumentNullException |
| 5 | Currency mismatch throws | GEL transaction, USD account | throws ArgumentException |

**WithdrawalTransaction Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | TransactionType is WITHDRAWAL | new WithdrawalTransaction | `"WITHDRAWAL"` |
| 2 | Execute withdraws from account | balance=1000, withdraw 400 | Balance=`600m`, returns `true` |
| 3 | Execute fails with insufficient funds | balance=100, withdraw 500 | returns `false`, Balance=`100m` |
| 4 | Failed withdrawal marks FAILED | insufficient funds | Status = `"FAILED"` |
| 5 | Successful withdrawal marks COMPLETED | sufficient funds | Status = `"COMPLETED"` |

**TransferTransaction Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | TransactionType is TRANSFER | new TransferTransaction | `"TRANSFER"` |
| 2 | Execute transfers between accounts | from=1000, to=500, transfer 300 | from=700, to=800, returns `true` |
| 3 | Execute fails when source insufficient | from=100, transfer 500 | returns `false`, both balances unchanged |
| 4 | Same account throws | source == destination | throws ArgumentException |
| 5 | GetSummary includes account numbers | after execute | contains both account numbers |
| 6 | Null source throws | source = null | throws ArgumentNullException |
| 7 | Null destination throws | destination = null | throws ArgumentNullException |

---

## Exercise 3: Transaction Validator Interface

**Concepts:** Interface definition, multiple implementations, polymorphism through interfaces

### The HSP Inspiration

```sql
-- HSP validates transactions differently based on configuration
-- Basic: just check amount > 0 and sufficient balance
-- Strict: also check daily limits and account status
-- VIP: higher limits, fewer restrictions
```

### Your Task

```csharp
// ITransactionValidator.cs
public interface ITransactionValidator
{
    // TODO: Define the contract
    // - bool Validate(Transaction transaction, BankAccount account)
    // - string ValidatorName { get; }
}
```

```csharp
// BasicValidator.cs
public class BasicValidator : ITransactionValidator
{
    // TODO: ValidatorName => "Basic"
    //
    // TODO: Validate logic:
    //   1. transaction must not be null
    //   2. account must not be null
    //   3. transaction.Amount must be > 0
    //   4. For withdrawals/transfers: account.AvailableBalance >= transaction.Amount
    //   5. Return true if all checks pass
}
```

```csharp
// StrictValidator.cs
public class StrictValidator : ITransactionValidator
{
    // TODO: Private field: _dailyLimit (decimal), set in constructor
    //   Default daily limit: 50,000

    // TODO: Constructor(decimal dailyLimit = 50_000m)

    // TODO: ValidatorName => "Strict"
    //
    // TODO: Validate logic:
    //   1. All BasicValidator checks (amount > 0, sufficient balance)
    //   2. PLUS: transaction.Amount must be <= _dailyLimit
    //   3. PLUS: account must be active (account.IsActive must be true)
    //   4. Return true if ALL checks pass
}
```

### Test Cases — Implement in `ValidatorTests.cs`

**BasicValidator Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Valid deposit passes | amount=500, balance=0 | `true` |
| 2 | Valid withdrawal passes | amount=200, balance=1000 | `true` |
| 3 | Insufficient funds fails | amount=500, balance=100 | `false` |
| 4 | Null transaction fails | transaction=null | `false` (or throws) |
| 5 | Null account fails | account=null | `false` (or throws) |
| 6 | ValidatorName is "Basic" | | `"Basic"` |

**StrictValidator Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Under daily limit passes | amount=1000, limit=50000, active account | `true` |
| 2 | Over daily limit fails | amount=60000, limit=50000 | `false` |
| 3 | Exactly at limit passes | amount=50000, limit=50000 | `true` |
| 4 | Inactive account fails | account not activated | `false` |
| 5 | Custom limit respected | limit=100, amount=200 | `false` |
| 6 | ValidatorName is "Strict" | | `"Strict"` |

**Polymorphism Test (use both interchangeably):**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Both validators work through interface | `ITransactionValidator v = new BasicValidator();` | compiles and works |
| 2 | List of validators | `List<ITransactionValidator>` with both types | each validates correctly |

---

## Exercise 4: Fee Calculator Interface

**Concepts:** Interface, multiple implementations, `Math.Max`/`Math.Min`, polymorphism

### The HSP Function

```sql
-- Source: hp.fnCalcFeeAmount (simplified)
-- The HSP system has a single function that uses IF/ELSE for different fee types
CREATE FUNCTION hp.fnCalcFeeAmount(
    @Amount DECIMAL, @FeeType INT, @FixedFee DECIMAL,
    @Percent DECIMAL, @MinFee DECIMAL, @MaxFee DECIMAL
) RETURNS DECIMAL AS BEGIN
    IF @FeeType = 1 RETURN @FixedFee;
    IF @FeeType = 2 RETURN dbo.fnClamp(@Amount * @Percent / 100, @MinFee, @MaxFee);
    IF @FeeType = 3 ...  -- tiered logic
    RETURN 0;
END;
```

In C#, we'll use the interface pattern to eliminate the IF/ELSE:

### Your Task

```csharp
// IFeeCalculator.cs
public interface IFeeCalculator
{
    // TODO: Define the contract
    // - decimal Calculate(decimal transactionAmount)
    // - string FeeType { get; }
}
```

```csharp
// FixedFeeCalculator.cs
public class FixedFeeCalculator : IFeeCalculator
{
    // TODO: Private readonly field _fixedAmount (decimal)
    // TODO: Constructor(decimal fixedAmount) — validate > 0
    // TODO: FeeType => "Fixed"
    // TODO: Calculate(decimal amount) => always returns _fixedAmount
}
```

```csharp
// PercentFeeCalculator.cs
public class PercentFeeCalculator : IFeeCalculator
{
    // TODO: Private readonly fields: _percent, _minFee, _maxFee
    //
    // TODO: Constructor(decimal percent, decimal minFee, decimal maxFee)
    //   - Validates: percent must be > 0
    //   - Validates: minFee must be >= 0
    //   - Validates: maxFee must be >= minFee
    //
    // TODO: FeeType => "Percent"
    //
    // TODO: Calculate(decimal amount)
    //   - fee = amount * _percent / 100
    //   - return Math.Max(_minFee, Math.Min(_maxFee, fee))
    //     (clamp between min and max)
}
```

```csharp
// TieredFeeCalculator.cs
public class TieredFeeCalculator : IFeeCalculator
{
    // TODO: FeeType => "Tiered"
    //
    // TODO: Calculate(decimal amount)
    //   Tier 1: amount <= 1,000 → fee = 1% (amount * 0.01)
    //   Tier 2: 1,000 < amount <= 10,000 → fee = 10 + 0.5% of amount above 1,000
    //   Tier 3: amount > 10,000 → fee = 55 + 0.2% of amount above 10,000
}
```

### Test Cases — Implement in `FeeCalculatorTests.cs`

**FixedFeeCalculator Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Returns fixed amount | fixedAmount=5, Calculate(100) | `5m` |
| 2 | Returns fixed regardless of amount | fixedAmount=5, Calculate(100000) | `5m` |
| 3 | FeeType is "Fixed" | | `"Fixed"` |

**PercentFeeCalculator Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Basic percentage | 2%, min=0, max=1000, amount=1000 | `20m` |
| 2 | Minimum fee applied | 1%, min=5, max=100, amount=100 | `5m` (1% of 100 = 1, but min is 5) |
| 3 | Maximum fee applied | 5%, min=0, max=10, amount=1000 | `10m` (5% of 1000 = 50, but max is 10) |
| 4 | Fee between min and max | 2%, min=1, max=100, amount=500 | `10m` |
| 5 | FeeType is "Percent" | | `"Percent"` |

**TieredFeeCalculator Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Tier 1 | amount=500 | `5m` (500 * 0.01) |
| 2 | Tier 1 boundary | amount=1000 | `10m` (1000 * 0.01) |
| 3 | Tier 2 | amount=5000 | `30m` (10 + (5000-1000)*0.005) |
| 4 | Tier 2 boundary | amount=10000 | `55m` (10 + 9000*0.005) |
| 5 | Tier 3 | amount=20000 | `75m` (55 + 10000*0.002) |
| 6 | FeeType is "Tiered" | | `"Tiered"` |

**Polymorphism Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | All calculators through interface | `List<IFeeCalculator>` with all 3 types | each calculates correctly |
| 2 | Same amount, different strategies | Calculate(1000) with each | Fixed=5, Percent=20, Tiered=10 |

---

## Exercise 5: Transaction Processor — Composing Everything

**Concepts:** Combining inheritance and interfaces, Strategy pattern, polymorphism in action

### Your Task

```csharp
// TransactionProcessor.cs
public class TransactionProcessor
{
    // TODO: Private readonly fields
    // - _validator (ITransactionValidator)
    // - _feeCalculator (IFeeCalculator)

    // TODO: Constructor(ITransactionValidator validator, IFeeCalculator feeCalculator)
    //   - Validates: both must not be null

    // TODO: public (bool Success, decimal Fee) ProcessTransaction(
    //     Transaction transaction, BankAccount account)
    //
    //   Logic:
    //   1. Validate the transaction using _validator.Validate(transaction, account)
    //      - If validation fails: return (false, 0)
    //   2. Calculate fee using _feeCalculator.Calculate(transaction.Amount)
    //   3. Execute the transaction using transaction.Execute()
    //      - If execution fails: return (false, 0)
    //   4. If there's a fee > 0, withdraw the fee from the account
    //      (for deposits, fee is deducted from the deposited account;
    //       for withdrawals/transfers, fee is deducted from the source account)
    //   5. Return (true, fee)

    // TODO: public List<(Transaction Tx, bool Success, decimal Fee)>
    //     ProcessBatch(List<Transaction> transactions, BankAccount account)
    //
    //   Processes each transaction in order, collecting results.
    //   A failed transaction does NOT stop the batch — continue with the next one.
}
```

### Test Cases — Implement in `TransactionProcessorTests.cs`

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Valid deposit processes | BasicValidator, FixedFee(5), deposit 1000 | Success=true, Fee=5, Balance=995 |
| 2 | Valid withdrawal processes | BasicValidator, FixedFee(2), balance=1000, withdraw 400 | Success=true, Fee=2, Balance=598 |
| 3 | Failed validation stops processing | StrictValidator(limit=100), deposit 500 | Success=false, Fee=0 |
| 4 | Failed withdrawal returns false | balance=100, withdraw 500 | Success=false |
| 5 | Batch processes all transactions | 3 deposits of 100 each | all succeed, 3 results returned |
| 6 | Batch continues after failure | deposit 100, withdraw 9999, deposit 100 | first succeeds, second fails, third succeeds |
| 7 | Different fee strategies | same transaction, PercentFee vs FixedFee | different fee amounts |
| 8 | Null validator throws | null passed to constructor | throws ArgumentNullException |

---

## Exercise 6 (Challenge): Fee Charge Transaction — A Fourth Transaction Type

**Concepts:** Adding a new type to an existing hierarchy without modifying existing code

### Your Task

Create a `FeeChargeTransaction` that represents a fee being charged to an account. This demonstrates the power of inheritance + polymorphism: **you add a new type without changing any existing code**.

```csharp
// FeeChargeTransaction.cs
public class FeeChargeTransaction : Transaction
{
    // TODO: Properties
    // - TargetAccount (BankAccount): the account being charged
    // - RelatedTransactionId (long): the transaction this fee is for
    // - FeeType (string): "Fixed", "Percent", etc.

    // TODO: Constructor(long id, decimal amount, string ccy,
    //                   BankAccount account, long relatedTransactionId, string feeType)

    // TODO: Override TransactionType => "FEE"

    // TODO: Override Execute()
    //   - Withdraw fee amount from TargetAccount
    //   - If withdrawal fails (insufficient funds), mark failed
    //   - Otherwise mark completed

    // TODO: Override GetSummary()
    //   - Returns: "[FEE] 5.00 GEL for Transaction #456 (Fixed) - COMPLETED"
}
```

### Test Cases — Implement in `FeeChargeTransactionTests.cs`

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | TransactionType is FEE | new FeeChargeTransaction | `"FEE"` |
| 2 | Execute deducts fee | balance=1000, fee=5 | Balance=995, returns true |
| 3 | Insufficient balance fails | balance=2, fee=5 | returns false, Status=FAILED |
| 4 | GetSummary includes related ID | relatedId=456 | contains `"#456"` |
| 5 | Works in polymorphic list | `List<Transaction>` with deposits + fees | all execute correctly |

---

## Bonus Exercise: Notification System — Interface Polymorphism

**Concepts:** Pure interface polymorphism, multiple implementations, composing notification channels

### Your Task

```csharp
// INotificationSender.cs
public interface INotificationSender
{
    bool Send(string recipient, string message);
    string Channel { get; }
}

// ConsoleNotifier.cs (for testing — prints to console)
public class ConsoleNotifier : INotificationSender
{
    public string Channel => "Console";
    public List<string> SentMessages { get; } = new();  // for testing

    public bool Send(string recipient, string message)
    {
        string log = $"[{Channel}] To: {recipient} - {message}";
        SentMessages.Add(log);
        Console.WriteLine(log);
        return true;
    }
}

// CompositeNotifier.cs — sends through MULTIPLE channels
public class CompositeNotifier : INotificationSender
{
    // TODO: Holds a list of INotificationSender
    // TODO: Constructor takes params INotificationSender[] senders
    // TODO: Channel => "Composite"
    // TODO: Send() sends through ALL inner senders, returns true if ALL succeed
}
```

### Test Cases

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Single notifier works | ConsoleNotifier.Send(...) | returns true, message logged |
| 2 | Composite sends to all | 3 ConsoleNotifiers in composite | all 3 receive the message |
| 3 | Composite returns false if any fails | one failing notifier | returns false |
| 4 | All notifiers work through interface | `INotificationSender n = new ConsoleNotifier()` | compiles and works |

---

## Quick Reference: Inheritance & Interface Patterns

| Pattern | When to Use | Example |
|---|---|---|
| `abstract class` | Shared state + behavior, "is a" relationship | `Transaction` base with shared properties |
| `interface` | Contract for capability, "can do" relationship | `IFeeCalculator`, `ITransactionValidator` |
| `virtual`/`override` | Base provides default, derived can change | `GetSummary()` with default format |
| `abstract` method | No default makes sense, derived must provide | `Execute()` — every transaction type is different |
| `sealed` | Prevent further inheritance | Final, locked-down implementation |
| `: base(...)` | Call parent constructor | `DepositTransaction` calling `Transaction` constructor |
| `base.Method()` | Extend parent behavior | Override `GetSummary()` but include parent's output |
| Strategy pattern | Swap behavior via constructor | `TransactionProcessor` with different validators |

### The "Is A" vs "Can Do" Rule

```
"Is it a kind of...?"           → Inherit from base class
  DepositTransaction IS A Transaction         → : Transaction
  LoanAccount IS A FinancialProduct          → : FinancialProduct

"Can it do...?"                 → Implement an interface
  Transaction CAN BE validated               → ITransactionValidator
  Transaction CAN HAVE fees calculated       → IFeeCalculator
  Account CAN BE notified                   → INotificationSender
```
