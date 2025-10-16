## 🧱 1. Clean Architecture Overview

Clean Architecture separates concerns into **four concentric layers**:

```
Presentation (API/UI)
↑
Application (Use Cases)
↑
Domain (Entities & Logic)
↑
Infrastructure (External Integrations)
```

Each layer *depends inward* — outer layers can depend on inner layers, but not vice versa.

---

## 🗂️ 2. Folder & Project Structure

Here’s a recommended **solution layout**:

```
src/
 ├── MyApp.Domain/
 │    ├── Entities/
 │    ├── ValueObjects/
 │    ├── Enums/
 │    ├── Interfaces/
 │    └── DomainEvents/
 │
 ├── MyApp.Application/
 │    ├── DTOs/
 │    ├── Interfaces/
 │    ├── Features/
 │    │     ├── Users/
 │    │     │    ├── Commands/
 │    │     │    └── Queries/
 │    ├── Common/
 │    ├── Behaviors/
 │    └── Mappings/
 │
 ├── MyApp.Infrastructure/
 │    ├── Persistence/
 │    │     ├── MyAppDbContext.cs
 │    │     ├── Configurations/
 │    │     ├── Repositories/
 │    ├── Services/
 │    └── DependencyInjection.cs
 │
 ├── MyApp.API/
 │    ├── Controllers/
 │    ├── Models/              ← ViewModels or API models
 │    ├── Filters/
 │    ├── Middlewares/
 │    ├── Program.cs
 │    └── appsettings.json
 │
tests/
 ├── MyApp.UnitTests/
 └── MyApp.IntegrationTests/
```

---

## 🧩 3. Layer-by-Layer Breakdown

### 🧠 Domain Layer (`MyApp.Domain`)

> Pure business rules — no EF Core, no API dependencies.

**Contents:**

* Entities (`User`, `Order`, etc.)
* Value Objects (e.g. `Email`, `Money`)
* Interfaces for domain services (e.g. `IAggregateRoot`)
* Domain Events (if using DDD)

**Example:**

```csharp
public class User
{
    public Guid Id { get; private set; }
    public string FullName { get; private set; }
    public string Email { get; private set; }

    public User(string fullName, string email)
    {
        FullName = fullName;
        Email = email;
    }
}
```

---

### ⚙️ Application Layer (`MyApp.Application`)

> Contains use cases, business logic, and contracts between layers.

**Contents:**

* **DTOs** – Used to transfer data between Application and Presentation.
* **Interfaces** – Define contracts to be implemented in Infrastructure (e.g. `IUserRepository`).
* **Features** – Each use case (CQRS style: `Commands` and `Queries`).
* **Mappings** – AutoMapper profiles (Entity ⇆ DTO).
* **Validation** – FluentValidation or DataAnnotations.

**Example DTO:**

```csharp
public class UserDto
{
    public Guid Id { get; set; }
    public string FullName { get; set; }
}
```

**Example Command:**

```csharp
public record CreateUserCommand(string FullName, string Email) : IRequest<UserDto>;
```

**Example Command Handler:**

```csharp
public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, UserDto>
{
    private readonly IUserRepository _userRepository;
    private readonly IMapper _mapper;

    public CreateUserCommandHandler(IUserRepository userRepository, IMapper mapper)
    {
        _userRepository = userRepository;
        _mapper = mapper;
    }

    public async Task<UserDto> Handle(CreateUserCommand request, CancellationToken cancellationToken)
    {
        var user = new User(request.FullName, request.Email);
        await _userRepository.AddAsync(user);
        return _mapper.Map<UserDto>(user);
    }
}
```

---

### 🧩 Infrastructure Layer (`MyApp.Infrastructure`)

> Implements interfaces defined in the Application layer.

**Contents:**

* `Persistence/` for EF Core DbContext
* Repositories implementing interfaces
* External services (SMTP, AWS, etc.)
* `DependencyInjection.cs` for wiring it all up

**Example Repository:**

```csharp
public class UserRepository : IUserRepository
{
    private readonly MyAppDbContext _context;
    public UserRepository(MyAppDbContext context) => _context = context;

    public async Task AddAsync(User user)
    {
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
    }
}
```

---

### 🌐 Presentation Layer (`MyApp.API`)

> Web API entry point — handles HTTP, validation, and presentation models (ViewModels).

**Contents:**

* Controllers (use DTOs or ViewModels)
* ViewModels (for shaping data per API endpoint)
* Middlewares (error handling, logging)
* API versioning setup

**Example ViewModel:**

```csharp
public class CreateUserViewModel
{
    [Required] public string FullName { get; set; }
    [EmailAddress] public string Email { get; set; }
}
```

**Controller Example:**

```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateUserViewModel model)
    {
        var result = await _mediator.Send(new CreateUserCommand(model.FullName, model.Email));
        return CreatedAtAction(nameof(GetById), new { id = result.Id }, result);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(Guid id)
    {
        var user = await _mediator.Send(new GetUserByIdQuery(id));
        return Ok(user);
    }
}
```

---

## 🔁 4. Where Each Type Fits

| Type          | Layer                 | Purpose                                                     |
| ------------- | --------------------- | ----------------------------------------------------------- |
| **Entity**    | Domain                | Represents the core data and rules                          |
| **DTO**       | Application           | Transfers data between layers                               |
| **ViewModel** | Presentation (API/UI) | Shapes data for client needs (e.g. form inputs, UI binding) |

You could say:

* **Entities** are for the *business*.
* **DTOs** are for the *system*.
* **ViewModels** are for the *user*.

---

## 🧭 5. API Versioning and DTOs

Each version of your API can have its own DTOs:

```
MyApp.Application/
 ├── DTOs/
 │    ├── v1/
 │    └── v2/
```

In controllers:

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
```

This lets you evolve API contracts without breaking old clients.

---

## 🧰 6. Validation and Data Annotations

**On ViewModels (Presentation):**

```csharp
public class RegisterViewModel
{
    [Required] public string Username { get; set; }
    [EmailAddress] public string Email { get; set; }
}
```

**In Application Layer (FluentValidation):**

```csharp
public class CreateUserCommandValidator : AbstractValidator<CreateUserCommand>
{
    public CreateUserCommandValidator()
    {
        RuleFor(x => x.FullName).NotEmpty();
        RuleFor(x => x.Email).EmailAddress();
    }
}
```

---

## ⚙️ 7. Dependency Injection Wiring

Each layer registers its services in a `DependencyInjection.cs` file:

**Example (Infrastructure):**

```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<MyAppDbContext>(options =>
            options.UseSqlServer(config.GetConnectionString("DefaultConnection")));

        services.AddScoped<IUserRepository, UserRepository>();
        return services;
    }
}
```

Then in `Program.cs`:

```csharp
builder.Services
    .AddApplication()
    .AddInfrastructure(builder.Configuration);
```