# Hands-on Exercises

## Exercise 1: Type Mapping Drill

Create a console app that declares variables for a bank account using the correct C# types. For each variable, add a comment showing the SQL Server equivalent.

```csharp
// TODO: Declare these variables with appropriate C# types

// Account ID (like INT in SQL)
??? accountId = 100234;

// Account holder name (like NVARCHAR(100) in SQL)
??? holderName = "Davit Margvelashvili";

// Current balance (like DECIMAL(18,2) in SQL)
??? balance = 45000.50???;

// Is the account active? (like BIT in SQL)
??? isActive = ???;

// Date the account was opened (like DATE in SQL)
??? openedDate = ???;

// Account identifier (like UNIQUEIDENTIFIER in SQL)
??? accountGuid = ???;

// Print all values using string interpolation
Console.WriteLine($"Account: {accountId}");
Console.WriteLine($"Holder: {holderName}");
Console.WriteLine($"Balance: {balance:N2} GEL");
Console.WriteLine($"Active: {isActive}");
Console.WriteLine($"Opened: {openedDate:yyyy-MM-dd}");
Console.WriteLine($"GUID: {accountGuid}");
```

**Expected output:**
```
Account: 100234
Holder: Davit Margvelashvili
Balance: 45,000.50 GEL
Active: True
Opened: 2020-01-15
GUID: (some guid value)
```

---

## Exercise 2: Value Type vs Reference Type Experiment

Run this code and predict the output before looking at the answers:

```csharp
// Experiment 1: Value types
Console.WriteLine("=== Value Types ===");
int original = 100;
int copy = original;
copy = 999;
Console.WriteLine($"Original: {original}");  // What prints?
Console.WriteLine($"Copy: {copy}");          // What prints?

// Experiment 2: Reference types (arrays)
Console.WriteLine("\n=== Reference Types ===");
int[] arrayA = { 10, 20, 30 };
int[] arrayB = arrayA;
arrayB[0] = 999;
Console.WriteLine($"arrayA[0]: {arrayA[0]}");  // What prints?
Console.WriteLine($"arrayB[0]: {arrayB[0]}");  // What prints?

// Experiment 3: Strings (reference type but immutable)
Console.WriteLine("\n=== Strings (Special Case) ===");
string greeting = "Hello";
string other = greeting;
other = "World";
Console.WriteLine($"greeting: {greeting}");  // What prints?
Console.WriteLine($"other: {other}");        // What prints?
```

**Questions to answer:**
1. Why does changing `copy` not affect `original`?
2. Why does changing `arrayB[0]` affect `arrayA[0]`?
3. Why does changing `other` not affect `greeting`, even though strings are reference types?

---

## Exercise 3: Null Operators in Banking

Complete the following code using null operators (`??`, `?.`, `??=`):

```csharp
// Scenario: Reading transaction data that might have null values

string? customerName = null;
string? middleName = null;
string? lastName = "Margvelashvili";

// TODO 1: Use ?? to provide a default for null customerName
string displayName = ???;  // Should be "Unknown Customer"

// TODO 2: Use ?? chaining (like COALESCE) to find the first non-null name
string firstAvailable = ???;  // Should be "Margvelashvili"

// TODO 3: Use ??= to set customerName only if it's null
???  // Set customerName to "Guest" if null

// TODO 4: Use ?. to safely get the length of a nullable string
string? accountNote = null;
int noteLength = ???;  // Should be 0, not crash

// TODO 5: Use ?. with method calls
string? rawInput = "  HELLO WORLD  ";
string? cleaned = ???;  // Should trim and convert to lower case safely

Console.WriteLine($"Display: {displayName}");
Console.WriteLine($"First available: {firstAvailable}");
Console.WriteLine($"Customer: {customerName}");
Console.WriteLine($"Note length: {noteLength}");
Console.WriteLine($"Cleaned: {cleaned}");
```

**Expected output:**
```
Display: Unknown Customer
First available: Margvelashvili
Customer: Guest
Note length: 0
Cleaned: hello world
```

---

## Exercise 4: Currency Converter

Build a console app that converts between GEL, USD, and EUR. This is the core exercise for Session 2.

### Requirements:
1. Ask the user for an amount
2. Ask for the source currency (GEL, USD, EUR)
3. Ask for the target currency
4. Calculate and display the converted amount
5. Handle invalid input gracefully using `TryParse`

### Starter code:

```csharp
Console.WriteLine("=== Currency Converter ===");
Console.WriteLine();

// Exchange rates (as of today, relative to GEL)
const decimal GEL_TO_USD = 0.37m;
const decimal GEL_TO_EUR = 0.34m;
const decimal USD_TO_GEL = 2.70m;
const decimal EUR_TO_GEL = 2.94m;

// Step 1: Get amount from user
Console.Write("Enter amount: ");
string? amountInput = Console.ReadLine();

// TODO: Parse the amount using TryParse
// If invalid, print an error and exit

// Step 2: Get source currency
Console.Write("Source currency (GEL/USD/EUR): ");
string? sourceCurrency = Console.ReadLine()?.ToUpper()?.Trim();

// Step 3: Get target currency
Console.Write("Target currency (GEL/USD/EUR): ");
string? targetCurrency = Console.ReadLine()?.ToUpper()?.Trim();

// Step 4: Convert
// Strategy: Convert source to GEL first, then GEL to target
// TODO: Implement the conversion logic

// Step 5: Display result
// TODO: Show formatted output like:
// "1,000.00 GEL = 370.00 USD"
// "Exchange rate: 1 GEL = 0.37 USD"
```

### Bonus challenges:
- Add input validation for currency codes (only accept GEL, USD, EUR)
- Display the exchange rate used
- Ask if the user wants to do another conversion (loop)
- Handle the case where source and target are the same currency

---

## Exercise 5: String Operations -- Transaction Formatter

Write a program that takes raw transaction data and formats it for display:

```csharp
// Raw transaction data (imagine this came from an HSP procedure)
string rawData = "TXN-2026-0001|TRANSFER|45000.50|GEL|Davit Margvelashvili|Giorgi Beridze|2026-03-23 14:30:00";

// TODO: Split the raw data by '|' into parts
// TODO: Extract each field:
//   - Transaction ID
//   - Type
//   - Amount (parse to decimal)
//   - Currency
//   - Sender
//   - Receiver
//   - Date (parse to DateTime)

// TODO: Display formatted output:
Console.WriteLine("╔══════════════════════════════════════╗");
Console.WriteLine("║       TRANSACTION RECEIPT            ║");
Console.WriteLine("╠══════════════════════════════════════╣");
// Transaction ID:  TXN-2026-0001
// Type:            TRANSFER
// Amount:          45,000.50 GEL
// From:            Davit Margvelashvili
// To:              Giorgi Beridze
// Date:            23 March 2026, 14:30
Console.WriteLine("╚══════════════════════════════════════╝");

// Bonus: Mask the names for privacy
// "Davit Margvelashvili" → "D***t M*************i"
```

---

## Homework

Before Session 3:

1. **Complete Exercises 1-4** from this document
2. **Experiment:** Try each null operator (`??`, `?.`, `??=`) in your own code
3. **Read:** *C# 12 in a Nutshell* -- Chapter 2: C# Language Basics (focus on types and operators)
4. **Preview for Session 3:** Think about how `IF...ELSE` and `WHILE` in T-SQL would look in C#

---

*Previous: [Operators and Expressions](03-operators-and-expressions.md)*
