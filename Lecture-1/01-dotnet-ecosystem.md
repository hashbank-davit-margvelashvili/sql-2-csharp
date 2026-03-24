# The .NET Ecosystem -- A SQL Developer's Guide

## What is .NET?

You already know SQL Server -- it's a **runtime** that takes your T-SQL code, compiles it into an execution plan, and runs it. .NET works the same way, but for C# code.

| SQL Server | .NET |
|---|---|
| SQL Server Engine | CLR (Common Language Runtime) |
| T-SQL source code | C# source code |
| Execution Plan | IL (Intermediate Language) |
| Query Optimizer | JIT (Just-In-Time) Compiler |
| SSMS | Visual Studio / VS Code |
| SQL Server versions (2019, 2022) | .NET versions (8, 9, 10) |

## How .NET Executes Your Code

When SQL Server runs your stored procedure, this happens:

```
T-SQL Code → Parser → Query Optimizer → Execution Plan → Results
```

When .NET runs your C# program, a similar pipeline executes:

```
C# Code → C# Compiler → IL (Intermediate Language) → JIT Compiler → Machine Code → Results
```

### Step by step:

1. **You write C# code** in a `.cs` file (like writing T-SQL in a `.sql` file)
2. **The C# compiler (`csc`)** converts your code into **IL (Intermediate Language)** -- a platform-neutral bytecode stored in a `.dll` file
3. **The CLR loads the IL** at runtime and the **JIT compiler** converts it to native machine code just before execution
4. **Your code runs** as native machine code on the processor

### Why this two-step compilation?

The same reason SQL Server generates execution plans: **optimization**. The JIT compiler knows exactly what hardware your code is running on and can optimize for it. Your `.dll` file is portable -- it runs on Windows, Linux, or macOS without recompilation.

## .NET 10 -- The Version We Use

.NET releases follow a predictable schedule:

| Version | Release | Support |
|---|---|---|
| .NET 8 | Nov 2023 | **LTS** (Long-Term Support) -- 3 years |
| .NET 9 | Nov 2024 | STS (Standard-Term Support) -- 18 months |
| .NET 10 | Nov 2025 | **LTS** -- 3 years |

We use **.NET 10** because:
- It's the latest **LTS** version -- meaning Microsoft supports it with security patches for 3 years
- In banking, we always choose LTS versions for production stability
- It ships with **C# 14**, the latest version of the language

> **Think of it this way:** .NET 10 is to C# what SQL Server 2022 is to T-SQL -- the runtime determines which language features are available.

## Key .NET Components

### CLR (Common Language Runtime)
The engine that runs your code. It handles:
- **Memory management** -- automatic garbage collection (no manual memory allocation like in C/C++)
- **Type safety** -- catches type errors at compile time
- **Exception handling** -- structured error handling (like TRY...CATCH in T-SQL, but more powerful)
- **Security** -- code access security and verification

### BCL (Base Class Library)
A massive library of pre-built code that ships with .NET. Think of it as built-in system stored procedures and functions -- but for everything:
- File I/O (`System.IO`)
- Networking (`System.Net`)
- Collections (`System.Collections`)
- Text processing (`System.Text`)
- JSON/XML (`System.Text.Json`, `System.Xml`)
- Cryptography (`System.Security.Cryptography`)

### SDK (Software Development Kit)
The tools you install to build .NET applications:
- The C# compiler
- The `dotnet` CLI (command-line interface)
- Project templates
- Package manager (NuGet)

### Runtime
The minimal set of components needed to **run** a .NET application (CLR + BCL). In production, you only need the runtime, not the full SDK.

## .NET vs .NET Framework -- Don't Confuse Them

| | .NET Framework | .NET (modern) |
|---|---|---|
| Versions | 1.0 -- 4.8.x | 5, 6, 7, 8, 9, 10 |
| Platforms | Windows only | Windows, Linux, macOS |
| Status | Maintenance mode | Actively developed |
| Performance | Good | Excellent |
| Open source | Partially | Fully |

> **Important:** When someone says ".NET" without a version, they usually mean modern .NET (5+). ".NET Framework" is the legacy Windows-only version. We use modern .NET 10.

## Managed Code

C# code running on the CLR is called **managed code** because the CLR *manages* it:
- **Memory** is automatically allocated and freed (garbage collection)
- **Array bounds** are checked at runtime (no buffer overflows)
- **Type safety** is enforced

This is different from **unmanaged code** (C, C++) where the programmer must manage memory manually. In SQL Server, you never worry about memory management either -- the engine handles it. .NET works the same way.

## Summary

| Concept | What It Means | SQL Analogy |
|---|---|---|
| .NET 10 | The platform/runtime version | SQL Server 2022 |
| C# 14 | The programming language version | T-SQL dialect for that SQL version |
| CLR | The engine that runs your code | SQL Server Database Engine |
| IL | Compiled intermediate code | Execution Plan |
| JIT | Converts IL to machine code at runtime | Query Optimizer choosing operators |
| BCL | Built-in library of useful classes | System stored procedures and functions |
| SDK | Tools for building .NET apps | SSMS + SQLCMD + development tools |
| NuGet | Package manager for external libraries | Adding a linked server or referenced assembly |

---

*Next: [Project Structure -- Solutions, Projects, and Files](02-project-structure.md)*
