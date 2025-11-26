# Cirreum

<h4>Layered simplicity for modern .NET</h4>

<p align="right">
  <img src="https://raw.githubusercontent.com/cirreum/.github/main/profile/cirreum-icon-transparent-002.png" width=180" alt="Cirreum Logo" />
</p>

## **A modern, opinionated foundation framework for building scalable, runtime-agnostic .NET applications with Railway-Oriented Programming at its core.**

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

### 🚂 [Cirreum.Conductor](https://github.com/cirreum/Cirreum.Conductor)
Request/response pipeline with intercept-based architecture. Dispatch commands and queries through a configurable pipeline with automatic authorization, validation, and cross-cutting concerns.

**Key Features:**
- MediatR-style request dispatching with `Result<T>` return types
- Intercept pattern for cross-cutting concerns
- Generic constraint-based registration
- Full async/await support with cancellation

### 🔐 [Cirreum.Authorization](https://github.com/cirreum/Cirreum.Authorization)
Resource-level authorization with role-based access control. FluentValidation-style syntax for declaring authorization rules.

**Key Features:**
- Resource-specific authorization validators
- Runtime-wide policy validators
- Role hierarchy and inheritance
- Automatic integration with Conductor pipeline
- Works with any authentication provider

### ✅ [Cirreum.Validation](https://github.com/cirreum/Cirreum.Validation)
FluentValidation integration for request validation with automatic Railway conversion.

**Key Features:**
- Automatic validation through Conductor intercept
- Converts validation failures to `Result.Fail()`
- RFC 7807 ProblemDetails format
- Custom validation rules and async validators

### 🗄️ [Cirreum.Persistence](https://github.com/cirreum/Cirreum.Persistence)
Abstractions and implementations for data access with repository and unit of work patterns.

**Key Features:**
- Generic repository pattern
- Unit of work coordination
- Entity Framework Core integration
- Cosmos DB support
- Specification pattern for queries

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

### 🎨 [Cirreum.Blazor](https://github.com/cirreum/Cirreum.Blazor)
Reusable Blazor components with Bootstrap theming support.

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
dotnet add package Cirreum.Conductor
dotnet add package Cirreum.Authorization
dotnet add package Cirreum.Services.Server
```

### 2. Configure Services
```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Cirreum Server infrastructure
builder.Services.AddCirreumServer();

// Add Conductor with intercepts
builder.Services.AddCirreumConductor(options => {
    options.AddIntercept<AuthorizationIntercept>();
    options.AddIntercept<ValidationIntercept>();
});

// Add authorization
builder.Services.AddCirreumAuthorization(options => {
    options.RegisterRole<AdminRole>();
    options.RegisterRole<UserRole>();
});

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

### 3. Define a Request
```csharp
public record CreateResourceRequest(string Name) : IAuthorizableRequest<ResourceDto> {
    // Authorization handled automatically by Conductor
}
```

### 4. Implement Handler
```csharp
public class CreateResourceHandler : IRequestHandler<CreateResourceRequest, ResourceDto> {
    private readonly IRepository<Resource> _repository;

    public CreateResourceHandler(IRepository<Resource> repository) {
        _repository = repository;
    }

    public async Task<Result<ResourceDto>> Handle(
        CreateResourceRequest request,
        CancellationToken cancellationToken) {
        
        var resource = new Resource { Name = request.Name };
        await _repository.AddAsync(resource, cancellationToken);
        
        return Result<ResourceDto>.Success(
            new ResourceDto(resource.Id, resource.Name)
        );
    }
}
```

### 5. Create Controller
```csharp
[ApiController]
[Route("api/[controller]")]
public class ResourcesController : ControllerBase {
    private readonly IDispatcher _dispatcher;

    public ResourcesController(IDispatcher dispatcher) {
        _dispatcher = dispatcher;
    }

    [HttpPost]
    public async Task<Result<ResourceDto>> Create(
        [FromBody] CreateResourceRequest request,
        CancellationToken cancellationToken) {
        
        // Authorization, validation, and Result→HTTP conversion happen automatically
        return await _dispatcher.Dispatch(request, cancellationToken);
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

// Cirreum approach
public async Task<Result<ResourceDto>> Handle(GetResourceQuery query, CancellationToken ct) {
    var resource = await _repository.GetById(query.Id);
    
    return resource is not null
        ? Result<ResourceDto>.Success(MapToDto(resource))
        : Result<ResourceDto>.Fail(new NotFoundException("Resource not found"));
    
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

## Multi-Runtime Support

### Server (ASP.NET Core)
```csharp
[HttpGet("{id}")]
public async Task<Result<ResourceDto>> Get(string id) {
    return await _dispatcher.Dispatch(new GetResourceQuery(id));
}
// Returns: 200 OK with JSON, or 404/403 with ProblemDetails
```

### WASM (Blazor)
```csharp
private async Task LoadResource() {
    var result = await Dispatcher.Dispatch(new GetResourceQuery(ResourceId));
    
    result.Switch(
        onSuccess: resource => {
            Resource = (ResourceDto)resource!;
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
|---------|---------------------|---------|---------|
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
- [ ] Performance benchmarks
- [ ] Complete documentation site

## License

Cirreum is licensed under the MIT License. See individual repositories for details.

## Support

- 💬 **Discussions**: [GitHub Discussions](https://github.com/orgs/cirreum/discussions)
- 🐛 **Issues**: Report issues in the relevant repository
- 📧 **Contact**: [Contact Information]

---

**Built with ❤️ for the .NET community**
