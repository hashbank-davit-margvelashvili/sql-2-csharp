# Loops in C# — From WHILE and Cursors to foreach

## T-SQL Has One Loop. C# Has Four.

In T-SQL, `WHILE` is your only loop construct. Iteration over rows typically requires either set-based operations or a cursor. C# has multiple loop types, each designed for a specific scenario.

---

## WHILE — Direct Translation

```sql
-- T-SQL
DECLARE @i INT = 1;
WHILE @i <= 10
BEGIN
    PRINT @i;
    SET @i = @i + 1;
END
```

```csharp
// C#
int i = 1;
while (i <= 10)
{
    Console.WriteLine(i);
    i++;    // ++ increments by 1, same as SET @i = @i + 1
}
```

The `while` loop in C# is the direct equivalent of `WHILE` in T-SQL. Use it when you don't know how many iterations you need in advance.

### Increment shorthand

```csharp
i++;       // i = i + 1
i--;       // i = i - 1
i += 5;    // i = i + 5
i -= 3;    // i = i - 3
i *= 2;    // i = i * 2
```

---

## for — When You Know the Count

The `for` loop is the standard choice when you know exactly how many iterations you need:

```csharp
// for (initializer; condition; increment)
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);  // prints 0 through 9
}
```

Three parts:
1. **Initializer** `int i = 0` — runs once before the loop starts
2. **Condition** `i < 10` — checked before each iteration; loop runs while `true`
3. **Increment** `i++` — runs after each iteration

```csharp
// Practical banking example: generate 12 monthly statements
for (int month = 1; month <= 12; month++)
{
    Console.WriteLine($"Generating statement for month {month}");
}

// Iterate backwards
for (int i = transactions.Count - 1; i >= 0; i--)
{
    Console.WriteLine(transactions[i]);
}
```

> **No direct SQL equivalent.** T-SQL doesn't have a `FOR` loop. When you needed counted iteration in SQL, you used a `WHILE` with a counter variable — which is exactly what `for` compiles down to.

---

## foreach — Your Cursor Replacement

`foreach` is the most important loop in C# for application developers. It iterates over any collection — `List<T>`, arrays, results from a database query. Think of it as a **safe, automatic cursor**.

```sql
-- T-SQL cursor (what you're used to)
DECLARE @AccountId INT;
DECLARE cur CURSOR FOR SELECT AccountId FROM Accounts;
OPEN cur;
FETCH NEXT FROM cur INTO @AccountId;
WHILE @@FETCH_STATUS = 0
BEGIN
    -- process @AccountId
    FETCH NEXT FROM cur INTO @AccountId;
END;
CLOSE cur;
DEALLOCATE cur;
```

```csharp
// C# foreach — same concept, 1 line
foreach (int accountId in accountIds)
{
    // process accountId
}
```

`foreach` automatically:
- Opens the enumerator (like `OPEN cur`)
- Fetches the next item each iteration (like `FETCH NEXT`)
- Checks if there are more items (like `@@FETCH_STATUS = 0`)
- Closes the enumerator when done (like `CLOSE / DEALLOCATE`)

### foreach with complex objects

```csharp
List<Transaction> transactions = GetTransactions();

foreach (Transaction tx in transactions)
{
    Console.WriteLine($"ID: {tx.Id}, Amount: {tx.Amount:C}, Type: {tx.Type}");
}
```

### var in foreach

Using `var` lets the compiler infer the element type:

```csharp
foreach (var tx in transactions)   // var inferred as Transaction
{
    Console.WriteLine(tx.Amount);
}
```

---

## do-while — Execute At Least Once

The `do-while` loop executes the body *first*, then checks the condition. Use it when you need at least one execution regardless of the condition:

```csharp
int attempts = 0;
do
{
    attempts++;
    bool success = TryProcessPayment(payment);
    if (success) break;
} while (attempts < 3);
```

In T-SQL there's no direct equivalent — you'd need to use `WHILE 1=1` with a `BREAK` inside.

---

## break and continue

These keywords control loop execution:

```csharp
// break — exit the loop entirely
foreach (var tx in transactions)
{
    if (tx.Amount > 100000m)
    {
        Console.WriteLine("Large transaction found, stopping review.");
        break;   // exits the foreach loop
    }
}

// continue — skip to the next iteration
foreach (var tx in transactions)
{
    if (tx.Status == "Cancelled")
        continue;    // skip cancelled transactions, go to next

    ProcessTransaction(tx);
}
```

> **SQL analogy:** There's no direct equivalent in T-SQL loops, though you might mimic `continue` with a `GOTO` label at the end of a `WHILE` block — which is why C# code is cleaner.

---

## Choosing the Right Loop

| Scenario | Loop to Use |
|---|---|
| Known number of iterations | `for` |
| Iterating over a collection | `foreach` |
| Unknown number of iterations (condition-based) | `while` |
| Must execute body at least once | `do-while` |
| Processing database results row by row | `foreach` (after loading to `List<T>`) |

### The Golden Rule for Banking Code

**Prefer set-based operations (LINQ) over loops.** Just as you learned to avoid cursors in SQL and use `SELECT/UPDATE/JOIN` instead, in C# you'll often use LINQ (Session 7) instead of explicit loops. But you need to understand loops first — LINQ is built on the same concepts.

---

## Practical Example: Processing a Transaction Batch

```csharp
List<Transaction> transactions = LoadPendingTransactions();

decimal totalFees = 0m;
int processed = 0;
int skipped = 0;

foreach (var tx in transactions)
{
    // Skip if already processed
    if (tx.Status != "Pending")
    {
        skipped++;
        continue;
    }

    // Stop if we've hit the daily limit
    if (processed >= 1000)
    {
        Console.WriteLine("Daily processing limit reached.");
        break;
    }

    decimal fee = CalculateFee(tx.Amount, tx.Type);
    totalFees += fee;
    processed++;
}

Console.WriteLine($"Processed: {processed}, Skipped: {skipped}, Total fees: {totalFees:C}");
```

This pattern — iterate, filter with `continue`, exit with `break`, accumulate results — is extremely common in banking application code.
