<p align="center">
  <img src="https://raw.githubusercontent.com/cirreum/.github/main/profile/cirreum-icon-transparent-002.png" width="180" alt="Cirreum" />
</p>

<h1 align="center">Cirreum</h1>
<h4 align="center">Layered simplicity for modern .NET</h4>

---

## Build .NET applications once — run them across API, Blazor, and serverless hosts

Cirreum is an opinionated application framework for modern .NET systems.

It helps you define your domain models, operations, validation, authorization, and error-handling behavior once, then reuse that logic across:

- ASP.NET Core APIs
- Blazor WebAssembly applications
- Azure Functions
- background jobs and worker processes

Cirreum is designed for applications where the same business logic needs to run consistently across multiple hosts without duplicating infrastructure code.

---

## Why Cirreum exists

Modern .NET applications often spread the same logic across controllers, services, UI clients, background workers, and serverless functions.

That usually leads to duplicated authorization checks, inconsistent validation, exception-heavy control flow, and different error handling in each runtime.

Cirreum solves this by putting your application operations behind a shared execution pipeline.

You write the operation once:

```csharp
public record GetCustomer(Guid Id) : IAuthorizableOperation<Customer>;

public sealed class GetCustomerHandler(
    IRepository<CustomerEntity> repository
) : IOperationHandler<GetCustomer, Customer> {

    public async Task<Result<Customer>> HandleAsync(
        GetCustomer request,
        CancellationToken cancellationToken) {

        var entity = await repository.GetByIdAsync(request.Id, cancellationToken);

        return entity is not null
            ? entity.MapToCustomer()
            : Result<Customer>.Fail(new NotFoundException("Customer not found"));
    }
}
```

Then dispatch it from anywhere:

```csharp
var result = await dispatcher.Dispatch(new GetCustomer(customerId), cancellationToken);
```

The same operation can be used from an API endpoint, a Blazor component, an Azure Function, or a background job.

Authorization, validation, logging, caching, and error handling are applied by the pipeline instead of being repeated in every handler or endpoint.

---

## What Cirreum gives you

- **A shared operation pipeline** for commands, queries, and application workflows
- **Railway-Oriented Programming** with `Result` and `Result<T>`
- **Automatic validation and authorization** through pipeline intercepts
- **Consistent HTTP response mapping** for ASP.NET Core APIs
- **Runtime-specific integration packages** for Server, WASM, and Serverless hosts
- **Provider-based infrastructure** for authorization, identity, persistence, storage, messaging, secrets, email, and SMS
- **Clean domain separation** between public models, application operations, handlers, and persistence entities

---

## Who Cirreum is for

Cirreum is built for developers and teams creating structured .NET applications with more than one runtime or host.

It is especially useful when you are building:

- APIs with shared domain operations
- Blazor WebAssembly applications backed by ASP.NET Core
- systems with multiple identity providers
- applications with app-managed roles and authorization rules
- modular platforms made from many NuGet packages
- serverless or background-processing workflows that reuse the same business logic as the API

Cirreum is not trying to replace ASP.NET Core, Blazor, Azure Functions, or the .NET host model.

It sits on top of them and gives your application a consistent domain-first execution model.

---

## Core ideas

### One domain model

Your domain project defines the public shape of your application:

- models
- commands
- queries
- validators
- authorizers
- operation contracts

The domain does not depend on infrastructure.

### One execution pipeline

Operations are dispatched through Cirreum Conductor.

The pipeline can apply:

- validation
- authorization
- logging
- metrics
- caching
- error handling
- custom intercepts

### Multiple runtimes

The same operation can be consumed from different hosts:

- ASP.NET Core maps `Result<T>` to HTTP responses
- Blazor WASM can dispatch operations in-process
- Azure Functions can reuse the same handlers and result semantics
- background workers can execute the same domain workflows

### Explicit success and failure

Cirreum uses Railway-Oriented Programming to make success and failure part of the type system.

Instead of throwing exceptions for expected application outcomes, handlers return `Result` or `Result<T>`.

```csharp
return customer is not null
    ? Result<Customer>.Success(customer)
    : Result<Customer>.Fail(new NotFoundException("Customer not found"));
```

---

## Quick start

Install the server runtime package:

```bash
dotnet add package Cirreum.Runtime.Server
```

Create a Cirreum application:

```csharp
var builder = DomainApplication.CreateBuilder(args);

builder.AddAuthorization();

await using var app = builder.Build<MyDomainMarker>();

app.UseDefaultMiddleware();

app.MapApiEndpoints("/api/v1", api => {
    api.MapGet("/customers/{id}", static async (
        Guid id,
        IDispatcher dispatcher,
        CancellationToken cancellationToken) =>
            await dispatcher.Dispatch(new GetCustomer(id), cancellationToken));
});

await app.RunAsync();
```

Define an operation:

```csharp
public record GetCustomer(Guid Id) : IAuthorizableOperation<Customer>;
```

Implement a handler:

```csharp
public sealed class GetCustomerHandler(
    IRepository<CustomerEntity> repository
) : IOperationHandler<GetCustomer, Customer> {

    public async Task<Result<Customer>> HandleAsync(
        GetCustomer request,
        CancellationToken cancellationToken) {

        var entity = await repository.GetByIdAsync(request.Id, cancellationToken);

        return entity is not null
            ? entity.MapToCustomer()
            : Result<Customer>.Fail(new NotFoundException("Customer not found"));
    }
}
```

That operation can now flow through validation, authorization, result handling, and HTTP response mapping without duplicating that logic in the endpoint.

---

## Example: Result-to-HTTP mapping

A handler returns a domain result:

```csharp
return Result<Customer>.Fail(new NotFoundException("Customer not found"));
```

The server runtime maps it to the correct HTTP response:

```http
404 Not Found
```

Validation failures can become:

```http
422 Unprocessable Entity
```

Authorization failures can become:

```http
403 Forbidden
```

Successful results become:

```http
200 OK
```

The handler does not need to know it is running behind HTTP.

---

## Package families

Cirreum is split into small packages so applications can take only what they need.

### Main runtime packages

| Package | Purpose |
| ------- | ------- |
| `Cirreum.Core` | Conductor pipeline, dispatcher, operation contracts, intercepts |
| `Cirreum.Runtime.Server` | ASP.NET Core host integration |
| `Cirreum.Runtime.Wasm` | Blazor WebAssembly integration |
| `Cirreum.Runtime.Serverless` | Azure Functions integration |

### Common provider families

| Family | Purpose |
| ------ | ------- |
| Authorization | OIDC, Entra, API key, signed request, external JWT |
| Identity | user provisioning and identity-provider integration |
| Persistence | Cosmos DB, SQL Server, SQLite, provider abstractions |
| Storage | blob and browser storage abstractions |
| Messaging | queues and pub/sub abstractions |
| Communications | email and SMS abstractions |
| Secrets | secret-provider integration |

Applications typically reference the runtime package for the host they are building, plus the provider packages they need.

For the full repo-by-repo catalog, see [Full Library Catalog](#full-library-catalog) below.

---

## Philosophy

Cirreum is built around a few principles:

- business logic should not depend on the host
- expected failures should be explicit, not exception-driven
- authorization and validation should be centralized
- infrastructure should be replaceable
- applications should be composed from small, focused packages
- defaults should be useful, but not restrictive

The goal is not to hide .NET.

The goal is to give large .NET applications a consistent foundation.

---

## Architecture Overview

```
              ┌─────────────────────────────────────────────────┐
              │              Application Layer                   │
              │  Your handlers, queries, commands                │
              │  (runtime-agnostic, defined in your domain)      │
              └────────────────────┬────────────────────────────┘
                                   │
                                   ▼
              ┌─────────────────────────────────────────────────┐
              │             Cirreum.Conductor                    │
              │  Pipeline orchestration with intercepts          │
              │  ├─ Authorization · Validation                   │
              │  ├─ Logging · Metrics · Caching                  │
              │  └─ Custom intercepts                            │
              └────────────────────┬────────────────────────────┘
                                   │
                ┌──────────────────┼──────────────────┐
                ▼                  ▼                  ▼
          ┌──────────┐       ┌──────────┐       ┌──────────┐
          │  Server  │       │Functions │       │ ACA Jobs*│
          │  ASP.NET │       │  (Azure) │       │  (Azure) │
          │   Core   │       │          │       │          │
          │          │       │          │       │          │
          │  Result  │       │  Result  │       │  Result  │
          │  → HTTP  │       │  → JSON  │       │  → side  │
          │          │       │          │       │   effect │
          └─────┬────┘       └──────────┘       └──────────┘
                │
                │ HTTP / JSON
                ▼
   ┌────────────────────── Clients ──────────────────────────────┐
   │                                                              │
   │   ┌──────────────┐     ┌─────────────────────────────┐      │
   │   │ Blazor WASM**│     │ React · Angular · Vue       │      │
   │   │              │     │ Mobile · any HTTP client    │      │
   │   │ (Cirreum-    │     │ (no Cirreum reference)      │      │
   │   │  aware)      │     │                             │      │
   │   └──────────────┘     └─────────────────────────────┘      │
   └──────────────────────────────────────────────────────────────┘

    * ACA Jobs uses Cirreum.Runtime.Server (no dedicated runtime package)
   ** Blazor WASM also runs Cirreum.Conductor in-process for client-side
      dispatch — same handlers as the server, hosted in the browser
```

---

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

---

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

---

## Full Library Catalog

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

---

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
public async Task<Result<Resource>> HandleAsync(GetResourceQuery query, CancellationToken ct) {
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
public async Task<Result<InvoiceDto>> HandleAsync(CreateInvoiceCommand cmd, CancellationToken ct) {
    return await GetCustomer(cmd.CustomerId)
        .Then(customer => ValidateCredit(customer))
        .Map(customer => CreateInvoice(customer, cmd.Items))
        .Then(invoice => SaveInvoice(invoice))
        .Map(invoice => MapToDto(invoice));
}
```

See [Cirreum.Result](https://github.com/cirreum/Cirreum.Result) and [Cirreum.Exceptions](https://github.com/cirreum/Cirreum.Exceptions) for more details.

---

## Authorization Example

Define authorization rules using FluentValidation-style syntax:

```csharp
public class DeleteResourceAuthorizer : AuthorizerBase<DeleteResourceCommand> {
    public DeleteResourceAuthorizer() {
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

The authorization runs automatically through the Conductor pipeline — no need to call it explicitly in your handler.

---

## Multi-Runtime Support

The same operation can be dispatched from any host. The handler doesn't change; only the host wiring around it does.

### Server (ASP.NET Core)

```csharp
api.MapGet("/customers/{id}", static async (
    Guid id,
    IDispatcher dispatcher,
    CancellationToken ct) =>
        await dispatcher.Dispatch(new GetCustomer(id), ct));
```

### WASM (Blazor)

```csharp
private async Task LoadCustomer() {
    var result = await Dispatcher.Dispatch(new GetCustomer(CustomerId));

    result.Switch(
        onSuccess: value => {
            Customer = value;
            StateHasChanged();
        },
        onFailure: error => ErrorMessage = error.Message
    );
}
```

### Functions (Azure)

```csharp
[Function("GetCustomer")]
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequest req) {

    var result = await _dispatcher.Dispatch(new GetCustomer(Guid.Parse(req.Query["id"])));

    return result.Match(
        onSuccess: value => new OkObjectResult(value),
        onFailure: error => new ObjectResult(error) { StatusCode = 500 }
    );
}
```

---

## Roadmap

- [ ] More messaging providers (RabbitMQ, AWS SNS/SQS)
- [ ] More storage providers (AWS S3)
- [ ] EF Core persistence provider
- [ ] A complete documentation site and developer guide

---

## Contributing

Cirreum is open source and welcomes contributions. Each repository has its own contribution guidelines, but here are some general principles:

- Follow existing code style and conventions
- Include unit tests for new features
- Update documentation for user-facing changes
- Keep PRs focused on a single concern

---

## License

Cirreum is licensed under the MIT License. See individual repositories for details.

---

## Support

- 💬 **Discussions**: [GitHub Discussions](https://github.com/orgs/cirreum/discussions)
- 🐛 **Issues**: Report issues in the relevant repository

**Built with ❤️ for the .NET community**
