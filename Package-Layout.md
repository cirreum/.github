# Cirreum Package Architecture

This document describes the layered architecture of Cirreum packages. Each layer may depend on the layers below it; dependencies never flow the other way.

```
Runtime Extensions → Runtime → Infrastructure → Core → Common → Base
```

> **Note:** Not every package references every layer beneath it. A Core package, for example, typically references Base and Common but not every Common package.

Packages also belong to one of two **tracks**:

- **Main track** — the linear framework spine. App code references a few layers directly (the foundation, a host's services, and that host's runtime).
- **Provider track** — pluggable cross-cutting services (Authentication, Identity, Persistence, Communications, Messaging, Storage, Secrets). App code references only the app-facing umbrella; implementations flow in transitively.

---

## Base

Dependency-free — usable by any application or consumer, not just Cirreum.

- `Cirreum.Result` — `Result`, `Result<T>`, and `Optional<T>` monads (railway-oriented programming)
- `Cirreum.Exceptions` — predefined application exceptions, designed to be captured as `Result<T>.Fail(...)`
- `Cirreum.Kernel` — foundational cross-host abstractions, contracts, value types, and the framework bootstrap surface

---

## Common

Framework-neutral abstractions — dependency-light, usable in any ASP.NET host with or without the rest of Cirreum. Unless noted, assumed Server/Serverless.

- `Cirreum.Common` — cross-host primitives: Conductor (CQRS), caching, state, presence, remote services, file system, and the authorization vocabulary
- `Cirreum.Cors`
- `Cirreum.ExpressionBuilder` *(env/host agnostic)*
- `Cirreum.Logging.Deferred`
- `Cirreum.Startup` *(env/host agnostic)*
- `Cirreum.Providers` — provider-pattern plumbing shared across provider tracks
- `Cirreum.Messaging` — message queue / pub-sub abstractions
- `Cirreum.Messaging.Distributed` — distributed message-envelope abstractions
- `Cirreum.Persistence.NoSql`
- `Cirreum.Persistence.Sql`
- `Cirreum.Storage`
- `Cirreum.Storage.Browser` *(Blazor WASM / browser)*
- `Cirreum.Communications.Email`
- `Cirreum.Communications.Sms`

---

## Core

The framework spine plus the provider abstraction cores. Cross-host (browser, server, serverless).

### Main track
- `Cirreum.Shared` — cross-host spine implementations: Conductor dispatcher/publisher, caching, state, presence, remote services, and the authorization-pillar implementations
- `Cirreum.Components.WebAssembly` — Blazor WASM component library

### Provider track (abstraction cores)
- `Cirreum.AuthenticationProvider` — Authentication track contracts and registration
- `Cirreum.IdentityProvider` — identity provisioning contracts and instance-keying
- `Cirreum.SecretsProvider` — secret-store contracts and registration
- `Cirreum.ServiceProvider` — runtime service-registration plumbing used by other provider tracks

---

## Infrastructure

Provider implementations and host-specific services.

### Host services (main track)
- `Cirreum.Services.Server` — ASP.NET Core host services: Result-to-HTTP, ProblemDetails, the HTTP→`IInvocationContext` bridge, plus code-first SignalR and WebSocket invocation sources
- `Cirreum.Services.Wasm` — Blazor WASM host services
- `Cirreum.Services.Serverless` — Azure Functions host services

### Authentication providers
- `Cirreum.Authentication.ApiKey`
- `Cirreum.Authentication.Entra` — Microsoft Entra ID (workforce tenant)
- `Cirreum.Authentication.External` — external JWT bearer (arbitrary OIDC issuer)
- `Cirreum.Authentication.Oidc` — generic OIDC bearer
- `Cirreum.Authentication.SessionTicket` — opaque session-ticket scheme
- `Cirreum.Authentication.SignedRequest` — HMAC signed-request (server side)
- `Cirreum.Authentication.SignedRequest.Client` — outbound signing SDK

### Identity providers
- `Cirreum.Identity.Oidc` — generic OIDC provisioning (Auth0, Okta, Descope, Keycloak, …)
- `Cirreum.Identity.EntraExternalId` — Microsoft Entra External ID provisioning

### Communications / Messaging / Persistence / Storage / Secrets implementations
- `Cirreum.Communications.Email.Azure`
- `Cirreum.Communications.Email.SendGrid`
- `Cirreum.Communications.Sms.Azure`
- `Cirreum.Communications.Sms.Twilio`
- `Cirreum.Messaging.Azure` — Azure Service Bus
- `Cirreum.Persistence.Azure` — Azure Cosmos DB
- `Cirreum.Persistence.SqlServer` — SQL Server (Dapper)
- `Cirreum.Persistence.SQLite`
- `Cirreum.Storage.Azure` — Azure Blob Storage
- `Cirreum.Secrets.Azure` — Azure Key Vault

### Reusable infrastructure
- `Cirreum.Graph.Provider` — Microsoft Graph SDK provider
- `Cirreum.Introspection` — boot-time diagnostics and endpoint-posture analysis
- `Cirreum.QueryCache.Distributed` — distributed cache backing for cacheable operations
- `Cirreum.QueryCache.Hybrid` — hybrid (in-memory + distributed) cache backing

---

## Runtime

Host wiring and provider runtime cores.

### Main track
- `Cirreum.Runtime.Server`
- `Cirreum.Runtime.Wasm`
- `Cirreum.Runtime.Serverless`

### Provider track (runtime cores)
- `Cirreum.Runtime.AuthenticationProvider`
- `Cirreum.Runtime.IdentityProvider`
- `Cirreum.Runtime.SecretsProvider`
- `Cirreum.Runtime.ServiceProvider`

---

## Runtime Extensions

App-facing umbrellas — what your application actually installs. Each enables single-call registration that composes the lower layers.

### Authentication / Identity
- `Cirreum.Runtime.Authentication` — `builder.AddAuthentication(...)` umbrella
- `Cirreum.Runtime.Identity` — `builder.AddIdentity(...)` cross-protocol umbrella
- `Cirreum.Runtime.Identity.Oidc` *(generic OIDC: Auth0, Okta, Descope, Keycloak, …)*
- `Cirreum.Runtime.Identity.EntraExternalId` *(Microsoft Entra External ID)*
- `Cirreum.Runtime.Wasm.Msal` *(WASM client identity via MSAL — Entra workforce / B2C)*
- `Cirreum.Runtime.Wasm.Oidc` *(WASM client identity via generic OIDC)*

### Communications / Messaging
- `Cirreum.Runtime.Communications` *(email + SMS)*
- `Cirreum.Runtime.Messaging`

### Persistence
- `Cirreum.Runtime.Persistence` *(all persistence providers)*
- `Cirreum.Runtime.Persistence.Azure` *(Cosmos DB)*
- `Cirreum.Runtime.Persistence.SqlServer`
- `Cirreum.Runtime.Persistence.SQLite`

### Secrets / Storage
- `Cirreum.Runtime.Secrets` *(Azure Key Vault)*
- `Cirreum.Runtime.Storage` *(Azure Blob Storage)*

---

## Typical Server (API) package references

```xml
<PackageReference Include="Cirreum.Runtime.Server" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Secrets" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Authentication" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Identity" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Persistence.Azure" Version="1.0.*" />
<!-- OR -->
<PackageReference Include="Cirreum.Runtime.Persistence.SqlServer" Version="1.0.*" />
```

### Optional packages

For blob storage, SMS/email communications, and Service Bus messaging:

```xml
<PackageReference Include="Cirreum.Runtime.Storage" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Communications" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Messaging" Version="1.0.*" />
```
