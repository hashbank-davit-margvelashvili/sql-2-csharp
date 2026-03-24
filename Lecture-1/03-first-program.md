# Your First C# Program

## The `dotnet` CLI

Just as `sqlcmd` lets you interact with SQL Server from the command line, the `dotnet` CLI is your command-line interface for .NET.

### Essential Commands

| Command | What It Does | SQL Analogy |
|---|---|---|
| `dotnet new console` | Creates a new console project | `CREATE DATABASE MyApp` |
| `dotnet build` | Compiles the code into a `.dll` | Compiling a stored procedure |
| `dotnet run` | Builds and executes the program | `EXEC dbo.MyProcedure` |
| `dotnet add package <name>` | Adds a NuGet package dependency | Adding a linked server / CLR assembly |
| `dotnet test` | Runs automated tests | Running a test script |

## Creating Your First Project

Open a terminal (Command Prompt, PowerShell, or Terminal in Visual Studio) and run:

```bash
# Create a new console application named "HelloBanking"
dotnet new console -n HelloBanking

# Navigate into the project folder
cd HelloBanking
```

This creates the following files:

```
HelloBanking/
  ├── HelloBanking.csproj    ← Project configuration
  ├── Program.cs             ← Your code starts here
  └── obj/                   ← Intermediate build files
```

## The Default Program.cs

Open `Program.cs`. You'll see:

```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

That's the entire program. One line. Let's break it down:

| Part | Meaning |
|---|---|
| `Console` | A built-in class that represents the terminal window |
| `.` | Member access operator (access a method on the class) |
| `WriteLine` | A method that prints text followed by a new line |
| `("Hello, World!")` | The argument -- the text to print |
| `;` | Statement terminator (like the end of a SQL statement) |

### T-SQL equivalent:
```sql
PRINT 'Hello, World!'
```

## Running the Program

```bash
dotnet run
```

Output:
```
Hello, World!
```

What happened behind the scenes:
1. `dotnet run` first called `dotnet build` (compiled your C# into IL)
2. Then it executed the resulting program
3. `Console.WriteLine` printed the text to the terminal

## Building Without Running

```bash
dotnet build
```

This compiles your code and produces output in `bin/Debug/net10.0/`:
```
bin/
  └── Debug/
        └── net10.0/
              ├── HelloBanking.dll    ← Your compiled code (IL)
              ├── HelloBanking.exe    ← The launcher executable
              └── HelloBanking.deps.json  ← Dependency manifest
```

## Top-Level Statements

The single-line `Program.cs` we saw uses **top-level statements** -- a C# 9+ feature that removes boilerplate. Behind the scenes, the compiler generates this:

```csharp
// What the compiler actually creates:
internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
    }
}
```

You don't have to write the class or the `Main` method. But know they exist -- you'll see the explicit form in older code and tutorials.

> **Analogy:** It's like how SQL Server lets you run `SELECT 1` without wrapping it in a stored procedure. The engine handles the wrapper for you.

## Console.WriteLine and String Interpolation

### Basic Output

```csharp
Console.WriteLine("Hello, Banking World!");     // Prints with newline
Console.Write("Enter amount: ");                // Prints without newline
```

### String Interpolation -- Your New Best Friend

In T-SQL, you concatenate strings like this:
```sql
DECLARE @name NVARCHAR(50) = 'Hash Bank';
DECLARE @amount DECIMAL(18,2) = 15000.50;
PRINT 'Bank: ' + @name + ', Amount: ' + CAST(@amount AS NVARCHAR(20));
```

In C#, use **string interpolation** with the `$` prefix:

```csharp
string bankName = "Hash Bank";
decimal amount = 15000.50m;
Console.WriteLine($"Bank: {bankName}, Amount: {amount}");
// Output: Bank: Hash Bank, Amount: 15000.50
```

The `$` before the string enables `{}` placeholders. Anything inside `{}` is evaluated as a C# expression:

```csharp
int a = 10;
int b = 20;
Console.WriteLine($"Sum of {a} and {b} is {a + b}");
// Output: Sum of 10 and 20 is 30
```

### Formatting Inside Interpolation

```csharp
decimal balance = 12345.6789m;

// Currency format
Console.WriteLine($"Balance: {balance:C}");          // Balance: $12,345.68

// Fixed decimal places
Console.WriteLine($"Balance: {balance:F2}");         // Balance: 12345.68

// Number with thousands separator
Console.WriteLine($"Balance: {balance:N2}");         // Balance: 12,345.68

// Date formatting
DateTime now = DateTime.Now;
Console.WriteLine($"Date: {now:yyyy-MM-dd}");        // Date: 2026-03-23
Console.WriteLine($"Time: {now:HH:mm:ss}");          // Time: 14:30:45
```

| Format | C# | T-SQL Equivalent |
|---|---|---|
| Currency | `{amount:C}` | `FORMAT(amount, 'C')` |
| Fixed decimals | `{amount:F2}` | `CAST(amount AS DECIMAL(18,2))` |
| Date | `{date:yyyy-MM-dd}` | `FORMAT(date, 'yyyy-MM-dd')` |

## Reading User Input

```csharp
Console.Write("Enter your name: ");
string? name = Console.ReadLine();     // Reads a line of text from the user

Console.Write("Enter amount: ");
string? input = Console.ReadLine();
decimal amount = decimal.Parse(input!); // Convert string to decimal

Console.WriteLine($"Hello {name}, your amount is {amount:C}");
```

> **Note:** `Console.ReadLine()` returns a `string?` (nullable string) because the user might press Ctrl+C. We'll cover nullable types in depth in Session 6.

### T-SQL equivalent concept:
```sql
-- T-SQL doesn't have interactive input, but the concept of
-- taking a parameter and using it is the same:
CREATE PROCEDURE GreetUser @Name NVARCHAR(50), @Amount DECIMAL(18,2)
AS
BEGIN
    PRINT 'Hello ' + @Name + ', your amount is ' + CAST(@Amount AS NVARCHAR(20))
END
```

## Variables and Basic Types (Preview)

We'll cover types in depth in Session 2, but here's a quick taste to make your first program interesting:

```csharp
// Declaring variables -- like DECLARE in T-SQL
string bankName = "Hash Bank";              // NVARCHAR
int accountNumber = 1001;                  // INT
decimal balance = 25000.75m;               // DECIMAL (note the 'm' suffix)
bool isActive = true;                      // BIT
DateTime openDate = DateTime.Now;          // DATETIME

// Using var -- the compiler figures out the type
var transactionCount = 42;                 // Compiler knows this is int
var greeting = "Hello";                    // Compiler knows this is string
```

| C# | T-SQL | Notes |
|---|---|---|
| `string name = "test";` | `DECLARE @name NVARCHAR(50) = 'test'` | C# strings are always Unicode |
| `int count = 5;` | `DECLARE @count INT = 5` | Same concept |
| `decimal amount = 10.5m;` | `DECLARE @amount DECIMAL(18,2) = 10.5` | Use `m` suffix for decimal literals |
| `bool flag = true;` | `DECLARE @flag BIT = 1` | C# uses `true`/`false`, not `1`/`0` |

## Complete Example: Your First Banking Program

```csharp
// Program.cs -- First Banking Console App

Console.WriteLine("=== Welcome to HSP Account Viewer ===");
Console.WriteLine();

// Simulate account data (like a SELECT result)
string accountHolder = "Davit Margvelashvili";
int accountNumber = 100234;
decimal balance = 45_000.50m;    // Underscores for readability (like 45000.50)
string currency = "GEL";
bool isActive = true;
DateTime lastTransaction = new DateTime(2026, 3, 20);

// Display account information
Console.WriteLine("Account Details:");
Console.WriteLine($"  Holder:          {accountHolder}");
Console.WriteLine($"  Account #:       {accountNumber}");
Console.WriteLine($"  Balance:         {balance:N2} {currency}");
Console.WriteLine($"  Status:          {(isActive ? "Active" : "Inactive")}");
Console.WriteLine($"  Last Transaction: {lastTransaction:yyyy-MM-dd}");
Console.WriteLine();

// Simple calculation
Console.Write("Enter deposit amount: ");
string? input = Console.ReadLine();

if (decimal.TryParse(input, out decimal depositAmount))
{
    decimal newBalance = balance + depositAmount;
    Console.WriteLine($"  New Balance:     {newBalance:N2} {currency}");
}
else
{
    Console.WriteLine("  Invalid amount entered.");
}
```

### Output:
```
=== Welcome to HSP Account Viewer ===

Account Details:
  Holder:          Davit Margvelashvili
  Account #:       100234
  Balance:         45,000.50 GEL
  Status:          Active
  Last Transaction: 2026-03-20

Enter deposit amount: 5000
  New Balance:     50,000.50 GEL
```

## Key Takeaways

1. `dotnet new console` creates a new project
2. `dotnet run` compiles and executes
3. `Program.cs` is where execution starts
4. `Console.WriteLine()` prints to the screen (like `PRINT` in T-SQL)
5. String interpolation `$"text {variable}"` is your go-to for building strings
6. Variables are declared with a type and a name: `decimal amount = 100m;`
7. `Console.ReadLine()` reads user input as a string

---

*Previous: [Project Structure](02-project-structure.md) | Next: [NuGet Packages and Git Basics](04-nuget-and-git.md)*
