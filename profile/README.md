# Cirreum

<h4>Layered simplicity for modern .NET</h4>

[![FOSSA Status](https://app.fossa.com/api/projects/custom%2B59874%2FCirreum%5C.svg?type=shield&issueType=security)](https://app.fossa.com/projects/custom%2B59874%2FCirreum%5C?ref=badge_shield&issueType=security)

[![FOSSA Status](https://app.fossa.com/api/projects/custom%2B59874%2FCirreum%5C.svg?type=shield&issueType=license)](https://app.fossa.com/projects/custom%2B59874%2FCirreum%5C?ref=badge_shield&issueType=license)

<p align="right">
  <img src="https://raw.githubusercontent.com/cirreum/.github/main/profile/cirreum-icon-transparent-002.png" width="180" alt="Cirreum Logo" />
</p>

## **A modern, opinionated foundation framework built on Domain-first principles—define your models and operations once, consume them across multiple applications—with Railway-Oriented Programming ensuring clean, composable control flow throughout.**

Cirreum provides a comprehensive set of libraries that work seamlessly together to enable rapid development of enterprise-grade applications across Server (ASP.NET Core), Client (Blazor WebAssembly), and Serverless (Azure Functions) environments.

## Philosophy

Cirreum embraces **Railway-Oriented Programming** to eliminate exception-based control flow while maintaining clean, composable code. Write your business logic once, and run it anywhere—with authorization, validation, and error handling automatically applied through a powerful pipeline architecture.

```csharp
// Write once
public class GetResourceHandler : IRequestHandler<GetResourceQuery, ResourceDto> {
    public async Task<Result<ResourceDto>> Handle(GetResourceQuery query, CancellationToken ct) {
        var resource = await _repository.GetById(query.Id);
        return resource is not null
            ? Result<ResourceDto>.Success(resource)
            : Result<ResourceDto>.Fail(new NotFoundException("Resource not found"));
    }
}

// Run everywhere - Server returns HTTP 200/404, WASM updates UI, Functions map to IActionResult
await dispatcher.Dispatch(new GetResourceQuery(id), cancellationToken);
```

## Core Principles

- **Runtime-Agnostic**: Business logic works identically across Server, Client, and Serverless runtimes
- **Railway-Oriented**: Explicit success/failure paths with `Result<T>` instead of exceptions
- **Convention over Configuration**: Sensible defaults with escape hatches when needed
- **Type-Safe**: Leverage C#'s type system for compile-time safety
- **Composable**: Small, focused libraries that work better together
- **Clean Architecture**: Clear separation between business logic and infrastructure concerns

## Project Structure

Cirreum is intentionally flexible about where you place handlers and infrastructure concerns. Your domain remains pure regardless of application complexity.

### Simple Application

For smaller applications or microservices, handlers can live directly in your host project:

```
MyApp.Domain/
├── Models/                  # Public domain contracts (response data)
├── Commands/                # Input contracts with intent + validators/authorizers
├── Queries/                 # Input contracts for reads + validators/authorizers
└── Services/                # Domain service interfaces (if needed)

MyApp.Server/
├── Handlers/                # Command and query handlers
├── Entities/                # Persistence representations
├── Repositories/            # Data access implementations
├── Endpoints/               # API endpoint mappings
└── Program.cs
```

### Multi-Host Application

When multiple hosts consume the same domain, extract infrastructure to a shared layer:

```
MyApp.Domain/
├── Models/
├── Commands/
├── Queries/
└── Services/

MyApp.Infrastructure/
├── Handlers/                # Shared handler implementations
├── Entities/                # Persistence representations
├── Repositories/
└── ExternalServices/        # Third-party integrations

MyApp.Server/
├── Endpoints/
└── Program.cs

MyApp.Functions/
├── Functions/               # Azure Function triggers
└── Program.cs
```

### Enterprise Application

For larger systems with multiple bounded contexts or teams:

```
src/
├── Domain/
│   ├── MyApp.Domain.Customers/
│   │   ├── Models/
│   │   ├── Commands/
│   │   └── Queries/
│   └── MyApp.Domain.Orders/
│       ├── Models/
│       ├── Commands/
│       └── Queries/
│
├── Infrastructure/
│   ├── MyApp.Infrastructure.Customers/
│   │   ├── Handlers/
│   │   ├── Entities/
│   │   └── Repositories/
│   └── MyApp.Infrastructure.Orders/
│       ├── Handlers/
│       ├── Entities/
│       └── Repositories/
│
└── Hosts/
    ├── MyApp.Server/
    ├── MyApp.Functions/
    └── MyApp.Worker/
```

### Key Principles

| Layer | Contains | Depends On |
| ------- | ---------- | ------------ |
| **Domain** | Models, Commands, Queries, Validators, Authorizers, Domain Service Interfaces | Nothing (pure) |
| **Infrastructure** | Handlers, Entities, Repositories, External Service Implementations | Domain |
| **Host** | Endpoints, Startup, Configuration | Domain, Infrastructure |

**The domain never references infrastructure.** Handlers know about entities; your domain doesn't. This keeps your business logic portable and testable, regardless of how you choose to persist or host it.

### Terminology

| Term | Purpose | Example |
| ------ | --------- | --------- |
| **Model** | Public domain contract representing response data | `Customer`, `OrderSummary` |
| **Command** | Input contract representing an intent to change state | `CreateCustomerCommand` |
| **Query** | Input contract representing a request for data | `GetCustomerByIdQuery` |
| **Entity** | Persistence representation, shaped by storage needs | `CustomerEntity`, `OrderDocument` |
| **Handler** | Orchestrates a single command or query, lives in infrastructure | `CreateCustomerHandler` |

Models define your domain's public surface area—what consumers see. Entities are implementation details of how data is stored. Handlers bridge the two with explicit mapping, keeping your domain clean and your persistence flexible.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                         │
│  Your handlers, queries, commands - runtime agnostic         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  Cirreum.Conductor                           │
│  Pipeline orchestration with intercepts                      │
│  ├─ Authorization (automatic, role-based)                    │
│  ├─ Validation (FluentValidation integration)               │
│  ├─ Logging, Metrics, Caching                               │
│  └─ Custom intercepts                                        │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┬──────────────┐
         ▼                       ▼              ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Server          │    │ WASM            │    │ Functions       │
│ (ASP.NET Core)  │    │ (Blazor)        │    │ (Azure)         │
│                 │    │                 │    │                 │
│ Result → HTTP   │    │ Result → UI     │    │ Result → JSON   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Core Libraries

### 🚂 [Cirreum.Result](https://github.com/cirreum/Cirreum.Result)

A lightweight, allocation‑free, struct‑based Result monad designed for high‑performance .NET applications.
Provides a complete toolkit for functional, exception‑free control flow with full async support, validation, inspection, and monadic composition.

**Key Features:**

- Struct-based: No heap allocations on the success path.
- Unified Success/Failure Model using Result and Result&lt;T&gt;.
- Full async support
- ValueTask + Task
- Async variants of Map, Then, Ensure, Inspect, failure handlers, etc.
- Validation pipeline with Ensure (sync + async).
- Inspection helpers: Inspect, InspectTry.
- Composable monad API:
- Map, Then, Match, Switch, TryGetValue, TryGetError, and more.
- Ergonomic extension methods for async workflows.
- Zero exceptions for control flow—exceptions are captured as failures.

### 🚂 [Cirreum.Core](https://github.com/cirreum/Cirreum.Core)

#### Conductor

Request/response pipeline with intercept-based architecture. Dispatch commands and queries through a configurable pipeline with automatic authorization, validation, and cross-cutting concerns.

**Key Features:**

- MediatR-style request dispatching with `Result<T>` return types
- Intercept pattern for cross-cutting concerns
- Generic constraint-based registration
- Full async/await support with cancellation

#### 🔐 Cirreum.Authorization

Resource-level authorization with context-base, attribute-based, and role-based access control. FluentValidation-style syntax for declaring authorization rules.

**Key Features:**

- Resource-specific authorization validators
- Runtime-wide policy validators
- Role hierarchy and inheritance
- Automatic integration with Conductor pipeline
- Works with any authentication provider

#### ✅ Cirreum.Validation

FluentValidation integration for request validation with automatic Railway conversion.

**Key Features:**

- Automatic validation through Conductor intercept
- Converts validation failures to `Result.Fail()`
- RFC 7807 ProblemDetails format
- Custom validation rules and async validators

### 🗄️ [Cirreum.Persistence.NoSql](https://github.com/cirreum/Cirreum.Persistence.NoSql)

Document database abstractions with repository and unit of work patterns. Currently supports Azure Cosmos DB with a provider-agnostic interface.

**Key Features:**

- Generic repository pattern with specification support
- Unit of work coordination for transactional consistency
- Azure Cosmos DB implementation
- Optimistic concurrency with ETag support
- Partition key management

### 🗄️ [Cirreum.Persistence.Sql](https://github.com/cirreum/Cirreum.Persistence.Sql)

Lightweight SQL data access built on Dapper. Provides repository abstractions without the overhead of a full ORM.

**Key Features:**

- Generic repository pattern extending Dapper
- SQL Server and SQLite implementations
- Connection management and transaction support
- Bulk operations support
- Query builder helpers

### 📦 [Cirreum.Storage](https://github.com/cirreum/Cirreum.Storage)

Blob storage abstractions with Azure Blob Storage and local file system implementations.

**Key Features:**

- Provider-agnostic storage interface
- Streaming support for large files
- Metadata management
- SAS token generation for Azure

### 📧 [Cirreum.Messaging](https://github.com/cirreum/Cirreum.Messaging)

Email and SMS communication with Twilio (SendGrid) and Azure Communication Services support.

**Key Features:**

- Template-based messaging
- Retry logic with exponential backoff
- Health checks and monitoring
- Azure Key Vault integration for credentials

### 🔑 [Cirreum.Secrets](https://github.com/cirreum/Cirreum.Secrets)

Secret management with Azure Key Vault integration and local development fallbacks.

**Key Features:**

- Unified secret access interface
- Azure Key Vault provider
- Local development configuration
- Automatic credential refresh

### 🎨 [Cirreum.Components.WebAssembly](https://github.com/cirreum/Cirreum.Components.WebAssembly)

Reusable Blazor components with theming support.

**Key Features:**

- TreeView, DataGrid, and form components
- Focus trap and accessibility helpers
- Dark mode support
- Multiple theme variants (Office, Excel, Windows, Aspire)

### 🌐 [Cirreum.Services.Server](https://github.com/cirreum/Cirreum.Services.Server)

ASP.NET Core infrastructure for Railway-to-HTTP conversion and exception handling.

**Key Features:**

- Automatic `Result<T>` to HTTP response conversion
- Global exception handler with ProblemDetails
- RFC 7807 compliant error responses
- Content negotiation support

## Quick Start

### 1. Install Packages

```bash
dotnet add package Cirreum.Runtime.Server
dotnet add package Cirreum.Runtime.Secrets // Typical
dotnet add package Cirreum.Runtime.Authorization // Typical
dotnet add package Cirreum.Runtime.Persistence.Azure // Optional
dotnet add package Cirreum.Runtime.Persistence.SQLite // Optional
dotnet add package Cirreum.Runtime.Persistence.SqlServer // Optional
dotnet add package Cirreum.Runtime.Communications // Optional
dotnet add package Cirreum.Runtime.Storage // Optional
```

### 2. Configure Services

```csharp
// Program.cs
// ******************************************************************************
// Configure the DomainApplication
//
var builder = DomainApplication
    .CreateBuilder(args);


// ******************************************************************************
// Add application secrets
//
builder.AddSecrets();


// ******************************************************************************
// Add Authentication/Authorization
//
builder.AddAuthorization();


// ******************************************************************************
// Add Services to the DomainApplication
//

// Cirreum based service providers
builder
    .AddPersistence()
    .AddStorage()
    .AddMessaging()
    .AddEmailServices()
    .AddSmsServices();

// Your local Services
builder.Services
    .AddScoped<IMyService, MyService>();

// OpenApi (if Development)
if (builder.Environment.IsDevelopment()) {
    builder.Services.AddOpenApi(options => {
        options.AddDocumentTransformer<InfoAndContactTransformer>();
        options.AddSchemaTransformer<PropertyDescriptionTransformer>();
        options.AddDocumentTransformer<SecuritySchemesTransformer>();
    });
}

// *****************************************************************************************************************************
// Build the DomainApplication, referencing your referenced domain library (includes overrloads for multi-library scanning)
//
await using var app = builder.Build<MyDomainTypeInReferenceLib>();


// *****************************************************************************************************************************
// Use the default Middleware
//
app.UseDefaultMiddleware();


// ******************************************************************************
// Map Endpoints
//

// HealthCheck Endpoints
app.MapDefaultHealthChecks();

// Map Feature Endpoints (Application API)
app.MapApiEndpoints("/api/v1", api => {
    // [Static Class] CustomersApi.MapEndpoints(api);
    // [Static Inline] api.MapGet("/customers", Customers.GetAll);
    // [Inline]
    api.MapGet("/customers", static async (IDispatcher dispatcher, CancellationToken token) =>
      dispatcher.Dispatch(new GetAllCustomers(), token) 
    );
});

// OpenApi
if (app.Environment.IsDevelopment()) {
    app.MapOpenApi()
        .CacheOutput();
    app.MapScalarApiReference(options => {
        options.DefaultFonts = false;
        options.Layout = ScalarLayout.Modern;
        options.OperationSorter = OperationSorter.Method;
        options.AddPreferredSecuritySchemes("Bearer");
        options.WithDefaultHttpClient(ScalarTarget.CSharp, ScalarClient.Curl);
    });
}

// redirect root requests to the configured endpoint
app.UseLandingPage();


// *****************************************************************************************************************************
// Run the DomainApplication...
//
await app.RunAsync();

```

### 3. Define a Request in your Domain library

```csharp
public record GetAllCustomers() : IAuthorizableRequest<IReadOnlyList<Customer>> {
    // Validation and Authorization handled automatically by Conductor
}
// optionally if you want auditing and query caching
public record GetAllCustomers() : 
    IAuthorizableRequest<IReadOnlyList<Customer>>, 
    IAuditableRequest<IReadOnlyList<Customer>>, 
    ICacheableQuery<IReadOnlyList<Customer>> {
    // Caching, Auditing, Validation and Authorization handled automatically by Conductor
}
// -OR- consolidated Domain interface
public record GetAllCustomers() : IDomainCacheableQuery<IReadOnlyList<Customer>> {
    // Caching, Auditing, Validation and Authorization handled automatically by Conductor
}
```

### 4. Implement the Handler in your Infrastructure or Server library

```csharp
public class GetAllCustomersHandler(
    IRepository<Customer> repository
) : IRequestHandler<GetAllCustomers, IReadOnlyList<Customer>> {
    
    public async Task<Result<IReadOnlyList<Customer>>> HandleAsync(
        GetAllCustomers request,
        CancellationToken cancellationToken) =>
            await _repository.GetAll(cancellationToken);
    }
}
```

That's it! Authorization is enforced automatically, validation runs through the pipeline, and the `Result<T>` is converted to the appropriate HTTP response (200 OK, 403 Forbidden, 422 Unprocessable Entity, etc.).

## Railway-Oriented Programming

Cirreum embraces Railway-Oriented Programming to eliminate exception-based control flow:

```csharp
// Traditional approach
public async Task<ResourceDto> GetResource(string id) {
    try {
        var resource = await _repository.GetById(id);
        if (resource == null) {
            throw new NotFoundException("Resource not found");
        }
        if (!await _authService.CanAccess(resource)) {
            throw new ForbiddenException("Access denied");
        }
        return MapToDto(resource);
    } catch (Exception ex) {
        _logger.LogError(ex, "Failed to get resource");
        throw;
    }
}

// Cirreum approach (no-throw)
public async Task<Result<Resource>> Handle(GetResourceQuery query, CancellationToken ct) {
    var entity = await _repository.GetById(query.Id, ct);

    // Implicit-cast for simple case
    return entity.Map(); // your extension method maps to 'Resource' model/dto

    // Or, you control the exception type
    return entity is not null
        ? Result<Resource>.Success(entity.Map())
        : Result<Resource>.Fail(new NotFoundException("Resource not found"));

    // Authorization handled automatically by intercept
    // Logging handled by intercept
    // HTTP conversion handled by filter
}
```

### Result Composition

Chain operations with `Map`, `Then`, and `Where`:

```csharp
public async Task<Result<InvoiceDto>> Handle(CreateInvoiceCommand cmd, CancellationToken ct) {
    return await GetCustomer(cmd.CustomerId)
        .Then(customer => ValidateCredit(customer))
        .Map(customer => CreateInvoice(customer, cmd.Items))
        .Then(invoice => SaveInvoice(invoice))
        .Map(invoice => MapToDto(invoice));
}
```

See [Cirreum.Result](https://github.com/cirreum/Cirreum.Result) and [Cirreum.Exceptions](https://github.com/cirreum/Cirreum.Exceptions) for more details.

## Multi-Runtime Support

### Server (ASP.NET Core)

```csharp
[HttpGet("{id}")]
public async Task<Result<ResourceDto>> Get(string id) {
    return await _dispatcher.Dispatch(new GetResourceQuery(id));
}
```

### WASM (Blazor)

```csharp
private async Task LoadResource() {
    var result = await Dispatcher.Dispatch(new GetResourceQuery(ResourceId));
    
    result.Switch(
        onSuccess: value => {
            Resource = value;
            StateHasChanged();
        },
        onFailure: error => ErrorMessage = error.Message
    );
}
```

### Functions (Azure)

```csharp
[Function("GetResource")]
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequest req) {
    
    var result = await _dispatcher.Dispatch(new GetResourceQuery(req.Query["id"]));
    
    return result.Match(
        onSuccess: value => new OkObjectResult(value),
        onFailure: error => new ObjectResult(error) { StatusCode = 500 }
    );
}
```

## Authorization Example

Define authorization rules using FluentValidation-style syntax:

```csharp
public class DeleteResourceValidator : AuthorizationValidatorBase<DeleteResourceCommand> {
    public DeleteResourceValidator() {
        RuleFor(context => context)
            .RequireRole<AdminRole>()
            .WithMessage("Only administrators can delete resources");
            
        RuleFor(context => context.Resource.Id)
            .MustAsync(async (id, ctx, ct) => {
                var resource = await GetResource(id);
                return resource.OwnerId == ctx.User.Id;
            })
            .WithMessage("You can only delete your own resources");
    }
}
```

The authorization runs automatically through the Conductor pipeline—no need to call it explicitly in your handler.

## HTTP Response Examples

### Success (200 OK)

```json
{
  "id": "123",
  "name": "Resource Name",
  "createdAt": "2025-11-13T10:30:00Z"
}
```

### Validation Error (422 Unprocessable Entity)

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.21",
  "title": "Unprocessable Entity",
  "status": 422,
  "detail": "Validation failed",
  "instance": "/api/resources",
  "failures": [
    {
      "propertyName": "Name",
      "errorMessage": "Name is required",
      "errorCode": "NotEmptyValidator",
      "severity": "Error"
    }
  ]
}
```

### Authorization Error (403 Forbidden)

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.4",
  "title": "Forbidden",
  "status": 403,
  "detail": "User 'john@example.com' lacks permission 'resource:delete'",
  "instance": "/api/resources/123"
}
```

## Why Cirreum?

| Feature | Traditional ASP.NET | MediatR | Cirreum |
| --------- | --------------------- | --------- | --------- |
| Railway-Oriented | ❌ | ❌ | ✅ |
| Runtime-Agnostic | ❌ | ✅ | ✅ |
| Auto Authorization | ❌ | ❌ | ✅ |
| Auto HTTP Conversion | ❌ | ❌ | ✅ |
| Consistent Error Format | ❌ | ❌ | ✅ |
| Built-in Validation | ❌ | ❌ | ✅ |

## Contributing

Cirreum is open source and welcomes contributions! Each repository has its own contribution guidelines, but here are some general principles:

- Follow existing code style and conventions
- Include unit tests for new features
- Update documentation for user-facing changes
- Keep PRs focused on a single concern

## Roadmap

- [ ] Fix all the documentation and readme files
- [ ] More messaging providers (RabbitMQ, AWS)
- [ ] More storage providers (AWS, others)
- [ ] More persistence providers (SQL based servers, EFCore)
- [ ] Add MCP-Native Endpoints Feature
- [ ] Performance benchmarks
- [ ] Complete documentation site

## License

Cirreum is licensed under the MIT License. See individual repositories for details.

## Support

- 💬 **Discussions**: [GitHub Discussions](https://github.com/orgs/cirreum/discussions)
- 🐛 **Issues**: Report issues in the relevant repository
- 📧 **Contact**: [Contact Information]

**Built with ❤️ for the .NET community**
