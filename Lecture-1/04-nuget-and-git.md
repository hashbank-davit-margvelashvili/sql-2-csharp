# NuGet Packages and Git Basics

## NuGet -- The .NET Package Manager

### What is NuGet?

In SQL Server, when you need functionality that doesn't exist built-in, you have limited options:
- Add a linked server
- Register a CLR assembly
- Write it yourself

In .NET, you have **NuGet** -- a package manager with over 400,000 libraries available for free. Need JSON handling? There's a package. Need a logging framework? There's a package. Need to send emails? There's a package.

> **Analogy:** NuGet is like an app store for code libraries. You search, install, and the package becomes part of your project.

### How NuGet Works

```
You install a package → NuGet downloads it → Your .csproj references it → You use it in code
```

### Installing a Package

```bash
# Install a package from the command line
dotnet add package Newtonsoft.Json

# Install a specific version
dotnet add package Serilog --version 4.0.0
```

After installation, your `.csproj` file gets updated:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>

  <!-- NuGet packages are listed here -->
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageReference Include="Serilog" Version="4.0.0" />
  </ItemGroup>
</Project>
```

### Using a Package in Code

```csharp
using Newtonsoft.Json;  // Import the namespace from the package

var account = new { Id = 1, Name = "Test Account", Balance = 1000m };
string json = JsonConvert.SerializeObject(account, Formatting.Indented);
Console.WriteLine(json);
```

Output:
```json
{
  "Id": 1,
  "Name": "Test Account",
  "Balance": 1000.0
}
```

### Packages We'll Use in This Course

| Package | Purpose | Session |
|---|---|---|
| `Microsoft.EntityFrameworkCore.SqlServer` | Database access (ORM) | 13 |
| `FluentValidation` | Input validation | 16 |
| `Serilog.AspNetCore` | Structured logging | 18 |
| `Swashbuckle.AspNetCore` | Swagger/API documentation | 19 |
| `Riok.Mapperly` | Object mapping (compile-time) | 19 |
| `xunit` | Unit testing | 25 |
| `Moq` | Mocking for tests | 27 |
| `FluentAssertions` | Readable test assertions | 26 |
| `RabbitMQ.Client` | Message broker communication | 41 |
| `Polly` | Resilience (retry, circuit breaker) | 43 |

### Restoring Packages

When you clone a project from Git, the packages aren't included (they're in `.gitignore`). Run:

```bash
dotnet restore
```

This downloads all packages listed in the `.csproj`. It's automatic when you run `dotnet build` or `dotnet run`.

### NuGet.org

Browse packages at [nuget.org](https://www.nuget.org/). Before installing a package, check:
- **Downloads** -- popular packages are usually safer
- **Last updated** -- avoid abandoned packages
- **License** -- ensure it's compatible with your project
- **Dependencies** -- fewer dependencies = simpler

---

## Git -- Version Control for Your Code

### Why Git?

In SQL Server development, you might version your stored procedures by:
- Saving copies with dates: `sp_GetAccount_2026_03_20.sql`
- Using comments: `-- Changed by Giorgi on 2026-03-20`
- Keeping "backup" copies of scripts

This is fragile and doesn't scale. **Git** solves this properly:
- Every change is tracked with who, when, and why
- You can go back to any previous version
- Multiple people can work on the same code simultaneously
- Changes are reviewed before being merged

### Core Concepts

| Git Concept | What It Means | Analogy |
|---|---|---|
| **Repository** | A folder tracked by Git | A database with full change history |
| **Commit** | A snapshot of all files at a point in time | A database backup with a label |
| **Branch** | A parallel line of development | A copy of the database for testing changes |
| **Merge** | Combining changes from two branches | Deploying test changes to production |
| **Remote** | A copy of the repository on a server (GitHub, Azure DevOps) | A replicated database on another server |

### Setting Up Git

```bash
# Configure your identity (one-time setup)
git config --global user.name "Your Name"
git config --global user.email "your.email@bank.ge"
```

### Initializing a Repository

```bash
# Inside your project folder
cd HelloBanking
git init
```

This creates a hidden `.git` folder that tracks all changes.

### The Git Workflow

```
1. Make changes to files
2. Stage changes (select what to commit)
3. Commit (save a snapshot with a message)
```

```bash
# 1. Check what changed
git status

# 2. Stage specific files
git add Program.cs
git add HelloBanking.csproj

# Or stage everything
git add .

# 3. Commit with a descriptive message
git commit -m "Create initial console application with account viewer"
```

### Essential Git Commands

```bash
# See current status (what's changed, what's staged)
git status

# See the history of commits
git log --oneline

# See what changed in detail
git diff

# Create a new branch
git checkout -b feature/add-deposit

# Switch between branches
git checkout main

# Merge a branch into the current branch
git merge feature/add-deposit
```

### Writing Good Commit Messages

Bad:
```
git commit -m "changes"
git commit -m "fix"
git commit -m "update"
```

Good:
```
git commit -m "Add deposit functionality to account viewer"
git commit -m "Fix balance calculation when currency conversion fails"
git commit -m "Add FluentValidation for transaction input"
```

A good commit message:
- Starts with a **verb** (Add, Fix, Update, Remove, Refactor)
- Describes **what** was changed and **why**
- Is specific enough to understand without reading the code

## .gitignore -- What NOT to Track

Some files should never be committed to Git:
- Build output (`bin/`, `obj/`)
- User-specific settings (`.vs/`, `*.user`)
- Secrets (`appsettings.Development.json` with passwords)
- OS files (`.DS_Store`, `Thumbs.db`)

Create a `.gitignore` file in your project root:

```bash
# .gitignore for .NET projects

## Build output
bin/
obj/

## Visual Studio
.vs/
*.user
*.suo

## User-specific settings
*.DotSettings.user

## NuGet packages (restored from .csproj)
packages/

## OS files
Thumbs.db
.DS_Store
```

> **Tip:** When you create a project with `dotnet new`, it includes a basic `.gitignore`. For a comprehensive one, use `dotnet new gitignore`.

### Creating a .gitignore from a Template

```bash
# Generate a comprehensive .NET gitignore
dotnet new gitignore
```

## Your First Git Workflow

Putting it all together:

```bash
# 1. Create the project
dotnet new console -n HelloBanking
cd HelloBanking

# 2. Create .gitignore
dotnet new gitignore

# 3. Initialize Git
git init

# 4. Stage all files
git add .

# 5. Make the first commit
git commit -m "Initialize HelloBanking console project with .NET 10"

# 6. Write some code...
# (edit Program.cs)

# 7. See what changed
git status
git diff

# 8. Commit the changes
git add Program.cs
git commit -m "Add account viewer with balance display"
```

## Visual Studio Git Integration

Visual Studio has built-in Git support. You can do everything from the UI:
- **Git Changes** window: see modified files, stage, commit
- **Git Repository** window: view branches, history
- **Right-click** any file: view history, compare, blame

For this course, we'll use both the CLI and Visual Studio's Git integration. Understanding the CLI helps you understand what the UI is doing.

## Summary

### NuGet
- NuGet is the package manager for .NET (like an app store for code libraries)
- `dotnet add package <name>` installs a package
- Packages are listed in `.csproj` under `<PackageReference>`
- `dotnet restore` downloads all required packages

### Git
- Git tracks every change to your code with full history
- `git init` starts tracking a folder
- `git add` stages changes, `git commit` saves a snapshot
- `.gitignore` prevents build artifacts and secrets from being tracked
- Write descriptive commit messages starting with a verb

---

*Previous: [Your First C# Program](03-first-program.md) | Next: [HSP Project Kickoff](05-hsp-project-kickoff.md)*
