# HSP Project Kickoff -- From Stored Procedures to Microservices

## The Journey Ahead

Throughout this course, you won't just learn C# in a vacuum. You will **extract a real stored procedure from the HSP database** and rewrite it as a production .NET microservice. By Session 20, you'll have a working, tested, documented API that replaces that procedure.

## Why We're Doing This

The HSP database is the bank's core system. It works, but it has limitations:

| HSP (Current) | Microservice (Target) |
|---|---|
| All logic in stored procedures | Logic in C# services with clear separation |
| Tightly coupled -- changing one procedure can break others | Loosely coupled -- each service is independent |
| Difficult to unit test | Comprehensive automated testing |
| Hard to scale (vertical only) | Easy to scale (horizontal, containerized) |
| Single point of failure | Distributed, resilient |
| Limited monitoring | Structured logging, metrics, health checks |
| Manual deployments | Automated CI/CD pipelines |

This course gives you **dual value**: you learn .NET development AND you produce a real microservice that moves the bank toward its target architecture.

## The Extraction Process

Here's how we'll transform a stored procedure into a microservice over the next 20 sessions:

```
Session 1-8:    Learn C# fundamentals, model the domain
Session 9-10:   Handle errors, serialization, HTTP
Session 11-12:  Build the REST API with ASP.NET Core
Session 13-15:  Connect to SQL Server with EF Core
Session 16:     Add validation and error handling
Session 17-19:  Add async, logging, mapping, documentation
Session 20:     Complete, review, and demo
```

## What Makes a Good Candidate Procedure?

Not every stored procedure is suitable for extraction. Look for procedures that:

### Good candidates:
- Have a **clear, bounded responsibility** (e.g., "process a wire transfer", "get account balance")
- Are **self-contained** -- don't call 50 other procedures
- Have **well-defined inputs and outputs**
- Represent a **business capability** that could stand on its own
- Are **frequently called** -- high value for the effort

### Poor candidates (for a first project):
- Massive procedures (1000+ lines) that do everything
- Procedures deeply entangled with others in a chain
- Batch/ETL jobs that process millions of rows
- Procedures that depend on undocumented side effects

## Activity: Identify Your Candidate Procedure

### Step 1: List 3-5 Candidate Procedures

For each candidate, fill in this template:

```
Procedure Name: ___________________________________
Schema:          ___________________________________
Lines of Code:   ___________________________________
What it does (1-2 sentences):
_____________________________________________________
_____________________________________________________
```

### Step 2: Document Your Top Choice

Choose the procedure you understand best and fill in this detailed specification:

```
=== HSP Procedure Specification ===

Procedure Name:    ___________________________________
Schema:            ___________________________________

Purpose:
What business function does this procedure perform?
_____________________________________________________
_____________________________________________________

Input Parameters:
| Parameter    | Type           | Required | Description          |
|-------------|----------------|----------|----------------------|
| @Param1     | INT            | Yes      |                      |
| @Param2     | NVARCHAR(50)   | No       |                      |
| @Param3     | DECIMAL(18,2)  | Yes      |                      |

Output:
What does the procedure return?
[ ] Result set (describe columns below)
[ ] Output parameters (list them)
[ ] Return value (success/error code)

Result Set Columns (if applicable):
| Column       | Type           | Description              |
|-------------|----------------|--------------------------|
|             |                |                          |
|             |                |                          |

Business Rules:
List the validation and business logic in the procedure:
1. ________________________________________________
2. ________________________________________________
3. ________________________________________________

Error Handling:
How does the procedure handle errors?
_____________________________________________________

Dependencies:
| Table/View  | How Used (Read/Write/Both) |
|-------------|---------------------------|
|             |                           |
|             |                           |

Called By:
What applications or procedures call this?
_____________________________________________________

Frequency:
How often is this procedure called?
[ ] Thousands/day  [ ] Hundreds/day  [ ] Tens/day  [ ] Rarely
```

### Step 3: Envision the Microservice

Start thinking about how this procedure maps to a REST API:

```
Stored Procedure Call:
  EXEC payments.ProcessTransfer @FromAccount, @ToAccount, @Amount, @Currency

Becomes REST API:
  POST /api/v1/transfers
  {
    "fromAccount": 10023,
    "toAccount": 20045,
    "amount": 500.00,
    "currency": "GEL"
  }

  Response (201 Created):
  {
    "transferId": 78901,
    "status": "completed",
    "fromAccount": 10023,
    "toAccount": 20045,
    "amount": 500.00,
    "currency": "GEL",
    "timestamp": "2026-03-23T14:30:00Z"
  }
```

Map your procedure:

```
My Procedure Call:
  EXEC _________._________ @_________, @_________, @_________

REST API Endpoint:
  [METHOD] /api/v1/___________

Request Body (for POST/PUT):
  {
    "___________": ___,
    "___________": ___
  }

Response:
  {
    "___________": ___,
    "___________": ___
  }
```

## Homework

Before Session 2:

1. **Install the tools** (if not done during the session):
   - Visual Studio 2026 (Community edition is free)
   - .NET 10 SDK
   - Git for Windows
   - SQL Server Management Studio (you already have this)

2. **Create your first project and commit it to Git:**
   ```bash
   dotnet new console -n HSP.MyServiceName
   cd HSP.MyServiceName
   dotnet new gitignore
   git init
   git add .
   git commit -m "Initialize HSP microservice project"
   ```

3. **Complete the procedure specification** (Step 2 above) for your chosen HSP procedure

4. **Read:** *C# 12 in a Nutshell* -- Chapter 1 (15-20 pages)

## What's Next

In **Session 2**, we'll dive into C# types and variables. You'll learn how every SQL Server data type maps to a C# type, and you'll start writing real logic. The procedure specification you create today will guide the domain model you build in Sessions 4-8.

---

*Previous: [NuGet Packages and Git Basics](04-nuget-and-git.md)*
