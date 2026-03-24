# Project Structure -- Solutions, Projects, and Files

## The SQL Server Analogy

You already organize code in SQL Server with a clear hierarchy:

```
SQL Server Instance
  └── Database (e.g., HSP)
        ├── Schema: dbo
        │     ├── Tables
        │     ├── Views
        │     └── Stored Procedures
        ├── Schema: payments
        │     ├── Tables
        │     └── Stored Procedures
        └── Schema: accounts
              ├── Tables
              └── Stored Procedures
```

.NET has an equivalent hierarchy:

```
Solution (.sln)
  └── Project (e.g., HSP.Transactions.csproj)
        ├── Namespace: HSP.Transactions.Models
        │     ├── Transaction.cs
        │     └── Account.cs
        ├── Namespace: HSP.Transactions.Services
        │     └── TransactionService.cs
        └── Namespace: HSP.Transactions.Data
              └── TransactionRepository.cs
```

## Mapping the Concepts

| SQL Server | .NET | Purpose |
|---|---|---|
| Server Instance | Machine / Development environment | Where everything runs |
| Database | Solution (`.sln`) | Top-level container grouping related components |
| Schema | Project (`.csproj`) / Namespace | Logical grouping and separation |
| Table | Class (`.cs` file) | Defines the structure and behavior of a concept |
| Column | Property | A named attribute with a type |
| Row | Object (instance of a class) | A concrete piece of data |
| Stored Procedure | Method | A named block of code that does something |

## Solution (`.sln`)

A **solution** is the top-level container. It groups one or more **projects** that work together.

```
CoreBankingApp.sln          <-- Solution file (like a Database)
  ├── CoreBankingApp/        <-- Main project
  ├── CoreBankingApp.Tests/  <-- Test project
  └── CoreBankingApp.Shared/ <-- Shared library project
```

The `.sln` file is a plain-text file that lists which projects belong to the solution. You rarely edit it by hand -- Visual Studio manages it.

> **Analogy:** A solution is like a database that contains multiple schemas. Just as one database groups related schemas, one solution groups related projects.

## Project (`.csproj`)

A **project** is a single deployable unit. It compiles into one `.dll` (library) or `.exe` (executable).

The `.csproj` file defines:
- Target framework (e.g., `net10.0`)
- Dependencies (NuGet packages)
- Build settings

Here is a minimal `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

| Element | Meaning |
|---|---|
| `OutputType` | `Exe` = console app, omit for a library (`.dll`) |
| `TargetFramework` | Which .NET version to target |
| `Nullable` | Enable nullable reference type checking |

> **Analogy:** A project is like a schema. Just as `dbo` and `payments` group related objects, each project groups related C# files with a clear purpose.

## Namespaces

**Namespaces** organize classes logically within a project, preventing name collisions.

```csharp
namespace HSP.Transactions.Models
{
    public class Transaction
    {
        public int Id { get; set; }
        public decimal Amount { get; set; }
    }
}
```

This is exactly like schema-qualifying objects in SQL:
- SQL: `payments.GetTransaction` vs `accounts.GetTransaction`
- C#: `HSP.Payments.Transaction` vs `HSP.Accounts.Transaction`

To use a class from another namespace, you `use` it:

```csharp
using HSP.Transactions.Models;  // Like referencing another schema
```

In modern .NET, namespaces typically match the folder structure:
```
HSP.Transactions/
  ├── Models/
  │     └── Transaction.cs     → namespace HSP.Transactions.Models
  ├── Services/
  │     └── TransactionService.cs → namespace HSP.Transactions.Services
  └── Program.cs                → namespace HSP.Transactions
```

## Key Files in a .NET Project

### `Program.cs` -- The Entry Point

This is where your application starts. In SQL Server, there's no single entry point -- the engine responds to incoming queries. In C#, every application must have a `Program.cs` that defines where execution begins.

```csharp
// Program.cs -- the simplest possible C# program
Console.WriteLine("Hello, Banking World!");
```

This is called a **top-level statement** (C# 9+). Behind the scenes, the compiler wraps this in a class and a `Main` method, but you don't have to write that boilerplate.

The equivalent in T-SQL would be:
```sql
PRINT 'Hello, Banking World!'
```

### `*.cs` -- C# Source Files

All C# code lives in `.cs` files. One file typically contains one class (though this isn't a hard rule).

```
Transaction.cs      → contains the Transaction class
Account.cs          → contains the Account class
TransactionService.cs → contains the TransactionService class
```

### `bin/` and `obj/` -- Build Output

When you build a project, the compiler outputs files to these folders:
- `obj/` -- intermediate build files (like tempdb for compilation)
- `bin/` -- the final output (`.dll` or `.exe`)

These folders are **generated** and should never be committed to Git (more on this later).

## Project Types

| Type | Template Command | Output | Use Case |
|---|---|---|---|
| Console App | `dotnet new console` | `.exe` | CLI tools, background services |
| Web API | `dotnet new webapi` | `.dll` (hosted) | REST APIs, microservices |
| Class Library | `dotnet new classlib` | `.dll` | Shared code between projects |
| xUnit Test | `dotnet new xunit` | `.dll` (test runner) | Automated tests |

For this course, we start with **Console App** (Sessions 1-10) and move to **Web API** (Session 11+).

## Folder Structure Convention

A typical .NET project follows this layout:

```
MyProject/
  ├── MyProject.sln           ← Solution file
  ├── src/
  │     └── MyProject/
  │           ├── MyProject.csproj  ← Project file
  │           ├── Program.cs        ← Entry point
  │           ├── Models/           ← Data classes
  │           ├── Services/         ← Business logic
  │           └── appsettings.json  ← Configuration
  ├── tests/
  │     └── MyProject.Tests/
  │           ├── MyProject.Tests.csproj
  │           └── TransactionTests.cs
  ├── .gitignore                ← Files to exclude from Git
  └── README.md                 ← Project documentation
```

## Summary

- A **Solution** groups related projects (like a Database groups schemas)
- A **Project** is a single compilable unit (like a Schema groups objects)
- **Namespaces** organize classes logically (like schema qualification)
- `Program.cs` is the application entry point
- `.csproj` defines the project's configuration and dependencies
- `bin/` and `obj/` are build artifacts -- never commit them

---

*Previous: [The .NET Ecosystem](01-dotnet-ecosystem.md) | Next: [Your First C# Program](03-first-program.md)*
