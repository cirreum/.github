# Cirreum Package Architecture

This document describes the layered architecture of Cirreum packages. Dependencies flow in one direction only:

```
L5 → L4 → L3 → L2 → L1 → L0
```

> **Note:** Not every package references all layers. For example, most Layer 3 packages reference Layer 1, but some also reference Layer 2.

---

## Layer 0 (Base)

Dependency free - could be used by any application/consumer, not just Cirreum.

- `Cirreum.Result` - Result, Result<T> and Optional<T?> monads/railway
- `Cirreum.Exceptions` - Predetermined application exceptions

---

## Layer 1 (Common)

Abstractions, dependency light - technically could be used by any application/consumer, not just Cirreum. Unless noted, assumed Server/Serverless only.

- `Cirreum.Communications.Email`
- `Cirreum.Communications.Sms`
- `Cirreum.Cors`
- `Cirreum.ExpressionBuilder` *(env/host agnostic)*
- `Cirreum.Logging.Deferred`
- `Cirreum.Messaging`
- `Cirreum.Persistence.NoSql`
- `Cirreum.Persistence.Sql`
- `Cirreum.Providers`
- `Cirreum.Startup` *(env/host agnostic)*
- `Cirreum.Storage`
- `Cirreum.Storage.Browser` *(Blazor WASM/Browser)*

---

## Layer 2 (Core)

Dependency light - Cross Env/Host (Browser, Server, Serverless).

### App Track
- `Cirreum.Components.WebAssembly` - Blazor WASM Client Components Library
- `Cirreum.Core` - Application Core/Kernel

### Provider Track
- `Cirreum.AuthorizationProvider` - extends Cirreum.Providers
- `Cirreum.SecretsProvider` - extends Cirreum.Providers
- `Cirreum.ServiceProvider` - extends Cirreum.Providers

---

## Layer 3 (Infrastructure)

### Provider Track (implementations)
- `Cirreum.Authorization.ApiKey` - extends Cirreum.AuthorizationProvider
- `Cirreum.Authorization.Entra` - extends Cirreum.AuthorizationProvider
- `Cirreum.Authorization.SignedRequest` - extends Cirreum.AuthorizationProvider
- `Cirreum.Authorization.SignedRequest.Client` - extends Cirreum.AuthorizationProvider
- `Cirreum.Communications.Email.Azure` - extends Cirreum.Communications.Email and Cirreum.ServiceProvider
- `Cirreum.Communications.Email.SendGrid` - extends Cirreum.Communications.Email and Cirreum.ServiceProvider
- `Cirreum.Communications.Sms.Azure` - extends Cirreum.Communications.Sms and Cirreum.ServiceProvider
- `Cirreum.Communications.Sms.Twilio` - extends Cirreum.Communications.Sms and Cirreum.ServiceProvider
- `Cirreum.Messaging.Azure` - extends Cirreum.Messaging and Cirreum.ServiceProvider
- `Cirreum.Persistence.Azure` - extends Cirreum.Persistence.NoSql and Cirreum.ServiceProvider
- `Cirreum.Persistence.SqlServer` - extends Cirreum.Persistence.Sql and Cirreum.ServiceProvider
- `Cirreum.Persistence.SQLite` - extends Cirreum.Persistence.Sql and Cirreum.ServiceProvider
- `Cirreum.Storage.Azure` - extends Cirreum.ServiceProvider
- `Cirreum.Secrets.Azure` - extends Cirreum.SecretsProvider

### Reusable Infrastructure
Depends on Cirreum.Core.

- `Cirreum.Graph.Provider` - Reusable MS Graph provider
- `Cirreum.QueryCache.Distributed` - optionally provides caching for Cirreum.Core (Conductor.ICacheableOperation)
- `Cirreum.QueryCache.Hybrid` - optionally provides caching for Cirreum.Core (Conductor.ICacheableOperation)

### App Track
Implements Cirreum.Core host-specific services, enforces patterns/conventions.

- `Cirreum.Services.Client`
- `Cirreum.Services.Server`
- `Cirreum.Services.Serverless`

---

## Layer 4 (Runtime)

Hosting implementation.

### Provider Track
- `Cirreum.Runtime.AuthorizationProvider`
- `Cirreum.Runtime.SecretsProvider`
- `Cirreum.Runtime.ServiceProvider`

### App Track
- `Cirreum.Runtime.Client`
- `Cirreum.Runtime.Server`
- `Cirreum.Runtime.Serverless`

---

## Layer 5 (Runtime Extensions)

Optional extensions that compose lower layers and services. Allows single-call registration of all Registrars using appsettings/Cirreum.Secrets.

### Provider Track
- `Cirreum.Runtime.Authorization`
- `Cirreum.Runtime.Client.Msal`
- `Cirreum.Runtime.Client.Oidc`
- `Cirreum.Runtime.Communications` *(SMS and Email)*
- `Cirreum.Runtime.Messaging`
- `Cirreum.Runtime.Persistence` *(all persistence providers)*
- `Cirreum.Runtime.Persistence.Azure` *(just the Azure provider)*
- `Cirreum.Runtime.Persistence.SqlServer` *(just the SqlServer provider)*
- `Cirreum.Runtime.Persistence.SQLite` *(just the SQLite provider)*
- `Cirreum.Runtime.Secrets` *(used internally by all others for KeyVault)*
- `Cirreum.Runtime.Storage`

---

## Typical Server (API) Package References

```xml
<PackageReference Include="Cirreum.Runtime.Server" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Secrets" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Authorization" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Persistence.Azure" Version="1.0.*" />
<!-- OR -->
<PackageReference Include="Cirreum.Runtime.Persistence.SqlServer" Version="1.0.*" />
```

### Optional Packages

For blob storage, SMS/email communications, and Service Bus messaging:

```xml
<PackageReference Include="Cirreum.Runtime.Storage" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Communications" Version="1.0.*" />
<PackageReference Include="Cirreum.Runtime.Messaging" Version="1.0.*" />
```
