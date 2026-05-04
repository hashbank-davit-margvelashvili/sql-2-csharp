# Hands-On Exercises — Session 6: Structs, Enums, Records, Tuples, and Other Type Constructs

## The Goal

Refactor the banking model from Sessions 4–5. Replace magic-number integers with enums, extract value objects into structs, convert DTOs to records, and add proper null safety throughout. You'll end up with a cleaner, more expressive model that's harder to misuse.

---

## Project Setup

Continue in the same solution from Sessions 4–5, or create a new project:

```bash
dotnet new classlib -n HspTypes
```

Organize your files:

```
HspTypes/
  Enums/
    TransactionStatus.cs
    TransactionType.cs
    AccountState.cs
    AccountPermissions.cs
  ValueObjects/
    Money.cs
    DateRange.cs
    AccountNumber.cs
  Records/
    AccountDto.cs
    TransactionDto.cs
    CustomerDto.cs
    TransactionResultDto.cs
  Models/
    BankAccount.cs        (updated from Session 4)
    Transaction.cs        (updated from Session 5)
```

Add a test project:

```bash
dotnet new xunit -n HspTypes.Tests
cd HspTypes.Tests
dotnet add reference ../HspTypes/HspTypes.csproj
```

Enable nullable reference types in both `.csproj` files:

```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

---

## Exercise 1: Enums — Replace Magic Numbers

**Concepts:** `enum`, underlying type, switch expressions, `[Flags]`

### The HSP Problem

```sql
-- HSP stores statuses as integers — prone to magic-number bugs
-- AccountState BIGINT: 0=Active, 1=Frozen, 2=Closed, 3=Blocked
-- The code has literals like: WHERE AccountState = 1
-- Nobody remembers what 1 means without checking the wiki
```

### Your Task

```csharp
// Enums/TransactionStatus.cs
public enum TransactionStatus
{
    // TODO: Define values matching the HSP transaction lifecycle:
    // Pending = 0, Completed = 1, Failed = 2, Cancelled = 3
}

// Enums/TransactionType.cs
public enum TransactionType
{
    // TODO: Define: Deposit = 1, Withdrawal = 2, Transfer = 3, Fee = 4
    // (start from 1, not 0, so default int is not a valid type)
}

// Enums/AccountState.cs
public enum AccountState : long
{
    // TODO: Define to match HSP AccountState column:
    // Active = 0, Frozen = 1, Closed = 2, Blocked = 3
}

// Enums/AccountPermissions.cs
[Flags]
public enum AccountPermissions
{
    // TODO: Define bitmask flags:
    // None = 0
    // CanDeposit  = 1  (bit 0)
    // CanWithdraw = 2  (bit 1)
    // CanTransfer = 4  (bit 2)
    // ReadOnly = CanDeposit  (deposit-only)
    // FullAccess = CanDeposit | CanWithdraw | CanTransfer
}
```

```csharp
// TODO: Add a static class AccountStateExtensions with:
// - string GetDescription(this AccountState state)
//   Returns a human-readable string via switch expression:
//     Active  → "Account is active"
//     Frozen  → "Account is frozen — no withdrawals allowed"
//     Closed  → "Account is closed"
//     Blocked → "Account is blocked by compliance"
//     _       → "Unknown state"
//
// - bool AllowsWithdrawals(this AccountState state)
//   Returns true only for Active
//
// - bool AllowsDeposits(this AccountState state)
//   Returns true for Active and Frozen
```

### Test Cases — Implement in `EnumTests.cs`

**TransactionStatus Tests:**

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Underlying value — Pending is 0 | `(int)TransactionStatus.Pending` | `0` |
| 2 | Underlying value — Completed is 1 | `(int)TransactionStatus.Completed` | `1` |
| 3 | Parse from int | `(TransactionStatus)2` | `TransactionStatus.Failed` |
| 4 | ToString | `TransactionStatus.Cancelled.ToString()` | `"Cancelled"` |
| 5 | TryParse from string | `Enum.TryParse<TransactionStatus>("Failed", out var v)` | `true`, `v == Failed` |

**AccountPermissions (Flags) Tests:**

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | HasFlag — single flag | `FullAccess.HasFlag(CanDeposit)` | `true` |
| 2 | HasFlag — missing flag | `ReadOnly.HasFlag(CanWithdraw)` | `false` |
| 3 | Combine flags | `CanDeposit \| CanWithdraw` | int value = 3 |
| 4 | None has no flags | `None.HasFlag(CanDeposit)` | `false` |

**Extension Method Tests:**

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Active description | `AccountState.Active.GetDescription()` | contains `"active"` |
| 2 | Frozen blocks withdrawals | `AccountState.Frozen.AllowsWithdrawals()` | `false` |
| 3 | Frozen allows deposits | `AccountState.Frozen.AllowsDeposits()` | `true` |
| 4 | Closed blocks both | `AccountState.Closed.AllowsWithdrawals()` | `false` |
| 5 | Closed blocks deposits | `AccountState.Closed.AllowsDeposits()` | `false` |

---

## Exercise 2: Money Struct — Value Object

**Concepts:** `readonly struct`, value semantics, operator overloading, immutability

### The HSP Pattern

```sql
-- Every monetary amount in HSP is always paired with a currency
-- Balance DECIMAL(28,6), CCY VARCHAR(5)
-- Treating them separately causes bugs: comparing amounts in different currencies
```

### Your Task

```csharp
// ValueObjects/Money.cs
public readonly struct Money : IEquatable<Money>
{
    // TODO: Properties
    // - Amount (decimal): get-only
    // - Currency (string): get-only, normalized to uppercase

    // TODO: Constructor(decimal amount, string currency)
    //   - Validates: amount >= 0 (negative money not allowed)
    //   - Validates: currency is not null, empty, and is exactly 3 characters
    //   - Normalizes currency to uppercase

    // TODO: Static factory
    // - public static Money Zero(string currency) => new Money(0m, currency)

    // TODO: Arithmetic — return new Money (immutable)
    // - Money Add(Money other)
    //   Throws InvalidOperationException if currencies differ
    // - Money Subtract(Money other)
    //   Throws InvalidOperationException if currencies differ
    //   Throws if result would be negative

    // TODO: Comparison
    // - bool IsGreaterThan(Money other) — same currency required
    // - bool IsLessThan(Money other) — same currency required

    // TODO: Implement IEquatable<Money>
    // - bool Equals(Money other): Amount == other.Amount && Currency == other.Currency
    // - override bool Equals(object? obj)
    // - override int GetHashCode()
    // - override string ToString() => "1,000.00 GEL" format

    // TODO: Operator overloads
    // - operator == and !=
    // - operator + (calls Add)
    // - operator - (calls Subtract)
}
```

### Test Cases — Implement in `MoneyTests.cs`

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Creation succeeds | `new Money(1000m, "GEL")` | Amount=1000, Currency="GEL" |
| 2 | Currency normalized | `new Money(100m, "gel")` | Currency = "GEL" |
| 3 | Negative amount throws | `new Money(-1m, "GEL")` | throws ArgumentException |
| 4 | Invalid currency throws (2 chars) | `new Money(100m, "GE")` | throws ArgumentException |
| 5 | Null currency throws | `new Money(100m, null!)` | throws ArgumentException |
| 6 | Add same currency | `1000 GEL + 500 GEL` | `1500 GEL` |
| 7 | Add different currency throws | `1000 GEL + 500 USD` | throws InvalidOperationException |
| 8 | Subtract succeeds | `1000 GEL - 300 GEL` | `700 GEL` |
| 9 | Subtract to negative throws | `100 GEL - 500 GEL` | throws InvalidOperationException |
| 10 | Equality — same values | `new Money(100m, "GEL") == new Money(100m, "GEL")` | `true` |
| 11 | Equality — different amount | `new Money(100m, "GEL") == new Money(200m, "GEL")` | `false` |
| 12 | Value copy semantics | assign `a` to `b`, change `b` | `a` unchanged |
| 13 | Zero factory | `Money.Zero("GEL")` | Amount=0, Currency="GEL" |
| 14 | ToString format | `new Money(1500m, "GEL").ToString()` | `"1,500.00 GEL"` |
| 15 | IsGreaterThan | `500 GEL.IsGreaterThan(300 GEL)` | `true` |

---

## Exercise 3: DateRange Struct — Another Value Object

**Concepts:** `readonly struct`, validation, computed properties

### The HSP Context

```sql
-- Deposits have validity ranges: ValidFrom DATETIME2, ValidTo DATETIME2
-- These dates always travel together and ValidFrom must be before ValidTo
```

### Your Task

```csharp
// ValueObjects/DateRange.cs
public readonly struct DateRange
{
    // TODO: Properties
    // - From (DateTime): get-only
    // - To (DateTime): get-only

    // TODO: Constructor(DateTime from, DateTime to)
    //   - Validates: from must be before to (throws ArgumentException if not)

    // TODO: Computed properties
    // - DurationDays (int): (To - From).Days
    // - IsActive (bool): DateTime.Now >= From && DateTime.Now <= To
    // - IsExpired (bool): DateTime.Now > To
    // - IsFuture (bool): DateTime.Now < From

    // TODO: bool Contains(DateTime date) — true if date is within [From, To]
    // TODO: bool Overlaps(DateRange other) — true if ranges overlap
    // TODO: override ToString() => "2025-01-01 — 2026-01-01 (365 days)"
}
```

### Test Cases — Implement in `DateRangeTests.cs`

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Valid range created | From=yesterday, To=tomorrow | succeeds |
| 2 | From > To throws | From=tomorrow, To=yesterday | throws ArgumentException |
| 3 | From == To throws | same date | throws ArgumentException |
| 4 | DurationDays | 1-Jan to 1-Feb | 31 |
| 5 | IsActive — current range | From=yesterday, To=tomorrow | `true` |
| 6 | IsExpired — past range | From=2020, To=2021 | `true` |
| 7 | IsFuture — future range | From=2030, To=2031 | `true` |
| 8 | Contains — date inside | range 2025–2026, date=2025-06-01 | `true` |
| 9 | Contains — date outside | range 2025–2026, date=2027-01-01 | `false` |
| 10 | Overlaps — overlapping ranges | 2025–2026 and 2025-06–2027 | `true` |
| 11 | Overlaps — non-overlapping | 2025–2026 and 2027–2028 | `false` |

---

## Exercise 4: Records — Immutable DTOs

**Concepts:** `record`, positional syntax, `with` expression, value equality, inheritance

### The HSP Pattern

```sql
-- When data is read from the database and passed to the API layer,
-- it should be a snapshot — immutable, equal by content, not by reference
-- SELECT AccountID, AccountNumber, Balance, CCY, AccountState FROM hw.Accounts
```

### Your Task

```csharp
// Records/AccountDto.cs
public record AccountDto(
    long AccountId,
    string AccountNumber,
    Money Balance,
    AccountState State,
    int CustomerId,
    DateTime CreateTime)
{
    // TODO: Computed property
    // - bool IsActive => State == AccountState.Active

    // TODO: Static factory
    // - public static AccountDto FromBankAccount(BankAccount account)
    //   Maps a BankAccount object to this DTO
    //   Use: new Money(account.Balance, account.CCY) for the Balance field
}
```

```csharp
// Records/TransactionDto.cs
public record TransactionDto(
    long TransactionId,
    TransactionType Type,
    Money Amount,
    TransactionStatus Status,
    DateTime Timestamp,
    string? Description)
{
    // TODO: Computed properties
    // - bool IsCompleted => Status == TransactionStatus.Completed
    // - bool IsPending   => Status == TransactionStatus.Pending
}
```

```csharp
// Records/TransactionResultDto.cs
// A result record — used to return processing outcome
public record TransactionResultDto(
    bool Success,
    TransactionDto Transaction,
    Money Fee,
    string? ErrorMessage)
{
    // TODO: Static factories
    // - public static TransactionResultDto Ok(TransactionDto tx, Money fee)
    //   Returns new result with Success=true, ErrorMessage=null
    //
    // - public static TransactionResultDto Fail(TransactionDto tx, string reason)
    //   Returns new result with Success=false, Fee=Money.Zero(tx.Amount.Currency)
}
```

### Test Cases — Implement in `RecordTests.cs`

**AccountDto Tests:**

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Value equality | two AccountDtos with same data | `dto1 == dto2` is `true` |
| 2 | Reference inequality | same data, different objects | not `ReferenceEquals(dto1, dto2)` |
| 3 | `with` — creates new record | `dto with { State = AccountState.Frozen }` | new record; original unchanged |
| 4 | IsActive — Active state | State=Active | `true` |
| 5 | IsActive — Frozen state | State=Frozen | `false` |
| 6 | Immutability | `dto.AccountId = 999` | does NOT compile |

**TransactionResultDto Tests:**

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Ok factory | `TransactionResultDto.Ok(tx, fee)` | Success=true, ErrorMessage=null |
| 2 | Fail factory | `TransactionResultDto.Fail(tx, "reason")` | Success=false, Fee=Zero |
| 3 | with on result | `result with { Success = false }` | new record with Success changed |

---

## Exercise 5: Tuples — Multi-Value Returns

**Concepts:** Named tuples, destructuring, `_` discard, local use

### Your Task

```csharp
// Add these methods to a static class AccountAnalyzer:

// TODO: (decimal Min, decimal Max, decimal Average, int Count)
//       GetBalanceStats(IEnumerable<AccountDto> accounts)
//   Returns min, max, average balance and count across all accounts.
//   Use a single pass through the list.

// TODO: (AccountDto? Richest, AccountDto? Poorest)
//       FindExtremes(IEnumerable<AccountDto> accounts)
//   Returns the account with the highest and lowest balance.
//   Returns (null, null) if the collection is empty.

// TODO: (List<AccountDto> Active, List<AccountDto> Inactive)
//       Partition(IEnumerable<AccountDto> accounts)
//   Splits accounts into active and inactive lists.
//   Active = AccountState.Active; Inactive = everything else.

// TODO: (bool IsValid, string? ErrorMessage)
//       ValidateTransfer(Money amount, AccountDto source, AccountDto destination)
//   Returns (false, reason) if:
//   - amount.Amount <= 0
//   - source.State != Active
//   - destination.State != Active
//   - source.Balance.Amount < amount.Amount
//   - source.Balance.Currency != amount.Currency
//   Returns (true, null) if all checks pass.
```

### Test Cases — Implement in `AccountAnalyzerTests.cs`

**GetBalanceStats Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Single account | balance=1000 GEL | Min=Max=Avg=1000, Count=1 |
| 2 | Three accounts | 500, 1000, 1500 GEL | Min=500, Max=1500, Avg=1000, Count=3 |
| 3 | Empty collection | empty list | Count=0, Min=Max=Avg=0 |

**FindExtremes Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | Find richest | balances: 500, 1000, 250 | Richest.Balance.Amount = 1000 |
| 2 | Find poorest | balances: 500, 1000, 250 | Poorest.Balance.Amount = 250 |
| 3 | Empty collection | empty list | (null, null) |

**Partition Tests:**

| # | Test case | Input | Expected |
|---|---|---|---|
| 1 | All active | 3 active accounts | Active.Count=3, Inactive.Count=0 |
| 2 | Mixed | 2 active, 1 frozen, 1 closed | Active.Count=2, Inactive.Count=2 |
| 3 | Destructure result | `var (active, inactive) = Partition(...)` | compiles and works |

**ValidateTransfer Tests:**

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Valid transfer | 500 GEL, active source (1000 GEL), active dest | `(true, null)` |
| 2 | Zero amount | amount=0 | `(false, non-null reason)` |
| 3 | Source frozen | source state=Frozen | `(false, non-null reason)` |
| 4 | Insufficient funds | source balance=100, amount=500 | `(false, non-null reason)` |
| 5 | Currency mismatch | amount=GEL, source=USD | `(false, non-null reason)` |

---

## Exercise 6: Nullable Types — Null Safety in the Model

**Concepts:** `T?`, `??`, `?.`, `#nullable enable`, `ArgumentNullException.ThrowIfNull`

### The HSP Context

```sql
-- Some columns are NULLable — CustomerID, Description, MaturityDate
-- The C# model should accurately represent this
```

### Your Task

Refactor `BankAccount` from Session 4 with proper null safety:

```csharp
// Models/BankAccount.cs
#nullable enable

public class BankAccount
{
    public long AccountId { get; }
    public string AccountNumber { get; }      // NOT nullable — always required

    // TODO: Make these properly nullable where they could be absent
    // - CustomerName (string?): might not be loaded
    // - Description (string?): optional
    // - ExternalReference (string?): might be absent

    public Money Balance { get; private set; }
    public AccountState State { get; private set; }
    public AccountPermissions Permissions { get; private set; }
    public DateTime CreateTime { get; }
    public DateTime? LastModified { get; private set; }   // null until first update

    // TODO: Constructor(long accountId, string accountNumber, string ccy)
    //   - Use ArgumentNullException.ThrowIfNull for accountNumber
    //   - Use ArgumentException for empty accountNumber
    //   - Initialize Balance to Money.Zero(ccy)
    //   - Initialize State to AccountState.Active
    //   - Initialize Permissions to AccountPermissions.FullAccess

    // TODO: bool Deposit(Money amount)
    //   - Validates: State.AllowsDeposits() — return false if not
    //   - Validates: amount.Currency == Balance.Currency — throws if mismatch
    //   - Adds amount to Balance
    //   - Updates LastModified = DateTime.Now
    //   - Returns true

    // TODO: bool Withdraw(Money amount)
    //   - Validates: Permissions.HasFlag(AccountPermissions.CanWithdraw)
    //   - Validates: State.AllowsWithdrawals()
    //   - Validates: Balance.Amount >= amount.Amount
    //   - All failed checks return false
    //   - Subtracts amount from Balance
    //   - Updates LastModified = DateTime.Now
    //   - Returns true

    // TODO: void Freeze()   → State = AccountState.Frozen; LastModified = now
    // TODO: void Unfreeze() → State = AccountState.Active;  LastModified = now
    // TODO: void Close()    → State = AccountState.Closed;  LastModified = now

    // TODO: AccountDto ToDto()
    //   Returns an AccountDto snapshot of this account's current state
}
```

### Test Cases — Implement in `BankAccountNullTests.cs`

| # | Test case | How to test | Expected |
|---|---|---|---|
| 1 | Null account number throws | `new BankAccount(1, null!, "GEL")` | throws ArgumentNullException |
| 2 | Empty account number throws | `new BankAccount(1, "", "GEL")` | throws ArgumentException |
| 3 | CustomerName is null by default | new BankAccount | CustomerName == null |
| 4 | LastModified is null initially | new BankAccount | LastModified == null |
| 5 | Deposit updates LastModified | `account.Deposit(new Money(100m, "GEL"))` | LastModified != null |
| 6 | Currency mismatch throws | GEL account, USD deposit | throws InvalidOperationException |
| 7 | Frozen account blocks withdrawals | State=Frozen, Withdraw | returns false |
| 8 | Frozen account allows deposits | State=Frozen, Deposit | returns true |
| 9 | Closed account blocks all | State=Closed, Deposit/Withdraw | both return false |
| 10 | ToDto returns AccountDto | after balance change | DTO reflects current state |
| 11 | Null-conditional on optional | `account.CustomerName?.ToUpper()` | null (no NRE) |

---

## Exercise 7 (Challenge): Anonymous Types and Projections

**Concepts:** Anonymous types, LINQ `Select`, `var`, local projections

### Your Task

Write a static class `AccountReporter` with these methods. Each method uses a collection of `AccountDto` and produces a report. Use anonymous types for the intermediate projections — **do not** create named classes for these.

```csharp
// AccountReporter.cs
public static class AccountReporter
{
    // TODO: void PrintBalanceSummary(IEnumerable<AccountDto> accounts)
    //   For each account, print: "ACC001: 5,000.00 GEL [Active]"
    //   Project into anonymous type { Number, Balance, State } first,
    //   then iterate and print.

    // TODO: void PrintTopN(IEnumerable<AccountDto> accounts, int n)
    //   Print the top N accounts by balance, descending.
    //   Project into anonymous type { Rank, Number, Balance }
    //   Print: "1. ACC001: 5,000.00 GEL"

    // TODO: void PrintStateBreakdown(IEnumerable<AccountDto> accounts)
    //   Print a count per AccountState:
    //   "Active: 5 accounts, Total: 25,000.00 GEL"
    //   "Frozen: 2 accounts, Total: 3,000.00 GEL"
    //   Use GroupBy, project into anonymous type { State, Count, Total }.

    // TODO: IEnumerable<string> GetSummaryLines(IEnumerable<AccountDto> accounts)
    //   Return each account as a string line.
    //   Format: "ACC001 | 5,000.00 GEL | Active | 2025-01-15"
    //   Use Select to project, then return the strings.
    //   (Here you cross the method boundary with strings, not the anonymous type)
}
```

### Test Cases — Implement in `AccountReporterTests.cs`

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | GetSummaryLines count | 3 accounts | Returns 3 strings |
| 2 | GetSummaryLines format | account: ACC001, 5000 GEL, Active | Line contains "ACC001", "5,000.00 GEL", "Active" |
| 3 | GetSummaryLines empty | empty list | Returns empty enumerable |

(PrintX methods print to console — test them manually by running a small program.)

---

## Exercise 8 (Challenge): Bringing It All Together

**Concepts:** Combining all Session 6 types in a realistic banking scenario

### Your Task

Create a `TransactionProcessor` that uses all the new types:

```csharp
// Models/TransactionProcessor.cs
public class TransactionProcessor
{
    // TODO: TransactionResultDto Process(
    //     BankAccount source,
    //     BankAccount? destination,    // null for deposits/withdrawals
    //     Money amount,
    //     TransactionType type)
    //
    //   Logic by type:
    //   - Deposit:    source.Deposit(amount)  → success or failure
    //   - Withdrawal: source.Withdraw(amount) → success or failure
    //   - Transfer:   source.Withdraw(amount) then destination!.Deposit(amount)
    //                 destination must not be null for Transfer
    //   - Fee:        source.Withdraw(amount) → fee charge
    //
    //   Return TransactionResultDto.Ok(...) or TransactionResultDto.Fail(...)
    //   Build a TransactionDto (with a fake auto-incremented ID) as part of the result
    //
    //   Apply a 0.5% fee (min 0.50 GEL, max 25 GEL) on Transfer transactions
    //   using a PercentFeeCalculator from Session 5 (if available), or inline logic.

    // TODO: IReadOnlyList<TransactionResultDto> ProcessBatch(
    //     IEnumerable<(BankAccount Source, BankAccount? Destination,
    //                  Money Amount, TransactionType Type)> transactions)
    //
    //   Process each transaction in order.
    //   A failure does NOT stop the batch.
    //   Return all results.
}
```

### Test Cases — Implement in `TransactionProcessorTests.cs`

| # | Test case | Setup | Expected |
|---|---|---|---|
| 1 | Deposit succeeds | source balance=0, deposit 1000 GEL | Success=true, source.Balance.Amount=1000 |
| 2 | Withdraw succeeds | source balance=1000, withdraw 400 GEL | Success=true, Balance=600 |
| 3 | Withdraw fails — NSF | source balance=100, withdraw 500 GEL | Success=false, Balance=100 |
| 4 | Transfer succeeds | from=1000, to=500, transfer 300 GEL | from=700, to=800 |
| 5 | Transfer applies fee | transfer 1000 GEL | Fee.Amount = 5.00 GEL (0.5%) |
| 6 | Transfer fails — NSF | from=100, transfer 500 | Success=false, both balances unchanged |
| 7 | Transfer null dest throws | type=Transfer, destination=null | throws ArgumentNullException |
| 8 | Batch — all succeed | 3 deposits | 3 results, all Success=true |
| 9 | Batch — continues after failure | deposit, over-withdraw, deposit | 3 results: true, false, true |
| 10 | Frozen source — deposit | State=Frozen, deposit | Success=true (deposits allowed while frozen) |
| 11 | Frozen source — withdraw | State=Frozen, withdraw | Success=false |
| 12 | Result DTO immutability | check Success field | cannot assign to Success — init-only |

---

## Quick Reference: Session 6 Type Decision Tree

```
What are you modeling?
│
├─ A named constant set (status, type, state)?
│   └─ enum  (or [Flags] enum for bitmask)
│
├─ A small value with identity-less semantics (Money, DateRange)?
│   ├─ Need auto-equality and ToString?  →  readonly record struct
│   └─ Need custom logic / IEquatable?   →  readonly struct
│
├─ An immutable data snapshot / DTO?
│   └─ record  (reference type, value equality, with expression)
│
├─ Multiple return values from a method?
│   ├─ Stays inside the method / file?  →  named tuple
│   └─ Crosses a layer boundary?        →  record
│
├─ Quick local projection (LINQ, throw-away shape)?
│   └─ anonymous type  new { Prop1, Prop2 }
│
└─ Mutable entity with identity and behavior?
    └─ class
```

### Null Safety Cheat Sheet

| SQL | C# |
|---|---|
| `INT NOT NULL` | `int` (non-nullable value type) |
| `INT NULL` | `int?` |
| `NVARCHAR NOT NULL` | `string` with `#nullable enable` |
| `NVARCHAR NULL` | `string?` |
| `ISNULL(x, 0)` | `x ?? 0` |
| `COALESCE(a, b, c)` | `a ?? b ?? c` |
| Guard: `IF @x IS NULL RAISERROR(...)` | `ArgumentNullException.ThrowIfNull(x)` |
| Conditional access: `CASE WHEN x IS NULL THEN NULL ELSE x.col END` | `x?.Col` |
