
<div>
<p align="middle">
  <img src="https://raw.githubusercontent.com/cirreum/.github/main/profile/cirreum-icon-transparent-002.png" width="180" alt="Cirreum Logo" />
  <br/>
  <span style="font-size: 32px;font-weight: bold;">Cirreum</span>
</p>
</div>

<h4>Layered simplicity for modern .NET</h4>

## **A modern, opinionated foundation framework built on Domain-first principles—define your models and operations once, consume them across multiple applications—with Railway-Oriented Programming ensuring clean, composable control flow throughout.**

Cirreum provides a comprehensive set of libraries that work seamlessly together to enable rapid development of enterprise-grade applications across Server (ASP.NET Core), Client (Blazor WebAssembly), and Serverless (Azure Functions) environments.

## Philosophy

Cirreum embraces **Railway-Oriented Programming** to eliminate exception-based control flow while maintaining clean, composable code. Write your business logic once, and run it anywhere—with authorization, validation, and error handling automatically applied through a powerful pipeline architecture.

```csharp
// Write once
public class GetResourceHandler : IOperationHandler<GetResourceQuery, ResourceDto> {
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

## Library Layout

Cirreum is organized into consistent dependency layers and split between two tracks. The layer tells you what a package depends on; the track tells you how you consume it.

### Layers (bottom to top)

| Layer | Purpose | Example |
| --------- | --------- | --------- |
| **Base** | Zero dependencies, usable anywhere | `Cirreum.Result` |
| **Common** | Framework-neutral, reusable across hosts | `Cirreum.Validation` |
| **Core** | The framework spine — once you reference this, you're committed to Cirreum | `Cirreum.Core` |
| **Services** | Host-specific glue (ASP.NET Core, WASM, Functions) | `Cirreum.Services.Server` |
| **Runtime** | App-facing entry points (`AddX()` / `MapX()` extensions) | `Cirreum.Runtime.Server` |

### Two tracks

**Main track** — the framework spine. Apps reference `Cirreum.Core` + a `Cirreum.Services.{host}` + a `Cirreum.Runtime.{host}` directly. Linear dependency chain. Not pluggable.

**Provider tracks** — pluggable cross-cutting services. Each provider family (Authorization, Identity, Persistence, Communications, Messaging, Storage, Secrets) follows the same shape:

```
Cirreum.{Family}Provider     # contracts & registration core      (Core/)
Cirreum.{Family}.{Impl}      # concrete implementation             (Infrastructure/)
Cirreum.Runtime.{Family}     # app-facing umbrella; pulls Impl transitively  (Runtime Extensions/)
```

App code installs only the `Cirreum.Runtime.{Family}` umbrella. Implementations flow in transitively, and you can swap providers without touching app code.

The full repo catalog below is organized by folder (each folder is a layer).

## Library Catalog

Every published Cirreum repo, organized by folder. Each folder is a layer; within a layer, a package is either part of the **main track** (the linear framework spine) or a **provider track** (a pluggable service family).

### Base — zero dependencies

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Result](https://github.com/cirreum/Cirreum.Result) | main | Struct-based, allocation-free `Result` / `Result<T>` monad with full async support and monadic composition (Map, Then, Ensure, Match, Switch). |
| [Cirreum.Exceptions](https://github.com/cirreum/Cirreum.Exceptions) | main | Typed exception hierarchy (`NotFoundException`, `ForbiddenException`, `ValidationException`, …) designed to be captured as `Result<T>.Fail(...)`. RFC 7807 ProblemDetails mapping. |

### Common — framework-neutral, host-agnostic abstractions

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Communications.Email](https://github.com/cirreum/Cirreum.Communications.Email) | Communications | Email sender contracts and templates. |
| [Cirreum.Communications.Sms](https://github.com/cirreum/Cirreum.Communications.Sms) | Communications | SMS sender contracts and templates. |
| [Cirreum.Cors](https://github.com/cirreum/Cirreum.Cors) | main | CORS configuration helpers. |
| [Cirreum.ExpressionBuilder](https://github.com/cirreum/Cirreum.ExpressionBuilder) | main | Dynamic LINQ expression-tree utilities. |
| [Cirreum.Logging.Deferred](https://github.com/cirreum/Cirreum.Logging.Deferred) | main | Deferred / batched logging primitives. |
| [Cirreum.Messaging](https://github.com/cirreum/Cirreum.Messaging) | Messaging | Message queue / pub-sub abstractions. |
| [Cirreum.Persistence.NoSql](https://github.com/cirreum/Cirreum.Persistence.NoSql) | Persistence | Document-database repository + unit-of-work abstractions. |
| [Cirreum.Persistence.Sql](https://github.com/cirreum/Cirreum.Persistence.Sql) | Persistence | SQL repository abstractions built on Dapper. |
| [Cirreum.Providers](https://github.com/cirreum/Cirreum.Providers) | infra | Provider-pattern plumbing shared across all provider tracks. |
| [Cirreum.Startup](https://github.com/cirreum/Cirreum.Startup) | main | `DomainApplication` host-bootstrap helpers. |
| [Cirreum.Storage](https://github.com/cirreum/Cirreum.Storage) | Storage | Blob storage abstractions. |
| [Cirreum.Storage.Browser](https://github.com/cirreum/Cirreum.Storage.Browser) | Storage | Browser-side storage helpers (LocalStorage, SessionStorage, IndexedDB). |

### Core — the framework spine

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Core](https://github.com/cirreum/Cirreum.Core) | main | Conductor pipeline (`IDispatcher`, intercepts), validation, the framework spine. |
| [Cirreum.Components.WebAssembly](https://github.com/cirreum/Cirreum.Components.WebAssembly) | main | Reusable Blazor components (TreeView, DataGrid, forms) with multi-theme support. |
| [Cirreum.AuthorizationProvider](https://github.com/cirreum/Cirreum.AuthorizationProvider) | Authorization (core) | Provider contracts and registration for authorization. |
| [Cirreum.IdentityProvider](https://github.com/cirreum/Cirreum.IdentityProvider) | Identity (core) | Provisioning contracts and instance-keying. |
| [Cirreum.SecretsProvider](https://github.com/cirreum/Cirreum.SecretsProvider) | Secrets (core) | Secret-store contracts and registration. |
| [Cirreum.ServiceProvider](https://github.com/cirreum/Cirreum.ServiceProvider) | infra | Runtime service-registration plumbing used by other provider tracks. |

### Infrastructure — provider implementations and host services

**Authorization providers**

| Package | Description |
| --------- | ------------- |
| [Cirreum.Authorization.ApiKey](https://github.com/cirreum/Cirreum.Authorization.ApiKey) | API-key bearer authorization. |
| [Cirreum.Authorization.Entra](https://github.com/cirreum/Cirreum.Authorization.Entra) | Microsoft Entra ID (workforce / employee tenant). |
| [Cirreum.Authorization.External](https://github.com/cirreum/Cirreum.Authorization.External) | External JWT bearer (arbitrary OIDC issuer). |
| [Cirreum.Authorization.Oidc](https://github.com/cirreum/Cirreum.Authorization.Oidc) | Generic OIDC bearer. |
| [Cirreum.Authorization.SignedRequest](https://github.com/cirreum/Cirreum.Authorization.SignedRequest) | HMAC-signed-request authorization (server side). |
| [Cirreum.Authorization.SignedRequest.Client](https://github.com/cirreum/Cirreum.Authorization.SignedRequest.Client) | HMAC-signed-request client SDK. |

**Identity providers**

| Package | Description |
| --------- | ------------- |
| [Cirreum.Identity.EntraExternalId](https://github.com/cirreum/Cirreum.Identity.EntraExternalId) | Microsoft Entra External ID provisioning. |
| [Cirreum.Identity.Oidc](https://github.com/cirreum/Cirreum.Identity.Oidc) | Generic OIDC provisioning (Auth0, Okta, Descope, Keycloak, …). |

**Persistence implementations**

| Package | Description |
| --------- | ------------- |
| [Cirreum.Persistence.Azure](https://github.com/cirreum/Cirreum.Persistence.Azure) | Azure Cosmos DB. |
| [Cirreum.Persistence.SQLite](https://github.com/cirreum/Cirreum.Persistence.SQLite) | SQLite. |
| [Cirreum.Persistence.SqlServer](https://github.com/cirreum/Cirreum.Persistence.SqlServer) | SQL Server (Dapper). |

**Communications implementations**

| Package | Description |
| --------- | ------------- |
| [Cirreum.Communications.Email.Azure](https://github.com/cirreum/Cirreum.Communications.Email.Azure) | Azure Communication Services email. |
| [Cirreum.Communications.Email.SendGrid](https://github.com/cirreum/Cirreum.Communications.Email.SendGrid) | Twilio SendGrid email. |
| [Cirreum.Communications.Sms.Azure](https://github.com/cirreum/Cirreum.Communications.Sms.Azure) | Azure Communication Services SMS. |
| [Cirreum.Communications.Sms.Twilio](https://github.com/cirreum/Cirreum.Communications.Sms.Twilio) | Twilio SMS. |

**Other implementations**

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Storage.Azure](https://github.com/cirreum/Cirreum.Storage.Azure) | Storage | Azure Blob Storage. |
| [Cirreum.Messaging.Azure](https://github.com/cirreum/Cirreum.Messaging.Azure) | Messaging | Azure Service Bus. |
| [Cirreum.Secrets.Azure](https://github.com/cirreum/Cirreum.Secrets.Azure) | Secrets | Azure Key Vault. |
| [Cirreum.Graph.Provider](https://github.com/cirreum/Cirreum.Graph.Provider) | infra | Microsoft Graph SDK provider. |
| [Cirreum.QueryCache.Distributed](https://github.com/cirreum/Cirreum.QueryCache.Distributed) | infra | Distributed cache backing for `Conductor.ICacheableOperation`. |
| [Cirreum.QueryCache.Hybrid](https://github.com/cirreum/Cirreum.QueryCache.Hybrid) | infra | Hybrid (in-memory + distributed) cache backing. |

**Host services (main track)**

| Package | Description |
| --------- | ------------- |
| [Cirreum.Services.Server](https://github.com/cirreum/Cirreum.Services.Server) | ASP.NET Core host services — Result-to-HTTP, ProblemDetails, etc. |
| [Cirreum.Services.Wasm](https://github.com/cirreum/Cirreum.Services.Wasm) | Blazor WASM host services. |
| [Cirreum.Services.Serverless](https://github.com/cirreum/Cirreum.Services.Serverless) | Azure Functions host services. |

### Runtime — main-track app entry points + provider runtime cores

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Runtime.Server](https://github.com/cirreum/Cirreum.Runtime.Server) | main | App package for ASP.NET Core hosts. |
| [Cirreum.Runtime.Wasm](https://github.com/cirreum/Cirreum.Runtime.Wasm) | main | App package for Blazor WASM hosts. |
| [Cirreum.Runtime.Serverless](https://github.com/cirreum/Cirreum.Runtime.Serverless) | main | App package for Azure Functions hosts. |
| [Cirreum.Runtime.AuthorizationProvider](https://github.com/cirreum/Cirreum.Runtime.AuthorizationProvider) | Authorization (runtime core) | Runtime-side core for the authorization provider track. |
| [Cirreum.Runtime.IdentityProvider](https://github.com/cirreum/Cirreum.Runtime.IdentityProvider) | Identity (runtime core) | Runtime-side core for the identity provider track. |
| [Cirreum.Runtime.SecretsProvider](https://github.com/cirreum/Cirreum.Runtime.SecretsProvider) | Secrets (runtime core) | Runtime-side core for the secrets provider track. |
| [Cirreum.Runtime.ServiceProvider](https://github.com/cirreum/Cirreum.Runtime.ServiceProvider) | infra | Runtime-side provider-pattern plumbing. |

### Runtime Extensions — provider-track app umbrellas (what your app actually installs)

| Package | Track | Description |
| --------- | ------- | ------------- |
| [Cirreum.Runtime.Authorization](https://github.com/cirreum/Cirreum.Runtime.Authorization) | Authorization | `builder.AddAuthorization(...)` umbrella; pulls authorization providers transitively. |
| [Cirreum.Runtime.Communications](https://github.com/cirreum/Cirreum.Runtime.Communications) | Communications | Email + SMS umbrella. |
| [Cirreum.Runtime.Identity](https://github.com/cirreum/Cirreum.Runtime.Identity) | Identity | `builder.AddIdentity(p => p.AddProvisioner<T>(key))` cross-protocol umbrella. |
| [Cirreum.Runtime.Identity.Oidc](https://github.com/cirreum/Cirreum.Runtime.Identity.Oidc) | Identity | Per-protocol entry: generic OIDC. |
| [Cirreum.Runtime.Identity.EntraExternalId](https://github.com/cirreum/Cirreum.Runtime.Identity.EntraExternalId) | Identity | Per-protocol entry: Entra External ID. |
| [Cirreum.Runtime.Messaging](https://github.com/cirreum/Cirreum.Runtime.Messaging) | Messaging | Messaging umbrella. |
| [Cirreum.Runtime.Persistence](https://github.com/cirreum/Cirreum.Runtime.Persistence) | Persistence | Persistence umbrella (provider-agnostic). |
| [Cirreum.Runtime.Persistence.Azure](https://github.com/cirreum/Cirreum.Runtime.Persistence.Azure) | Persistence | Cosmos DB. |
| [Cirreum.Runtime.Persistence.SQLite](https://github.com/cirreum/Cirreum.Runtime.Persistence.SQLite) | Persistence | SQLite. |
| [Cirreum.Runtime.Persistence.SqlServer](https://github.com/cirreum/Cirreum.Runtime.Persistence.SqlServer) | Persistence | SQL Server. |
| [Cirreum.Runtime.Secrets](https://github.com/cirreum/Cirreum.Runtime.Secrets) | Secrets | Azure Key Vault umbrella. |
| [Cirreum.Runtime.Storage](https://github.com/cirreum/Cirreum.Runtime.Storage) | Storage | Azure Blob Storage umbrella. |
| [Cirreum.Runtime.Wasm.Msal](https://github.com/cirreum/Cirreum.Runtime.Wasm.Msal) | Identity (WASM) | WASM client identity flows via MSAL (Entra workforce / B2C). |
| [Cirreum.Runtime.Wasm.Oidc](https://github.com/cirreum/Cirreum.Runtime.Wasm.Oidc) | Identity (WASM) | WASM client identity flows via generic OIDC. |

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
    builder.Services.AddOpenApi();
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
    app.MapOpenApi();
    app.MapScalarApiReference();
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
public record GetAllCustomers() : IAuthorizableOperation<IReadOnlyList<Customer>> {
    // Validation and Authorization handled automatically by Conductor
}
// optionally if you want auditing and query caching
public record GetAllCustomers() : 
    IAuthorizableOperation<IReadOnlyList<Customer>>, 
    ICacheableOperation<IReadOnlyList<Customer>> {
    // Caching, Validation and Authorization handled automatically by Conductor
}
// -OR- consolidated interface
public record GetAllCustomers() : IOwnerCacheableLookupOperation<IReadOnlyList<Customer>> {
    // Caching, Validation and Authorization handled automatically by Conductor
    string? OwnerId { get; set; }
}
```

### 4. Implement the Handler in your Infrastructure or Server library

```csharp
public class GetAllCustomersHandler(
    IRepository<Customer> repository
) : IOperationHandler<GetAllCustomers, IReadOnlyList<Customer>> {
    
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
public class DeleteResourceValidator : AuthorizerBase<DeleteResourceCommand> {
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

- [ ] More messaging providers (RabbitMQ, AWS SNS/SQS)
- [ ] More storage providers (AWS S3)
- [ ] EF Core persistence provider
- [ ] A complete documentation site and developer guide

## License

Cirreum is licensed under the MIT License. See individual repositories for details.

## Support

- 💬 **Discussions**: [GitHub Discussions](https://github.com/orgs/cirreum/discussions)
- 🐛 **Issues**: Report issues in the relevant repository

**Built with ❤️ for the .NET community**
