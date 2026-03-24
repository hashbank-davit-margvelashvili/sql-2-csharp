# SQL-to-.NET Developer Transition Program

## Course Overview

A comprehensive training program designed to transition SQL Server developers into independent back-end .NET/C# developers. The course uses a project-based approach where trainees extract and rewrite real stored procedures from the legacy HSP core banking system into production .NET microservices.

| | |
|---|---|
| **Duration** | Phase 1: 3 months (24 sessions) / Phase 2: 3 months (24 sessions) |
| **Schedule** | 2 sessions per week, 2 hours per session |
| **Group Size** | 3-5 trainees |
| **Tech Stack** | .NET 10 (LTS), C# 14, SQL Server, EF Core, ASP.NET Core |
| **Target Level** | Competency Matrix Level II-III (independent back-end developers) |
| **Prerequisites** | Strong SQL Server / T-SQL knowledge, database design experience |

### Guiding Principles

- Every C# concept is anchored to something trainees already know from SQL Server / T-SQL
- The capstone project (HSP procedure extraction) runs from Session 7 onward -- no session is purely academic
- Phase 1 delivers one production microservice; Phase 2 delivers a second with full DevOps pipeline
- Small group size enables personalized mentoring, pair programming, and peer code reviews

---

## PHASE 1: Core C# + ASP.NET Core + EF Core (Sessions 1-24, ~3 months)

---

### BLOCK A: Foundations of C# (Sessions 1-8)

---

#### Session 1: Welcome to .NET -- The Ecosystem and Your First Program

**Learning Objectives:**
- Understand the .NET runtime (CLR, JIT, IL) and how it compares to SQL Server's query engine compiling T-SQL
- Set up the development environment (Visual Studio 2026, .NET 10 SDK, Git)
- Write, compile, and run a first C# console application

**Topics Covered:**
- What is .NET 10, CLR, managed code -- analogy: "SQL Server is a runtime for T-SQL; CLR is a runtime for C#"
- Solution (.sln) vs Project (.csproj) structure -- analogy: "a database is like a solution; schemas are like projects"
- `dotnet new console`, `dotnet run`, `dotnet build`
- `Program.cs`, top-level statements in C# 14
- `Console.WriteLine()`, string interpolation
- NuGet packages -- "like adding a referenced database or linked server, but for code libraries"
- Git init, first commit, .gitignore basics

**HSP Project Connection:** Discuss the HSP legacy system architecture. Identify 3-5 candidate stored procedures that could become microservices. Trainees document the inputs, outputs, and business logic of one simple procedure they own.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 1: Introducing C# and .NET
- Microsoft Learn: "Get started with C#" tutorial
- Tim Corey YouTube: "Intro to C# and .NET"

---

#### Session 2: Types, Variables, and Expressions -- Thinking Beyond SQL Data Types

**Learning Objectives:**
- Map SQL Server data types to C# types confidently
- Understand value types vs reference types (a concept absent in SQL)
- Use operators, expressions, and type conversions

**Topics Covered:**
- Built-in types: `int`/`long` (INT/BIGINT), `decimal` (DECIMAL), `bool` (BIT), `string` (NVARCHAR), `DateTime`/`DateOnly`/`TimeSpan` (DATETIME/DATE)
- `var` keyword -- type inference, not dynamic typing
- Value types vs reference types -- "an `int` variable holds the number itself; a `string` variable holds a reference to the text, like a pointer"
- Constants (`const`, `readonly`)
- Arithmetic, logical, comparison operators -- mostly identical to T-SQL
- Null-coalescing `??` and `??=` -- "like ISNULL() or COALESCE() but as an operator"
- Null-conditional `?.` -- no direct T-SQL equivalent, explain the power
- Type casting: `(int)`, `Convert.ToInt32()`, `int.Parse()`, `int.TryParse()` -- "like CAST and TRY_CAST"
- String manipulation: `Length`, `Substring`, `Contains`, `Replace`, `Split`, interpolation

**Hands-on Exercise:** Write a console app that takes a transaction amount and currency code as input, performs conversions, and outputs formatted results (banking-relevant).

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 2: C# Language Basics
- Microsoft Learn: "C# type system"

---

#### Session 3: Control Flow and Methods -- From IF/WHILE to C# Statements

**Learning Objectives:**
- Write conditional and loop statements, mapping them from T-SQL equivalents
- Define and call methods with parameters and return values
- Understand scope and the call stack

**Topics Covered:**
- `if/else if/else` -- "you know IF...ELSE in T-SQL, identical logic, different syntax"
- `switch` statements and switch expressions (C# pattern matching syntax)
- Ternary operator `? :` -- "like IIF() in T-SQL"
- Loops: `for`, `foreach`, `while`, `do-while` -- "WHILE exists in T-SQL; `for` and `foreach` are more structured versions"
- `break`, `continue`, `return`
- Methods: parameters, return types, `void` -- "a method is like a stored procedure that returns a value; `void` is like a procedure with no output parameter"
- Optional parameters and default values -- "like parameters with defaults in stored procedures"
- `ref`, `out`, `params` -- "`out` is like an OUTPUT parameter in a stored procedure"
- Method overloading -- "same procedure name, different parameter signatures -- SQL doesn't allow this, but C# does"

**Hands-on Exercise:** Rewrite a small piece of T-SQL business logic (e.g., a fee calculation procedure) as C# methods in a console app.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 2: C# Language Basics (continued)
- Microsoft Learn: "Methods in C#"

---

#### Session 4: Classes, Objects, and Access Modifiers -- The Building Blocks

**Learning Objectives:**
- Define classes with fields, properties, and constructors
- Understand encapsulation and access modifiers
- See how a class models a concept that would be a table + stored procedures in SQL

**Topics Covered:**
- Classes and objects -- "a class is like a table definition; an object is like a row, but it also contains behavior (methods)"
- Fields vs properties (`{ get; set; }`, `{ get; init; }`) -- "columns vs computed columns with controlled access"
- Constructors and constructor chaining -- "like a default value + constraint setup when INSERTing a row"
- Access modifiers: `public`, `private`, `protected`, `internal` -- "SQL has GRANT/DENY for access; C# uses these keywords to control who sees what"
- `static` classes and members -- "like a scalar function that doesn't need a table -- it just exists"
- Namespaces and `using` -- "like schema qualification: `dbo.GetCustomer` becomes `Banking.Services.CustomerService`"
- `partial` classes (awareness)

**Hands-on Exercise:** Model a `BankAccount` class with properties (AccountNumber, Balance, Currency, Status), a constructor, and methods (Deposit, Withdraw with validation). Demonstrate encapsulation by making Balance private with controlled access.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 3: Creating Types in C#
- Microsoft Learn: "Classes and objects"

---

#### Session 5: Inheritance, Interfaces, and Polymorphism

**Learning Objectives:**
- Implement class inheritance and understand when to use it
- Define and implement interfaces as contracts
- Understand polymorphism -- treating different implementations through a common interface

**Topics Covered:**
- Inheritance (`: BaseClass`) -- "like a view that extends a base table with additional logic"
- `virtual`, `override`, `abstract`, `sealed`
- `base` keyword for calling parent members
- Interfaces -- "an interface is a contract, like a stored procedure signature that multiple implementations can fulfill"
- `abstract class` vs `interface` -- when to use each
- Polymorphism -- "imagine different payment processors (card, wire, cash) all implementing `IPaymentProcessor.Process()` -- the calling code doesn't care which one it gets"
- OOP Pillar 1: Encapsulation (recap from Session 4)
- OOP Pillar 2: Abstraction -- hiding complexity behind simple interfaces
- OOP Pillar 3: Inheritance -- reuse and extend
- OOP Pillar 4: Polymorphism -- interchangeable implementations

**Hands-on Exercise:** Create a `Transaction` base class and derive `DepositTransaction`, `WithdrawalTransaction`, `TransferTransaction`. Create an `ITransactionValidator` interface with multiple implementations.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 3: Creating Types in C# (Inheritance, Interfaces)
- Refactoring.Guru: "What is Polymorphism?"

---

#### Session 6: Structs, Enums, Records, Tuples, and Other Type Constructs

**Learning Objectives:**
- Choose the right type construct for different situations
- Use records for immutable data transfer objects
- Understand value semantics vs reference semantics in depth

**Topics Covered:**
- `struct` -- value-type alternative to class, when to use (small, immutable data like `Money(decimal Amount, string Currency)`)
- `enum` -- "like a lookup table with fixed values: `enum TransactionStatus { Pending, Completed, Failed }`"
- `record` and `record struct` -- immutable data types with value equality, perfect for DTOs
- Tuples: `(int Id, string Name)` -- "like returning multiple columns from a query without creating a table type"
- Named tuples vs anonymous tuples
- Nullable value types `int?` -- "like a NULLable INT column"
- Nullable reference types `string?` and `#nullable enable` -- compile-time null safety
- Anonymous types `new { Name = "test", Amount = 100 }` -- "like a SELECT without INTO"

**Hands-on Exercise:** Refactor the banking model from Sessions 4-5 using records for DTOs, enums for statuses, and structs for value objects like `Money`.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 3: Creating Types (Structs, Enums)
- *C# 12 in a Nutshell* -- Chapter 4: Advanced C# (Records, Tuples)

---

#### Session 7: Collections and LINQ -- Your SQL Superpowers in C#

**Learning Objectives:**
- Use all major collection types and know when each is appropriate
- Write LINQ queries leveraging existing SQL query-writing intuition
- Understand deferred execution

**Topics Covered:**
- Arrays `int[]` -- fixed-size, fast access
- `List<T>` -- "the workhorse, like a temp table you can add to and remove from"
- `Dictionary<TKey, TValue>` -- "like a hash index lookup table"
- `Queue<T>` (FIFO), `Stack<T>` (LIFO), `HashSet<T>` (unique items)
- **LINQ -- "SQL for C# objects":**
  - `Where()` = WHERE, `Select()` = SELECT, `OrderBy()` = ORDER BY
  - `FirstOrDefault()` = TOP 1, `Any()` = EXISTS, `Count()` = COUNT
  - `GroupBy()` = GROUP BY, `Distinct()` = DISTINCT, `Skip()`/`Take()` = OFFSET/FETCH
  - `Sum()`, `Min()`, `Max()`, `Average()` = aggregate functions
  - `Join()` = JOIN (but usually avoided in favor of navigation properties in EF)
  - Query syntax vs method syntax (show both, prefer method syntax)
  - **Deferred execution** -- "the query doesn't run until you enumerate it, like a view definition vs `SELECT * FROM view`"

**Hands-on Exercise:** Given a `List<Transaction>` with 100 sample banking transactions, write 15+ LINQ queries: filter by date range, group by category, calculate running balances, find duplicates, etc. Compare the C# LINQ with equivalent T-SQL side by side.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 8: LINQ Queries
- Microsoft Learn: "LINQ overview"
- Tim Corey YouTube: "Intro to LINQ"

---

#### Session 8: Delegates, Lambdas, Events, Generics, and Advanced Language Features

**Learning Objectives:**
- Understand delegates as type-safe function references and use `Func<>`, `Action<>`, and lambdas
- Write generic classes and methods with constraints
- Use extension methods, pattern matching, and attributes

**Topics Covered:**
- Delegates -- "a variable that holds a reference to a method, like a function pointer"
- `Func<T, TResult>` (returns a value), `Action<T>` (returns void), `Predicate<T>` (returns bool)
- Lambda expressions `=>` -- "the arrow function of C#; you already used them in LINQ"
- Events -- publisher/subscriber pattern (awareness level; deeper in Phase 2)
- Generics -- `class Repository<T>`, `T GetById<T>(int id)` -- "like a stored procedure that works for any table"
- Generic constraints: `where T : class`, `where T : IEntity`
- Extension methods -- "adding a method to an existing type without modifying it: `myString.ToTitleCase()`"
- Pattern matching -- `is`, property patterns, switch expressions with patterns
- Attributes -- `[Obsolete]`, `[Required]`, custom attributes
- Operator overloading -- `Money + Money`

**Hands-on Exercise:** Create a generic `Repository<T>` class that stores entities in a `List<T>`, with methods like `Add`, `GetById`, `Find(Predicate<T>)`. Use extension methods to add a `.ToDisplayString()` on the `Transaction` class.

**HSP Project Checkpoint:** By now, trainees should have selected their target HSP stored procedure and documented its full specification: input parameters, output structure, business rules, error handling, dependent tables. Begin modeling the domain in C# classes.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 4: Advanced C# (Delegates, Events, Generics)
- Microsoft Learn: "Generics in .NET"

---

### BLOCK B: Exception Handling, Serialization, and the Bridge to Web (Sessions 9-12)

---

#### Session 9: Exception Handling and Resource Management

**Learning Objectives:**
- Implement structured exception handling (try/catch/finally)
- Create custom exception types for domain-specific errors
- Understand `IDisposable` and the `using` statement

**Topics Covered:**
- `try / catch / finally` -- "like TRY...CATCH in T-SQL, but more structured"
- Exception hierarchy: `Exception` > `SystemException` > `ArgumentException`, `InvalidOperationException`, `NullReferenceException`, etc.
- Throwing exceptions: `throw new ArgumentException("...")`
- Creating custom exceptions: `InsufficientFundsException`, `AccountNotActiveException`
- When to catch vs when to let propagate -- "don't swallow exceptions; handle what you can, let the rest bubble up"
- `IDisposable` and `using` statement -- "like closing a cursor or connection when you're done"
- `using` declaration (C# 8+)

**Hands-on Exercise:** Add comprehensive error handling to the banking model. Throw `InsufficientFundsException` on overdraft, `AccountFrozenException` on frozen accounts. Write calling code that catches and handles each appropriately.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 4: Advanced C# (Exceptions, Disposal)
- Microsoft Learn: "Exception handling"

---

#### Session 10: Streams, I/O, Serialization, and HttpClient

**Learning Objectives:**
- Read and write files using streams
- Serialize/deserialize objects to JSON and XML
- Make HTTP calls using `HttpClient`

**Topics Covered:**
- File I/O: `File.ReadAllText()`, `File.WriteAllText()`, `StreamReader`, `StreamWriter`
- `MemoryStream` -- working with data in memory as a stream
- JSON serialization with `System.Text.Json` -- "converting a C# object to JSON is like FOR JSON in SQL Server"
- `JsonSerializer.Serialize()`, `JsonSerializer.Deserialize<T>()`
- JSON options: camelCase naming, ignoring nulls, custom converters
- XML serialization with `XmlSerializer` (brief, for legacy integration awareness)
- `HttpClient` -- making GET/POST requests -- "calling an API is like calling a linked server, but over HTTP"
- `HttpResponseMessage`, reading response content, deserializing API responses
- Dates and times deep dive: `DateTime`, `DateOnly`, `TimeOnly`, `DateTimeOffset`, `TimeSpan` -- "like DATETIME vs DATETIMEOFFSET in SQL"
- String formatting: `ToString("C")`, `ToString("yyyy-MM-dd")`, composite formatting

**Hands-on Exercise:** Write a console app that calls a public exchange rate API, deserializes the JSON response into C# records, and writes a formatted report to a file. Bonus: read HSP procedure specs from a JSON file.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 15: Streams and I/O
- *C# 12 in a Nutshell* -- Chapter 11: Other XML/JSON Technologies
- Microsoft Learn: "HttpClient class"

---

#### Session 11: HTTP, REST Fundamentals, and Your First ASP.NET Core API

**Learning Objectives:**
- Understand HTTP methods, status codes, and REST principles
- Create a minimal ASP.NET Core Web API project
- Build a first controller with basic CRUD actions

**Topics Covered:**
- HTTP protocol: methods (GET, POST, PUT, PATCH, DELETE), status codes (200, 201, 204, 400, 401, 404, 409, 500)
- REST principles: resource-based URLs, statelessness, uniform interface
- Request/response structure: headers, body, content-type
- `dotnet new webapi` -- project structure walkthrough
- `Program.cs` -- application startup, service registration, middleware pipeline
- What middleware is -- "like triggers that fire on every request in sequence"
- First controller: `[ApiController]`, `[Route("api/[controller]")]`, `ControllerBase`
- Action methods: `[HttpGet]`, `[HttpPost]`, `[HttpGet("{id}")]`
- Return types: `IActionResult`, `ActionResult<T>`, `Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`
- Route parameters, query parameters, request body: `[FromRoute]`, `[FromQuery]`, `[FromBody]`
- Testing with Swagger UI

**Hands-on Exercise:** Create a `TransactionsController` with in-memory data (a `List<Transaction>`). Implement GET all, GET by id, POST (create), PUT (update), DELETE. Test with Swagger.

**HSP Project Connection:** Map the HSP stored procedure's interface to REST endpoints. "This procedure takes @AccountId and @Amount as input and returns a result set -- that becomes `POST /api/transfers` with a JSON body `{ accountId, amount }` returning a response object."

**Learning Materials:**
- *ASP.NET Core in Action, 3rd Ed.* -- Chapters 1-5
- Microsoft Learn: "Create web APIs with ASP.NET Core"
- Tim Corey YouTube: "ASP.NET Core Web API Tutorial"

---

#### Session 12: Dependency Injection, Services, and Configuration

**Learning Objectives:**
- Understand Dependency Injection as the backbone of ASP.NET Core
- Register and inject services with proper lifetimes
- Use configuration and the Options pattern

**Topics Covered:**
- Why DI matters -- "instead of a stored procedure creating its own connections, imagine if the connection was passed in -- that's DI"
- The DI container in `Program.cs`
- Service lifetimes: `AddTransient<T>()` (new instance every time), `AddScoped<T>()` (one per request -- like a transaction scope), `AddSingleton<T>()` (one for the app lifetime)
- Constructor injection -- the standard pattern
- Registering interfaces to implementations: `builder.Services.AddScoped<ITransactionService, TransactionService>()`
- `appsettings.json` and `appsettings.Development.json`
- Reading configuration with `IConfiguration`
- The Options pattern: `builder.Services.Configure<DatabaseSettings>(config.GetSection("Database"))`
- `IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`
- Environment-specific configuration -- Development vs Production

**Hands-on Exercise:** Refactor the TransactionsController to use an `ITransactionService` injected through DI. Move business logic out of the controller into the service. Add configuration for things like max transaction amount from `appsettings.json`.

**Learning Materials:**
- *ASP.NET Core in Action, 3rd Ed.* -- Chapters 8-10 (DI, Configuration)
- Microsoft Learn: "Dependency injection in ASP.NET Core"

---

### BLOCK C: Data Access with EF Core (Sessions 13-16)

---

#### Session 13: Entity Framework Core -- DbContext, Code First, and Entity Modeling

**Learning Objectives:**
- Set up EF Core with SQL Server
- Define entity models and configure them with Fluent API
- Create and apply migrations

**Topics Covered:**
- What is an ORM -- "instead of writing `SELECT * FROM Accounts WHERE Id = @id`, you write `dbContext.Accounts.Find(id)` -- EF generates the SQL"
- Installing EF Core packages: `Microsoft.EntityFrameworkCore.SqlServer`, `Microsoft.EntityFrameworkCore.Tools`
- `DbContext` -- "the connection + session with your database; like a SqlConnection that understands your schema"
- `DbSet<T>` -- "represents a table"
- Code First approach: C# classes define the schema, EF creates the database
- Entity modeling: navigation properties, foreign keys -- "you know how to design tables and relationships; now express them as C# classes"
- Entity configuration with Fluent API in `IEntityTypeConfiguration<T>` -- "like adding constraints, indexes, and column definitions, but in C#"
- Data Annotations vs Fluent API (prefer Fluent API for complex config)
- Migrations: `dotnet ef migrations add InitialCreate`, `dotnet ef database update`

**Hands-on Exercise:** Model the HSP procedure's underlying tables as EF Core entities. Create a `DbContext`, configure relationships with Fluent API, generate and apply a migration. Compare the generated SQL schema with the original HSP tables.

**Learning Materials:**
- *Entity Framework Core in Action, 2nd Ed.* -- Chapters 1-4
- Microsoft Learn: "Getting started with EF Core"

---

#### Session 14: EF Core Querying and CRUD Operations

**Learning Objectives:**
- Perform CRUD operations through EF Core
- Write efficient queries using LINQ-to-Entities
- Understand change tracking and `SaveChangesAsync()`

**Topics Covered:**
- Querying: `dbContext.Accounts.Where(a => a.Status == AccountStatus.Active).ToListAsync()`
- "Your LINQ skills from Session 7 now generate real SQL -- EF translates LINQ to T-SQL"
- Eager loading with `.Include()` -- "like a JOIN that loads related entities"
- Filtering, sorting, paging: `.Where()`, `.OrderBy()`, `.Skip()`, `.Take()`
- Projections: `.Select(a => new AccountDto { ... })` -- "like selecting specific columns instead of `SELECT *`"
- Finding by key: `.FindAsync(id)`
- Adding: `.Add(entity)` + `SaveChangesAsync()`
- Updating: modify tracked entity + `SaveChangesAsync()` -- "EF tracks changes like SQL Server tracks modifications in a transaction"
- Change tracking explained -- how EF knows what changed
- Deleting: `.Remove(entity)` + `SaveChangesAsync()`
- `AsNoTracking()` for read-only queries -- performance optimization
- **Async all the way**: every DB operation uses `await` and `Async` methods

**Hands-on Exercise:** Connect the `TransactionService` from Session 12 to a real SQL Server database through EF Core. Implement full CRUD. Use SQL Server Profiler or EF Core logging to see the generated SQL and compare with hand-written T-SQL.

**Learning Materials:**
- *Entity Framework Core in Action, 2nd Ed.* -- Chapters 5-7
- Microsoft Learn: "Querying data with EF Core"

---

#### Session 15: EF Core Advanced Configuration, Migrations Workflow, and SqlClient

**Learning Objectives:**
- Configure complex entity relationships and constraints
- Manage migrations in a team environment
- Use raw SQL and SqlClient when EF Core is not the right tool

**Topics Covered:**
- Advanced Fluent API: composite keys, unique constraints, indexes, value conversions, owned types
- Seed data in migrations
- Migration workflow: adding, applying, reverting, scripting (`dotnet ef migrations script`)
- When EF Core is not enough -- using raw SQL: `dbContext.Database.ExecuteSqlRawAsync()`
- `SqlClient` (Microsoft.Data.SqlClient) -- "for those cases where you need raw ADO.NET, like calling an existing HSP stored procedure"
- `SqlConnection`, `SqlCommand`, `SqlDataReader` -- manual data access
- Calling stored procedures from C# -- both via SqlClient and EF Core
- When to use EF Core vs SqlClient vs raw SQL -- decision framework

**HSP Project Connection:** Use SqlClient to call the original HSP stored procedure from C# to verify behavior. Then implement the same logic as a proper .NET service with EF Core. Compare outputs.

**Learning Materials:**
- *Entity Framework Core in Action, 2nd Ed.* -- Chapter 8-10
- Microsoft Learn: "Raw SQL queries with EF Core"

---

#### Session 16: Validation, Error Handling in APIs, and Middleware

**Learning Objectives:**
- Implement input validation using FluentValidation
- Build consistent API error handling with middleware and filters
- Understand the ASP.NET Core middleware pipeline

**Topics Covered:**
- Data Annotations: `[Required]`, `[Range]`, `[MaxLength]`, `[RegularExpression]`
- FluentValidation -- "more powerful, testable validation: `RuleFor(x => x.Amount).GreaterThan(0).WithMessage(...)`"
- Registering validators in DI
- Global error handling middleware -- catching unhandled exceptions and returning consistent error responses
- `ProblemDetails` for standardized error format (RFC 7807)
- Exception filters: `IExceptionFilter`
- Action filters: `IActionFilter` -- "like triggers that run before/after every action"
- Custom middleware: logging request/response, correlation IDs
- Model binding and `[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromHeader]`

**Hands-on Exercise:** Add FluentValidation to the HSP microservice. Create a global exception handling middleware that returns `ProblemDetails`. Add a request logging middleware that logs each request with a correlation ID.

**Learning Materials:**
- *ASP.NET Core in Action, 3rd Ed.* -- Chapters 6-7 (Middleware, Filters)
- FluentValidation documentation: https://docs.fluentvalidation.net/

---

### BLOCK D: Async, Logging, Mapping, and API Polish (Sessions 17-20)

---

#### Session 17: Async/Await Deep Dive and Concurrency Basics

**Learning Objectives:**
- Understand the Task-based asynchronous pattern thoroughly
- Avoid common async pitfalls (deadlocks, async void, fire-and-forget)
- Understand threads vs tasks

**Topics Covered:**
- Why async matters in web APIs -- "SQL Server has its own concurrency model with worker threads; ASP.NET Core uses async to avoid blocking threads while waiting for I/O"
- `Task` and `Task<T>` -- represent an asynchronous operation
- `async` and `await` -- how the state machine works (conceptual)
- `async Task` vs `async Task<T>` vs `async void` -- "never use `async void` except for event handlers"
- `Task.WhenAll()` -- running multiple independent operations in parallel
- `Task.WhenAny()` -- waiting for the first completion
- Cancellation with `CancellationToken` -- "like aborting a long-running query"
- Threading basics: `Thread`, `ThreadPool`, `Task.Run()` -- know the difference, prefer Tasks
- Common pitfalls: `.Result` and `.Wait()` causing deadlocks, not awaiting tasks, exception handling in async code
- `.ConfigureAwait(false)` -- when and why (library code)

**Hands-on Exercise:** Make all service and repository methods async. Add `CancellationToken` support. Implement a scenario where two independent database queries run in parallel with `Task.WhenAll()`.

**Learning Materials:**
- *C# 12 in a Nutshell* -- Chapter 14: Concurrency and Asynchrony
- Microsoft Learn: "Asynchronous programming with async and await"
- Nick Chapsas YouTube: "How async/await really works in C#"

---

#### Session 18: Logging with Serilog and Structured Logging

**Learning Objectives:**
- Configure Serilog for structured logging
- Write meaningful log entries with appropriate levels and context
- Understand structured logging vs text logging

**Topics Covered:**
- Why logging matters -- "in SQL you might use PRINT or log tables; in .NET, logging is systematic"
- Built-in `ILogger<T>` -- log levels: Trace, Debug, Information, Warning, Error, Critical
- Structured logging -- "don't log `$"User {userId} transferred {amount}"`, log `Log.Information("Transfer completed", new { UserId = userId, Amount = amount })` so you can query it"
- Setting up Serilog: install packages, configure in `Program.cs`
- Serilog sinks: Console, File, Seq (for viewing structured logs)
- Log scopes -- adding context (CorrelationId, UserId) to all log entries in a request
- What NOT to log: passwords, PII, full request bodies with sensitive data (banking awareness)
- Log level configuration per namespace in `appsettings.json`

**Hands-on Exercise:** Add Serilog to the HSP microservice. Log at appropriate levels: Information for successful operations, Warning for business rule violations, Error for exceptions. Add a correlation ID to all logs via middleware.

**Learning Materials:**
- Serilog documentation: https://serilog.net/
- Nick Chapsas YouTube: "Structured logging in ASP.NET Core with Serilog"

---

#### Session 19: Object Mapping, Swagger/OpenAPI, and API Versioning

**Learning Objectives:**
- Use Mapperly (or AutoMapper) to map between entities, DTOs, and API models
- Configure Swagger/OpenAPI documentation
- Implement API versioning

**Topics Covered:**
- Why mapping matters -- separation between database entities, domain models, and API DTOs
- Mapperly (compile-time, source-generated mapper) -- preferred for new projects
  - Defining mapper classes, mapping conventions, custom mappings
- AutoMapper (runtime reflection-based) -- know it exists, understand the difference
- Swagger/OpenAPI specification:
  - `Swashbuckle.AspNetCore` configuration
  - XML comments for API documentation
  - Describing request/response models
  - Grouping endpoints, adding examples
- API versioning:
  - URL path versioning: `/api/v1/transactions`
  - Header versioning
  - `Asp.Versioning.Mvc` package
- Routing deep dive: attribute routing, conventional routing, route constraints

**Hands-on Exercise:** Create separate DTOs for the HSP microservice (CreateTransactionRequest, TransactionResponse, TransactionListResponse). Configure Mapperly for all mappings. Add API versioning. Document all endpoints in Swagger with descriptions and examples.

**Learning Materials:**
- Mapperly documentation: https://mapperly.riok.app/
- Microsoft Learn: "ASP.NET Core web API documentation with Swagger / OpenAPI"

---

#### Session 20: Putting It Together -- First HSP Microservice Completion

**Learning Objectives:**
- Complete a production-quality microservice end-to-end
- Apply all learned concepts in an integrated, working service
- Conduct a code review as a team

**Topics Covered:**
- Review and refactor the HSP microservice built over Sessions 11-19:
  - Clean project structure: Controllers / Services / Repositories / Models / DTOs / Validators / Mappers
  - Consistent error handling and validation
  - Proper async/await throughout
  - Structured logging
  - Swagger documentation
  - Configuration with Options pattern
- Code organization principles applied:
  - DRY -- no duplicated logic
  - YAGNI -- no speculative features
  - KISS -- simplest solution that works
  - Separation of Concerns -- each class has one job
- Equality and comparison: `IEquatable<T>`, `IComparable<T>`, `==` overloading -- when needed in domain models

**HSP Project Milestone:** The first HSP stored procedure has been fully extracted into a .NET microservice with: REST API, EF Core data access, FluentValidation, Serilog logging, Swagger documentation. Trainees demo their microservice to the group. Peer code review session.

**Learning Materials:**
- Review all previous materials
- *Clean Architecture* -- Introduction (preview for Phase 2)

---

### BLOCK E: OOP Deep Dive and SOLID (Sessions 21-24)

---

#### Session 21: OOP Principles Applied -- Encapsulation, Abstraction, and Composition

**Learning Objectives:**
- Apply encapsulation beyond simple private fields -- information hiding in architecture
- Use composition over inheritance as a design strategy
- Understand delegation and how it differs from inheritance

**Topics Covered:**
- Encapsulation revisited -- "it's not just `private` fields; it's designing classes so that internal changes don't break external code"
- Abstraction revisited -- "hiding complexity; your `ITransactionService` hides whether data comes from EF Core, SqlClient, or a cached store"
- Composition vs Inheritance -- "prefer 'has-a' over 'is-a': a `TransferService` HAS a `IFeeCalculator` and `IAccountRepository` rather than inheriting from a base class"
- Delegation -- forwarding work to composed objects
- Real refactoring exercise: identify inheritance misuse in sample code and refactor to composition

**Hands-on Exercise:** Refactor the HSP microservice: extract fee calculation into a composed `IFeeCalculator`, extract notification into `INotificationService`. Show how the service is now flexible -- swap implementations without changing the service.

**Learning Materials:**
- *Clean Architecture* -- Chapter 5-6: Object-Oriented Programming
- Refactoring.Guru: "Composition over Inheritance"

---

#### Session 22: SOLID Principles -- SRP, OCP, LSP

**Learning Objectives:**
- Apply Single Responsibility Principle to class and method design
- Understand Open/Closed Principle with practical extension examples
- Recognize Liskov Substitution violations

**Topics Covered:**
- **SRP (Single Responsibility Principle)** -- "a class should have only one reason to change"
  - Banking example: `TransactionService` should not also handle email notifications
  - "In SQL, a stored procedure that handles validation, business logic, logging, and email sending violates SRP"
  - Refactoring exercise: split a "god class" into focused classes
- **OCP (Open/Closed Principle)** -- "open for extension, closed for modification"
  - Example: adding a new transaction type without modifying existing code
  - Strategy pattern introduction (interfaces + DI = OCP)
- **LSP (Liskov Substitution Principle)** -- "any subclass should be usable in place of its parent without breaking the program"
  - Classic violation: `Square : Rectangle` where `SetWidth` breaks invariants
  - Banking example: `ReadOnlyAccount` that throws on `Withdraw()` violates LSP if it extends `Account`

**Hands-on Exercise:** Review the HSP microservice for SOLID violations. Refactor one concrete violation of each principle.

**Learning Materials:**
- *Clean Architecture* -- Chapters 7-11: SOLID Principles
- Microsoft Learn: "SOLID principles"

---

#### Session 23: SOLID Principles -- ISP, DIP, and Dependency Injection Revisited

**Learning Objectives:**
- Design focused interfaces following ISP
- Architect layers with DIP -- high-level modules depend on abstractions
- See how ASP.NET Core DI is the mechanism that enables DIP

**Topics Covered:**
- **ISP (Interface Segregation Principle)** -- "clients should not depend on methods they don't use"
  - Example: `IAccountService` with 20 methods vs splitting into `IAccountReader`, `IAccountWriter`, `IAccountAdmin`
  - "In SQL, you might have one massive stored procedure that does everything -- ISP says split it"
- **DIP (Dependency Inversion Principle)** -- "high-level modules should not depend on low-level modules; both should depend on abstractions"
  - The `TransactionService` (business logic) depends on `ITransactionRepository` (abstraction), not `SqlTransactionRepository` (implementation)
  - This is WHY we use Dependency Injection -- DI is the mechanism, DIP is the principle
- Revisiting DI with deeper understanding:
  - Multiple implementations of the same interface
  - Keyed services in .NET 10
  - Factory pattern with DI

**Hands-on Exercise:** Refactor the HSP microservice to follow ISP and DIP. Create a clean layered architecture: API layer depends on Service interfaces, Service layer depends on Repository interfaces, implementations are registered in DI.

**Learning Materials:**
- *Clean Architecture* -- Chapters 7-11 (continued)
- Nick Chapsas YouTube: "Dependency Injection explained"

---

#### Session 24: Phase 1 Review, Assessment, and Project Showcase

**Learning Objectives:**
- Demonstrate competency across all Phase 1 topics
- Present the completed HSP microservice to stakeholders
- Identify areas for Phase 2 growth

**Topics Covered:**
- Comprehensive review covering:
  - C# language features (types, collections, LINQ, generics, async)
  - OOP and SOLID principles
  - ASP.NET Core (controllers, middleware, DI, configuration)
  - EF Core (modeling, querying, migrations)
  - Error handling, validation, logging
  - Code organization (DRY, YAGNI, KISS, Separation of Concerns)
- Each trainee presents their HSP microservice:
  - Architecture walkthrough
  - Live demo
  - Code quality discussion
  - Comparison with original stored procedure: lines of code, maintainability, testability
- Self-assessment against competency matrix Level I-II
- Knowledge gaps identification for Phase 2 planning

**Phase 1 Deliverable:** A complete, working, documented .NET microservice that replaces one HSP stored procedure. Source code in Git with meaningful commit history. API documentation in Swagger.

---

## PHASE 2: Testing, Docker, Messaging, Architecture, and DevOps (Sessions 25-48, ~3 months)

---

### BLOCK F: Testing in Depth (Sessions 25-30)

---

#### Session 25: Unit Testing Fundamentals with xUnit

**Learning Objectives:**
- Set up a test project and write first unit tests
- Apply the AAA (Arrange-Act-Assert) pattern consistently
- Use `[Fact]` and `[Theory]` with `[InlineData]`

**Topics Covered:**
- Why testing matters -- "in SQL, you might run a procedure and check results manually; automated tests do this for every change, every time"
- xUnit setup: `dotnet new xunit`, adding reference to the main project
- Test project naming conventions: `ProjectName.Tests`, `ProjectName.UnitTests`
- `[Fact]` -- a single test case
- `[Theory]` + `[InlineData]` -- parameterized tests, "like running a stored procedure with different parameter combinations"
- AAA Pattern:
  - **Arrange** -- set up the data and dependencies
  - **Act** -- call the method under test
  - **Assert** -- verify the result
- `Assert.Equal()`, `Assert.True()`, `Assert.False()`, `Assert.Null()`, `Assert.NotNull()`, `Assert.Throws<T>()`
- Test naming: `MethodName_Scenario_ExpectedResult`
- Running tests: `dotnet test`, Test Explorer in Visual Studio

**Hands-on Exercise:** Write 10 unit tests for the fee calculation logic from the HSP microservice. Cover happy paths, edge cases (zero amount, negative amount, max amount), and error cases.

**Learning Materials:**
- *Unit Testing Principles, Practices, and Patterns* -- Chapters 1-3
- Microsoft Learn: "Unit testing in .NET"

---

#### Session 26: FluentAssertions and Testing Best Practices

**Learning Objectives:**
- Write expressive assertions with FluentAssertions
- Organize tests with Setup/Teardown and shared fixtures
- Understand what makes a good unit test

**Topics Covered:**
- FluentAssertions -- "more readable assertions: `result.Should().Be(expected)`, `list.Should().HaveCount(3).And.ContainSingle(x => x.IsActive)`"
- Object comparison: `.Should().BeEquivalentTo()`
- Exception assertions: `.Should().ThrowAsync<ArgumentException>().WithMessage(...)`
- Collection assertions: `.Should().ContainSingle()`, `.BeInAscendingOrder()`
- Setup and Teardown:
  - Constructor and `IDisposable` (xUnit's way -- no `[SetUp]`/`[TearDown]` like NUnit)
  - `IAsyncLifetime` for async setup
- Shared fixtures: `IClassFixture<T>`, `ICollectionFixture<T>` -- sharing expensive resources across tests
- Test isolation -- each test must be independent, no shared mutable state
- What NOT to test: private methods, framework code, trivial getters/setters

**Hands-on Exercise:** Refactor the tests from Session 25 using FluentAssertions. Add a shared fixture for common test data setup. Ensure all tests pass in any order.

**Learning Materials:**
- *Unit Testing Principles, Practices, and Patterns* -- Chapters 4-5
- FluentAssertions documentation: https://fluentassertions.com/

---

#### Session 27: Mocking with Moq -- Isolating Units Under Test

**Learning Objectives:**
- Use Moq to create mock implementations of dependencies
- Verify that mocked methods are called with expected parameters
- Understand the difference between mocks, stubs, and fakes

**Topics Covered:**
- Why mocking -- "you want to test the `TransactionService` logic, not the database; a mock replaces the real database repository"
- Mocks vs Stubs vs Fakes:
  - Stub: returns pre-configured data
  - Mock: verifies interactions (was method called? with what arguments?)
  - Fake: a working but simplified implementation (e.g., in-memory database)
- Moq basics:
  - `new Mock<ITransactionRepository>()`
  - `.Setup(x => x.GetByIdAsync(It.IsAny<int>())).ReturnsAsync(transaction)`
  - `.Verify(x => x.SaveAsync(It.Is<Transaction>(t => t.Amount > 0)), Times.Once)`
- Mocking async methods: `.ReturnsAsync()`, `.ThrowsAsync()`
- Mocking `ILogger<T>` -- verifying log calls
- Auto-mocking (awareness of libraries like AutoFixture)

**Hands-on Exercise:** Write unit tests for the `TransactionService`, mocking `ITransactionRepository`, `IFeeCalculator`, and `ILogger<T>`. Test that the service calls the repository with the correct entity, logs the right messages, and throws on invalid input.

**Learning Materials:**
- *Unit Testing Principles, Practices, and Patterns* -- Chapter 5: Mocks and test fragility
- Moq GitHub documentation

---

#### Session 28: Integration Testing with WebApplicationFactory and TestServer

**Learning Objectives:**
- Write integration tests that test the full HTTP pipeline
- Use `WebApplicationFactory` to spin up a test server
- Replace real dependencies (database) with test doubles in integration tests

**Topics Covered:**
- Unit vs Integration tests -- "unit tests check one method in isolation; integration tests check that controllers, services, repositories, and the database all work together"
- `WebApplicationFactory<Program>` -- spins up your entire ASP.NET Core app in-memory for testing
- Making HTTP requests in tests: `client.GetAsync("/api/transactions")`
- Replacing services for testing: `.WithWebHostBuilder(builder => builder.ConfigureServices(...))`
- Using EF Core in-memory database or SQL Server test containers for integration tests
- Testing the full request pipeline: serialization, routing, validation, authorization, response codes
- `TestServer` -- the underlying server that `WebApplicationFactory` creates

**Hands-on Exercise:** Write 5 integration tests for the HSP microservice: test GET all, GET by id (found and not found), POST with valid data, POST with invalid data (validation errors). Verify HTTP status codes and response bodies.

**Learning Materials:**
- *ASP.NET Core in Action, 3rd Ed.* -- Chapter 36: Testing
- Microsoft Learn: "Integration tests in ASP.NET Core"

---

#### Session 29: Test Coverage, TDD Introduction, and Testing Strategy

**Learning Objectives:**
- Measure and interpret code coverage metrics
- Practice the Red-Green-Refactor TDD cycle
- Design a testing strategy that targets 80%+ coverage

**Topics Covered:**
- Code coverage: what it measures, what it doesn't -- "100% coverage doesn't mean zero bugs, but 20% coverage means you're flying blind"
- Coverage tools: `dotnet test --collect:"XPlat Code Coverage"`, ReportGenerator
- What to aim for: 80%+ line coverage, focus on business logic
- **Test-Driven Development (TDD):**
  - Red: write a failing test first
  - Green: write the minimum code to make it pass
  - Refactor: clean up the code while keeping tests green
  - Practice: implement a new business rule using TDD (e.g., "daily transfer limit")
- Testing strategy for a microservice:
  - Unit tests: service layer, validators, mappers (fast, many)
  - Integration tests: API endpoints end-to-end (slower, fewer)
  - Testing pyramid concept

**Hands-on Exercise:** Use TDD to implement a new feature in the HSP microservice: "a customer cannot transfer more than 10,000 GEL per day." Write the test first, then the implementation, then refactor. Measure coverage before and after.

**Learning Materials:**
- *Unit Testing Principles, Practices, and Patterns* -- Chapters 6-9
- Tim Corey YouTube: "Intro to Test Driven Development"

---

#### Session 30: Testing Review and Hands-on Lab

**Learning Objectives:**
- Achieve 80%+ test coverage on the HSP microservice
- Write tests independently without guidance
- Review and critique test quality

**Topics Covered:**
- Dedicated lab session: write tests for all untested parts of the HSP microservice
- Common testing anti-patterns:
  - Testing implementation details instead of behavior
  - Brittle tests that break on refactoring
  - Tests that depend on execution order
  - Excessive mocking (testing the mocks, not the code)
- Parameterized test design: `[Theory]` with `[MemberData]` and `[ClassData]`
- Review session: each trainee presents their tests, group critique on quality and coverage

**HSP Project Milestone:** HSP microservice now has comprehensive unit and integration tests with 80%+ coverage.

---

### BLOCK G: Design Patterns (Sessions 31-33)

---

#### Session 31: Creational and Structural Patterns -- Strategy, Factory, Builder

**Learning Objectives:**
- Apply the Strategy pattern for interchangeable algorithms
- Use the Factory pattern for object creation logic
- Use the Builder pattern for constructing complex objects step by step

**Topics Covered:**
- **Strategy Pattern** -- "you already used this with DI: `IFeeCalculator` with `PercentageFeeCalculator` and `FlatFeeCalculator` are strategies"
  - When to use: multiple algorithms for the same operation, selected at runtime
  - Banking example: different interest calculation strategies for different account types
- **Factory Pattern** -- "creating objects without specifying the exact class"
  - Simple Factory vs Factory Method vs Abstract Factory
  - Banking example: `TransactionFactory.Create(TransactionType type)` returns the right subclass
- **Builder Pattern** -- "constructing complex objects step by step"
  - Banking example: building a complex transaction request with optional fields
  - Fluent builder API: `new TransactionBuilder().WithAmount(100).WithCurrency("GEL").WithDescription("...").Build()`
  - Records with `with` expressions as a simpler alternative

**Hands-on Exercise:** Implement the Strategy pattern for fee calculation in the HSP microservice (different fee strategies for different transaction types). Implement a Builder for constructing `TransactionRequest` objects in tests.

**Learning Materials:**
- *Head First Design Patterns, 2nd Ed.* -- Chapters 1-4
- Refactoring.Guru: Strategy, Factory, Builder patterns

---

#### Session 32: Structural Patterns -- Decorator, Adapter, Repository, Unit of Work

**Learning Objectives:**
- Use the Decorator pattern to add behavior without modifying existing classes
- Apply the Adapter pattern for integrating incompatible interfaces
- Implement Repository and Unit of Work patterns

**Topics Covered:**
- **Decorator Pattern** -- "wrapping an existing service to add behavior"
  - Banking example: `LoggingTransactionService` wraps `TransactionService` to add logging without modifying it
  - DI registration with decoration
- **Adapter Pattern** -- "converting one interface to another"
  - Banking example: adapting the old HSP procedure's output format to the new microservice's model
  - "You're already doing this -- your SqlClient code that calls the HSP procedure and maps results to C# objects IS an adapter"
- **Repository Pattern** -- "abstracting data access behind an interface"
  - `IRepository<T>` with `GetByIdAsync`, `GetAllAsync`, `AddAsync`, `UpdateAsync`, `DeleteAsync`
  - Generic repository vs specific repositories -- pros and cons
- **Unit of Work Pattern** -- "coordinating multiple repository changes in one transaction"
  - EF Core's `DbContext` IS a Unit of Work (`SaveChanges` commits all tracked changes)
  - When you need an explicit Unit of Work vs relying on DbContext

**Hands-on Exercise:** Implement the Repository pattern in the HSP microservice. Add a logging decorator to the transaction service. Create an adapter for the legacy HSP data format.

**Learning Materials:**
- *Head First Design Patterns, 2nd Ed.* -- Chapters on Decorator, Adapter
- Refactoring.Guru: Decorator, Adapter, Repository patterns

---

#### Session 33: Behavioral Patterns -- Observer and Practical Pattern Application

**Learning Objectives:**
- Understand the Observer pattern and its relation to events and messaging
- Know when to apply patterns and when to keep things simple
- Recognize patterns already present in the .NET framework

**Topics Covered:**
- **Observer Pattern** -- "publishers notify subscribers when something happens"
  - C# events as the built-in Observer implementation
  - Banking example: when a transaction is completed, notify audit log, notify customer, update balance
  - Connection to RabbitMQ (preview): "the Observer pattern within one service; messaging extends it across services"
- Patterns already in .NET that you're using:
  - Iterator: `foreach` and `IEnumerable<T>`
  - Strategy: DI + interfaces
  - Template Method: ASP.NET Core middleware pipeline
  - Observer: events, `IObservable<T>`
  - Builder: `WebApplicationBuilder`, `HostBuilder`
- When NOT to use patterns -- "a pattern is a solution to a recurring problem; if you don't have the problem, don't force the pattern"
- Pattern decision framework: identify the problem first, then consider if a pattern fits

**Hands-on Exercise:** Implement an event-based notification system in the HSP microservice: when a transaction is processed, raise a domain event that triggers audit logging and (simulated) customer notification. Discuss how this will evolve to use RabbitMQ in later sessions.

**Learning Materials:**
- *Head First Design Patterns, 2nd Ed.* -- Chapter on Observer
- Refactoring.Guru: Observer pattern, "When to use patterns"

---

### BLOCK H: Architecture (Sessions 34-36)

---

#### Session 34: Layered Architecture and Clean Architecture

**Learning Objectives:**
- Understand 3-tier/N-tier architecture and implement it in .NET
- Apply Clean Architecture principles with proper dependency direction
- Structure a .NET solution following Clean Architecture

**Topics Covered:**
- **3-tier architecture** -- "Presentation (API) > Business Logic (Services) > Data Access (Repositories/EF Core)"
  - Each tier only knows about the tier directly below it
  - "In the HSP world, the stored procedure is all tiers in one file; now we separate them"
- **Clean Architecture** (Robert C. Martin):
  - Concentric circles: Entities > Use Cases > Interface Adapters > Frameworks
  - The dependency rule: dependencies point inward, inner layers know nothing about outer layers
  - .NET solution structure:
    - `Domain` project: entities, value objects, domain events, repository interfaces
    - `Application` project: use cases/services, DTOs, validators, mapper interfaces
    - `Infrastructure` project: EF Core, SqlClient, external service clients
    - `WebApi` project: controllers, middleware, DI registration
  - How DI makes Clean Architecture possible -- outer layers register implementations of inner layer interfaces

**Hands-on Exercise:** Restructure the HSP microservice into a Clean Architecture solution with 4 projects. Move all code to the appropriate layer. Verify that dependency direction is correct (Domain depends on nothing, Application depends on Domain, Infrastructure depends on Application, WebApi depends on all).

**Learning Materials:**
- *Clean Architecture* -- Chapters 17-22
- Nick Chapsas YouTube: "Clean Architecture in .NET"

---

#### Session 35: Domain-Driven Design Basics -- Strategic Patterns

**Learning Objectives:**
- Understand Bounded Contexts and Ubiquitous Language
- Map the HSP system's domain boundaries
- Apply strategic DDD to microservice design decisions

**Topics Covered:**
- **DDD Strategic Patterns:**
  - Ubiquitous Language -- "the same terms used by business people and developers; in the HSP world, what does the bank call a 'transaction'? Use that exact term in code"
  - Bounded Context -- "a boundary within which a model is consistent; the 'Account' in Lending is different from 'Account' in Payments"
  - Context Mapping -- how bounded contexts relate (Partnership, Customer-Supplier, Conformist, Anti-Corruption Layer)
  - "Your HSP database is one massive bounded context with everything coupled; microservices split it into proper bounded contexts"
- Mapping the banking domain:
  - Identify bounded contexts in the HSP system: Customer Management, Account Management, Transactions, Lending, Payments, Reporting
  - Define the ubiquitous language for each context
  - Draw context maps showing relationships

**Hands-on Exercise:** Map the HSP database into bounded contexts. Identify which tables and procedures belong to which context. Define the ubiquitous language for the HSP microservice's bounded context. Document it as a glossary that both developers and business analysts can use.

**Learning Materials:**
- *Domain-Driven Design Distilled* -- Chapters 1-4
- Microsoft Learn: "Domain-driven design"

---

#### Session 36: Domain-Driven Design Basics -- Tactical Patterns

**Learning Objectives:**
- Implement Entities, Value Objects, and Aggregates
- Use Domain Events for cross-boundary communication
- Apply the tactical patterns to the HSP microservice

**Topics Covered:**
- **DDD Tactical Patterns:**
  - Entity -- "has identity, is tracked over time: `Account` with an Id"
  - Value Object -- "defined by its attributes, no identity: `Money(100, 'GEL')` -- two Money objects with the same amount and currency are equal"
  - Aggregate -- "a cluster of entities treated as a unit for data changes; `Account` aggregate contains `Balance`, `Transactions` -- you always load and save the whole aggregate"
  - Aggregate Root -- "the entry point to an aggregate; external code only references the root"
  - Domain Event -- "something that happened in the domain: `TransactionProcessed`, `AccountOpened`"
  - Domain Service -- "logic that doesn't naturally belong to any entity: `TransferService` that coordinates between two accounts"
- Rich domain models vs anemic domain models:
  - Anemic: entity is just data, all logic in services (what we've been doing)
  - Rich: entity encapsulates behavior -- `account.Withdraw(amount)` contains business rules
  - "In the HSP stored procedure, the logic is separate from the data. DDD says put the logic WITH the data."

**Hands-on Exercise:** Refactor the HSP microservice's domain layer using DDD tactical patterns. Create a `TransactionAggregate` with value objects (`Money`, `TransactionReference`). Encapsulate business rules inside the aggregate. Raise domain events on state changes.

**Learning Materials:**
- *Domain-Driven Design Distilled* -- Chapters 5-7
- Nick Chapsas YouTube: "DDD in .NET"

---

### BLOCK I: Docker and Containerization (Sessions 37-39)

---

#### Session 37: Docker Fundamentals -- Containers for .NET Developers

**Learning Objectives:**
- Understand what containers are and why they matter for microservices
- Install Docker Desktop and run basic containers
- Run a SQL Server container for development

**Topics Covered:**
- What is Docker -- "a lightweight VM that packages your app with everything it needs; no more 'it works on my machine'"
- Containers vs VMs -- containers share the OS kernel, much lighter
- Docker Desktop installation and setup on Windows
- Docker CLI basics:
  - `docker run` -- run a container
  - `docker ps` -- list running containers
  - `docker stop` / `docker start` / `docker rm`
  - `docker images` -- list images
  - `docker pull` -- download an image
- Running SQL Server in Docker: `docker run -e SA_PASSWORD=... mcr.microsoft.com/mssql/server:2022-latest`
- "Instead of installing SQL Server on every developer's machine, run it in a container"
- Volumes -- persisting data outside the container: `docker run -v sql_data:/var/opt/mssql`
- Port mapping -- exposing container ports: `-p 1433:1433`

**Hands-on Exercise:** Run SQL Server in a Docker container. Connect to it from SSMS and from the HSP microservice. Create the database and run migrations against the containerized SQL Server.

**Learning Materials:**
- *Docker in Action, 2nd Ed.* -- Chapters 1-3
- Docker official docs: "Get started"

---

#### Session 38: Dockerizing the .NET Microservice

**Learning Objectives:**
- Write a multi-stage Dockerfile for a .NET application
- Build and run the microservice as a Docker container
- Understand image layers and optimization

**Topics Covered:**
- Dockerfile syntax: `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `ENTRYPOINT`
- Multi-stage Dockerfile for .NET:
  - Build stage: `FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build`
  - Publish stage: `dotnet publish -c Release -o /app`
  - Runtime stage: `FROM mcr.microsoft.com/dotnet/aspnet:10.0`
- Image optimization: `.dockerignore`, ordering layers for cache efficiency
- Building: `docker build -t hsp-microservice:1.0 .`
- Running: `docker run -p 8080:8080 hsp-microservice:1.0`
- Environment variables in containers: `-e ConnectionStrings__DefaultConnection=...`
- Viewing logs: `docker logs <container>`
- Health checks in Dockerfile

**Hands-on Exercise:** Write a Dockerfile for the HSP microservice. Build the image, run it as a container, and test it via Swagger on `localhost:8080`. Connect it to the SQL Server container using Docker networking.

**Learning Materials:**
- *Docker in Action, 2nd Ed.* -- Chapters 4-6
- Microsoft Learn: "Containerize a .NET app"

---

#### Session 39: Docker Compose -- Multi-Container Applications

**Learning Objectives:**
- Define multi-container applications with Docker Compose
- Orchestrate the microservice, database, and supporting services together
- Use Docker Compose for local development workflow

**Topics Covered:**
- `docker-compose.yml` syntax: services, networks, volumes
- Defining the full stack:
  ```yaml
  services:
    api:
      build: .
      ports: ["8080:8080"]
      depends_on: [sqlserver]
    sqlserver:
      image: mcr.microsoft.com/mssql/server:2022-latest
      volumes: [sql_data:/var/opt/mssql]
  ```
- `docker compose up`, `docker compose down`, `docker compose logs`
- Service discovery: containers reference each other by service name (`sqlserver` instead of `localhost`)
- Environment variable files: `.env`
- Adding RabbitMQ to the compose file (preview for upcoming sessions)
- Docker networking: bridge networks, service-to-service communication

**Hands-on Exercise:** Create a `docker-compose.yml` for the HSP microservice with SQL Server. Add a `docker-compose.override.yml` for development overrides. Run the entire stack with one command. Verify everything works end-to-end.

**Learning Materials:**
- *Docker in Action, 2nd Ed.* -- Chapter 7-8
- Docker official docs: "Docker Compose overview"

---

### BLOCK J: Messaging with RabbitMQ (Sessions 40-42)

---

#### Session 40: Message Brokers and RabbitMQ Fundamentals

**Learning Objectives:**
- Understand why message brokers exist and when to use asynchronous messaging
- Learn RabbitMQ core concepts: exchanges, queues, bindings, routing keys
- Set up RabbitMQ in Docker and explore the management UI

**Topics Covered:**
- Why messaging -- "instead of Service A calling Service B synchronously (and waiting), Service A publishes a message and moves on; Service B processes it when ready"
- Synchronous (HTTP/REST) vs Asynchronous (messaging) communication -- when to use each
  - "In HSP, if Procedure A calls Procedure B and waits, that's synchronous. If Procedure A writes to a queue table and a job picks it up later, that's asynchronous -- you already understand this pattern"
- RabbitMQ core concepts:
  - **Producer** -- sends messages (your microservice publishing events)
  - **Exchange** -- routes messages to queues (Direct, Fanout, Topic types)
  - **Queue** -- stores messages until consumed
  - **Binding** -- rules connecting exchanges to queues
  - **Consumer** -- receives and processes messages
  - **Routing Key** -- the address label on a message
- RabbitMQ in Docker: `docker run -p 5672:5672 -p 15672:15672 rabbitmq:3-management`
- Management UI at `localhost:15672` -- create exchanges, queues, publish test messages

**Hands-on Exercise:** Add RabbitMQ to the Docker Compose stack. Use the management UI to create an exchange and queue, publish messages manually, and observe them being routed.

**Learning Materials:**
- *RabbitMQ in Depth* -- Chapters 1-3
- RabbitMQ official tutorials: https://www.rabbitmq.com/tutorials

---

#### Session 41: Publishing and Consuming Messages in .NET

**Learning Objectives:**
- Publish messages to RabbitMQ from a .NET application
- Consume messages with a background service
- Handle serialization and message contracts

**Topics Covered:**
- RabbitMQ .NET client (`RabbitMQ.Client` NuGet package)
- Creating a connection and channel
- Publishing messages:
  - Serialize the message to JSON
  - Publish to an exchange with a routing key
  - `IModel.BasicPublish(exchange, routingKey, body)`
- Consuming messages:
  - `EventingBasicConsumer` or `AsyncEventingBasicConsumer`
  - Acknowledging messages: `BasicAck`, `BasicNack`, `BasicReject`
  - "Acknowledgment is like COMMIT in a transaction -- until you ack, RabbitMQ keeps the message"
- Background services in ASP.NET Core: `IHostedService`, `BackgroundService`
  - Running a consumer as a hosted service
- Message contracts -- shared classes or packages that define the message schema
- Wrapping RabbitMQ in a service: `IMessagePublisher`, `IMessageConsumer`

**Hands-on Exercise:** When a transaction is processed in the HSP microservice, publish a `TransactionProcessed` event to RabbitMQ. Create a simple consumer service (another console app) that reads these events and logs them. Add both to Docker Compose.

**Learning Materials:**
- *RabbitMQ in Depth* -- Chapters 4-6
- RabbitMQ .NET client documentation

---

#### Session 42: Messaging Patterns, Error Handling, and Reliability

**Learning Objectives:**
- Implement Publish-Subscribe and Request-Response patterns
- Handle message processing failures with dead-letter queues
- Ensure reliable message delivery

**Topics Covered:**
- **Publish-Subscribe** pattern -- fanout exchange: one event, multiple consumers
  - Banking example: `TransactionProcessed` event consumed by Audit Service, Notification Service, and Reporting Service
- **Request-Response** over messaging -- RPC pattern (awareness)
- Error handling in messaging:
  - What happens when a consumer fails? Message is requeued or dead-lettered
  - Dead Letter Exchange (DLX) -- "a parking lot for messages that couldn't be processed"
  - Retry strategies: immediate retry, delayed retry with exponential backoff
  - Poison messages -- messages that always fail; move to DLX after N retries
- Message durability: persistent messages survive broker restarts
- Consumer prefetch: `BasicQos` -- controlling how many messages a consumer takes at once
- Idempotent consumers -- "processing the same message twice should produce the same result, like an idempotent stored procedure"

**Hands-on Exercise:** Implement a dead-letter queue for the HSP microservice's consumer. Simulate a processing failure and observe messages moving to the DLQ. Implement idempotent message processing (check if transaction was already processed before applying).

**Learning Materials:**
- *RabbitMQ in Depth* -- Chapters 7-9
- RabbitMQ official docs: "Dead letter exchanges"

---

### BLOCK K: Integrations, Security, and DevOps (Sessions 43-47)

---

#### Session 43: REST API Design, Resilience with Polly, and Communication Patterns

**Learning Objectives:**
- Design robust REST APIs following best practices
- Implement resilience patterns (retry, circuit breaker, timeout) with Polly
- Understand communication patterns for microservices

**Topics Covered:**
- REST API design best practices:
  - Resource naming conventions
  - Idempotence -- "PUT and DELETE should be idempotent; POST is not -- same as in database design"
  - Statelessness -- "no session state on the server; each request contains everything needed"
  - HATEOAS (awareness)
- Polly for resilience:
  - "When your microservice calls another service and it's down, what do you do? In SQL, a linked server timeout kills your query. With Polly, you can retry, wait, or fall back"
  - Retry policy: `Policy.Handle<HttpRequestException>().RetryAsync(3)`
  - Circuit Breaker: "after N failures, stop trying for a while -- like a real circuit breaker"
  - Timeout policy
  - Combining policies: retry + circuit breaker + timeout
  - Integration with `HttpClientFactory`
- Communication patterns recap:
  - Request-Response (REST, synchronous)
  - Publish-Subscribe (RabbitMQ, asynchronous)
  - When to use which

**Hands-on Exercise:** The HSP microservice needs to call an external service (e.g., currency exchange rates). Implement an `HttpClient` call with Polly retry (3 attempts with exponential backoff) and circuit breaker (break after 5 failures, wait 30 seconds).

**Learning Materials:**
- *Designing Data-Intensive Applications* -- Chapter 8: The Trouble with Distributed Systems
- Polly documentation: https://github.com/App-vNext/Polly

---

#### Session 44: Security -- OWASP, Authentication, and Authorization

**Learning Objectives:**
- Understand OWASP Top 10 threats and their mitigation in ASP.NET Core
- Implement authentication and authorization basics
- Understand OpenID Connect and OAuth2 concepts

**Topics Covered:**
- **OWASP Top 10** awareness:
  - Injection -- "you know SQL injection intimately; EF Core parameterizes everything; be careful with raw SQL"
  - Broken Authentication, Broken Access Control, Security Misconfiguration
  - XSS, CSRF -- ASP.NET Core's built-in protections
  - Sensitive Data Exposure -- "never log or return PII, card numbers, etc."
- Authentication vs Authorization -- "authentication = who are you? authorization = what can you do?"
- ASP.NET Core Identity (overview) -- user management, password hashing, roles
- **OpenID Connect / OAuth2 basics:**
  - OAuth2 flows: Authorization Code, Client Credentials (for service-to-service)
  - JWT tokens -- what they contain, how they're validated
  - `[Authorize]` attribute, policy-based authorization
  - "In the bank, you'll integrate with the existing identity provider; you need to understand the protocol"
- `builder.Services.AddAuthentication().AddJwtBearer()`
- Role-based and claims-based authorization
- Banking-specific: API key authentication for service-to-service calls

**Hands-on Exercise:** Add JWT-based authentication to the HSP microservice. Protect endpoints with `[Authorize]`. Add role-based authorization (admin can delete, regular users can only read). Test authenticated and unauthenticated requests.

**Learning Materials:**
- *ASP.NET Core in Action, 3rd Ed.* -- Chapters 14-15 (Authentication, Authorization)
- Microsoft Learn: "Introduction to Identity on ASP.NET Core"

---

#### Session 45: Git Workflows, Code Reviews, and PR Practices

**Learning Objectives:**
- Follow a branching strategy appropriate for the team
- Conduct effective code reviews
- Write meaningful commit messages and PRs

**Topics Covered:**
- Branching strategies:
  - **Git Flow** -- feature branches, develop, release, hotfix branches
  - **Trunk-Based Development** -- short-lived feature branches, frequent merges to main
  - **Release Flow** -- Microsoft's approach, release branches
  - "For your team size and banking environment, Release Flow or a simplified Git Flow works best"
- Pull Request practices:
  - PR template: description, testing steps, screenshots
  - Small PRs -- "a 2000-line PR doesn't get properly reviewed; aim for 200-400 lines"
  - Code review checklist: functionality, readability, tests, security, performance
- Effective code reviews:
  - Review for understanding, not just approval
  - Be constructive: "consider using X pattern here" not "this is wrong"
  - Review your own PR first before requesting reviews
- Branch policies: require PR, require approvals, require build to pass
- Commit message conventions: conventional commits or descriptive messages

**Hands-on Exercise:** Set up a Git repository for the HSP microservice with branch policies. Each trainee creates a feature branch, implements a small feature, creates a PR, and reviews another trainee's PR. Merge with squash.

**Learning Materials:**
- Microsoft Learn: "Git branching strategies"
- GitHub docs: "About pull requests"

---

#### Session 46: Azure DevOps and CI/CD Pipeline Basics

**Learning Objectives:**
- Understand CI/CD concepts and their value
- Set up a basic build pipeline in Azure DevOps
- Configure automated testing in the pipeline

**Topics Covered:**
- CI/CD concepts:
  - Continuous Integration -- "every commit triggers a build and test run; broken builds are caught immediately"
  - Continuous Delivery -- "every successful build produces a deployable artifact"
  - Continuous Deployment -- "every successful build deploys automatically (not always appropriate in banking)"
- Azure DevOps overview:
  - Repos, Boards, Pipelines, Artifacts, Test Plans
- Setting up a build pipeline:
  - YAML pipeline definition
  - Build steps: restore, build, test, publish
  - Pipeline triggers: on push to main, on PR
- Running tests in the pipeline:
  - `dotnet test` with coverage reporting
  - Publishing test results
  - Failing the build on test failure
- Build artifacts: publishing the Docker image or deployment package
- Pipeline variables and secrets management

**Hands-on Exercise:** Create an Azure DevOps pipeline for the HSP microservice. Configure it to: restore packages, build, run all tests (with coverage report), and build the Docker image. Set it to trigger on PRs to the main branch.

**Learning Materials:**
- Microsoft Learn: "Create your first pipeline"
- Azure DevOps documentation

---

#### Session 47: Second HSP Microservice -- End-to-End with All Concepts

**Learning Objectives:**
- Build a second HSP microservice from scratch applying all Phase 2 concepts
- Practice Clean Architecture, DDD, Docker, testing, and messaging together
- Work as a team with proper Git workflow

**Topics Covered:**
- This is a full lab session -- minimal lecture, maximum hands-on
- Each trainee selects a second HSP stored procedure to extract
- Requirements:
  - Clean Architecture solution structure
  - DDD tactical patterns in the domain layer
  - Full test coverage (unit + integration) > 80%
  - FluentValidation, Serilog, Swagger
  - Docker + Docker Compose (with SQL Server and RabbitMQ)
  - Publishes events to RabbitMQ on state changes
  - Consumes events from the first microservice (if applicable)
  - JWT authentication
  - Polly resilience for any external calls
  - Feature branch, PR, code review by another trainee
  - Azure DevOps pipeline
- Instructor provides guidance but does not drive -- trainees work independently

**HSP Project Connection:** This session validates that trainees can independently build a production-quality microservice. It directly contributes to the bank's migration from HSP to microservices.

---

#### Session 48: Final Review, Assessment, and Graduation

**Learning Objectives:**
- Demonstrate competency across all Phase 1 and Phase 2 topics
- Present both HSP microservices with architecture documentation
- Create a personal development plan for continued growth

**Topics Covered:**
- Each trainee presents both microservices:
  - Architecture decisions and rationale
  - Live demo including Docker Compose startup
  - Test suite execution and coverage report
  - Code walkthrough: Clean Architecture layers, DDD patterns
  - RabbitMQ messaging demonstration
  - CI/CD pipeline execution
- Group discussion: lessons learned, what was hardest, what clicked
- Self-assessment against competency matrix Level II-III
- Assessment by instructor against Level II-III criteria
- Topics for continued self-study:
  - Advanced EF Core (lazy/eager/explicit loading, TPH/TPT inheritance)
  - Caching (Memory and Distributed Caching, Redis)
  - Hosted and Background Services (advanced)
  - gRPC (Client and Services, Protobuf)
  - Kubernetes basics
  - Advanced concurrency (semaphores, channels)
  - CQRS and Event Sourcing
  - Saga pattern for distributed transactions
  - Elasticsearch integration
  - Advanced monitoring and observability

**Final Deliverable:** Two complete HSP microservices in Git with:
- Clean Architecture solution structure
- Domain model following DDD patterns
- REST API with Swagger documentation
- EF Core data access with migrations
- Comprehensive test suite (80%+ coverage)
- Docker + Docker Compose deployment
- RabbitMQ event publishing/consuming
- JWT authentication
- Serilog structured logging
- Azure DevOps CI/CD pipeline
- Proper Git history with PRs and code reviews

---

## SQL-to-C# Bridge Reference Guide

This table serves as a quick reference for trainees throughout the course:

| SQL Server Concept | C# / .NET Equivalent | Session |
|---|---|---|
| Database | Solution (.sln) | 1 |
| Schema | Namespace / Project | 1 |
| Table | Class / Entity | 4, 13 |
| Column | Property | 4 |
| Row | Object instance | 4 |
| Stored Procedure | Method / Service class | 3, 12 |
| SP Parameters | Method parameters | 3 |
| OUTPUT Parameters | `out` parameters / return type | 3 |
| INT, BIGINT | `int`, `long` | 2 |
| DECIMAL | `decimal` | 2 |
| VARCHAR / NVARCHAR | `string` | 2 |
| BIT | `bool` | 2 |
| DATETIME | `DateTime`, `DateTimeOffset` | 2, 10 |
| NULL | `null`, nullable types `int?` | 6 |
| ISNULL / COALESCE | `??` (null-coalescing) | 2 |
| CAST / TRY_CAST | `(int)`, `int.TryParse()` | 2 |
| IIF() | `? :` (ternary) | 3 |
| IF...ELSE | `if...else` | 3 |
| WHILE | `while`, `for`, `foreach` | 3 |
| Temp table | `List<T>` | 7 |
| WHERE | `.Where()` | 7 |
| SELECT | `.Select()` | 7 |
| ORDER BY | `.OrderBy()` | 7 |
| GROUP BY | `.GroupBy()` | 7 |
| TOP 1 | `.FirstOrDefault()` | 7 |
| EXISTS | `.Any()` | 7 |
| COUNT / SUM / AVG | `.Count()`, `.Sum()`, `.Average()` | 7 |
| OFFSET FETCH | `.Skip()`, `.Take()` | 7 |
| JOIN | `.Join()` or `.Include()` | 7, 14 |
| VIEW | LINQ query / projection | 7 |
| TRY...CATCH | `try...catch...finally` | 9 |
| FOR JSON | `JsonSerializer.Serialize()` | 10 |
| Linked Server call | `HttpClient` | 10, 43 |
| GRANT / DENY | Access modifiers | 4 |
| Transaction (BEGIN TRAN) | `SaveChangesAsync()` / Unit of Work | 14, 32 |
| Execution plan analysis | EF Core generated SQL logging | 14 |
| Queue table + job | RabbitMQ | 40-42 |
| Lookup table | `enum` | 6 |
| User-Defined Table Type | `class` / `record` | 6 |
| EXEC sp_name | Method call / Service call | 3 |
| Cursor | `foreach` | 3 |

---

## Recommended Books

### Core References (use throughout the course)

1. **"C# 12 in a Nutshell"** by Joseph Albahari (O'Reilly)
   *The definitive C# reference. Covers every language feature in depth. Best used as a reference alongside the course rather than reading cover to cover. Particularly strong on LINQ, async/await, and collections.*
   **Sessions:** All, especially 1-8

2. **"ASP.NET Core in Action, Third Edition"** by Andrew Lock (Manning, 2023)
   *The best single book on ASP.NET Core. Covers everything from project setup to middleware, DI, configuration, EF Core, authentication, and deployment.*
   **Sessions:** 11-20, 28, 44

3. **"Entity Framework Core in Action, Second Edition"** by Jon P Smith (Manning, 2021)
   *Deep coverage of EF Core with practical patterns. Especially valuable for SQL developers transitioning to an ORM, as it explains what EF generates and why.*
   **Sessions:** 13-15

### Architecture and Design

4. **"Clean Architecture: A Craftsman's Guide to Software Structure and Design"** by Robert C. Martin (Pearson, 2017)
   *The foundational text on Clean Architecture. Short enough to read in a week. Required reading before Session 34.*
   **Sessions:** 21-23, 34

5. **"Domain-Driven Design Distilled"** by Vaughn Vernon (Addison-Wesley, 2016)
   *A concise (under 200 pages) introduction to DDD. Covers strategic and tactical patterns without the weight of the original Evans book.*
   **Sessions:** 35-36

### Testing

6. **"Unit Testing Principles, Practices, and Patterns"** by Vladimir Khorikov (Manning, 2020)
   *Covers what makes a good test, mock vs stub decisions, integration testing strategies, and anti-patterns.*
   **Sessions:** 25-30

### Infrastructure

7. **"Docker in Action, Second Edition"** by Jeff Nickoloff and Stephen Kuenzli (Manning, 2019)
   *Practical guide to Docker fundamentals. Chapters 1-6 cover everything needed for the course.*
   **Sessions:** 37-39

8. **"RabbitMQ in Depth"** by Gavin M. Roy (Manning, 2017)
   *The fundamentals of AMQP, exchanges, queues, and messaging patterns.*
   **Sessions:** 40-42

### Broader Perspective

9. **"Designing Data-Intensive Applications"** by Martin Kleppmann (O'Reilly, 2017)
    *Not .NET-specific, but essential background reading for anyone building microservices in banking. Covers distributed systems, consistency, messaging, and data modeling at a conceptual level.*
    **Sessions:** Supplementary reading throughout Phase 2

---

## Online Resources

| Resource | URL | Best For |
|---|---|---|
| Microsoft Learn -- C# | https://learn.microsoft.com/en-us/dotnet/csharp/ | Language reference, tutorials |
| Microsoft Learn -- ASP.NET Core | https://learn.microsoft.com/en-us/aspnet/core/ | Framework documentation |
| Microsoft Learn -- EF Core | https://learn.microsoft.com/en-us/ef/core/ | Data access documentation |
| Tim Corey YouTube | https://www.youtube.com/@IAmTimCorey | C# fundamentals, clear explanations |
| Nick Chapsas YouTube | https://www.youtube.com/@nickchapsas | Advanced .NET, performance, new features |
| RabbitMQ Tutorials | https://www.rabbitmq.com/tutorials | Messaging patterns step-by-step |
| Docker Get Started | https://docs.docker.com/get-started/ | Container fundamentals |
| Refactoring.Guru | https://refactoring.guru/design-patterns | Visual pattern explanations |
| FluentValidation Docs | https://docs.fluentvalidation.net/ | Validation library reference |
| Serilog Docs | https://serilog.net/ | Logging library reference |
| Mapperly Docs | https://mapperly.riok.app/ | Object mapping reference |

---

## Competency Matrix Coverage Map

This table maps each competency matrix area to the course sessions that address it:

| Matrix Area | Phase 1 Sessions | Phase 2 Sessions |
|---|---|---|
| OOP | 4, 5, 6, 21 | -- |
| SOLID | 22, 23 | -- |
| C# Language (Level I) | 1-10 | -- |
| C# ASP.NET Core (Level II) | 11-12, 16-20 | 43, 44 |
| C# EF Core (Level II) | 13-15 | -- |
| C# Async/Concurrency (Level II) | 17 | -- |
| C# Testing Libraries (Level II) | -- | 25-30 |
| C# Logging (Level II) | 18 | -- |
| C# Object Mapping (Level II) | 19 | -- |
| Code Organization & Design Patterns | 20, 21 | 31-33 |
| Architecture (3-tier, Clean, DDD) | -- | 34-36 |
| Integrations & REST | 11 | 43 |
| Security | -- | 44 |
| Unit & Integration Testing | -- | 25-30 |
| Code Versioning | 1 | 45 |
| DevOps / CI-CD | -- | 46 |
| Docker & K8s | -- | 37-39 |
| RabbitMQ | -- | 40-42 |
