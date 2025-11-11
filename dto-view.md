# Understanding DTO vs View Models vs Entities — with examples

Nice slide — this is a classic set of concepts every .NET dev needs in their toolbox. I’ll explain each concept, show typical C# examples, give mapping patterns (manual vs AutoMapper), and cover API versioning and validation (data annotations + custom validation). I’ll be pragmatic and point out common pitfalls and recommendations along the way.

---

# Entities

**What:** Domain objects that represent your persisted state (database rows). They usually include persistence concerns (IDs, navigation properties, EF Core attributes), and domain behavior in DDD-style apps.

**Where used:** Data layer, repositories, EF Core `DbContext`.

**Responsibility:** Shape of stored data and invariants that belong to the domain model.

**Example (EF Core entity):**

```csharp
public class User
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Email { get; set; } = default!;
    public string PasswordHash { get; set; } = default!;
    public bool IsEmailConfirmed { get; set; }

    // Navigation property
    public ICollection<UserRole> Roles { get; set; } = new List<UserRole>();

    // Domain method example
    public void ConfirmEmail() => IsEmailConfirmed = true;
}
```

**Notes:**

* Entities often have persistence attributes (`[Key]`, `[Column]`) or fluent mappings.
* Keep entities focused on domain/DB concerns; avoid wiring in presentation or API concerns (like JSON-only fields, or validation attributes meant for user input).

---

# DTOs (Data Transfer Objects)

**What:** Lightweight objects used to transfer data across boundaries — typically between API and clients or between microservices. DTOs *do not* usually contain persistence logic or heavy domain behavior.

**Why use them:**

* Avoid leaking entity implementation to clients (security, coupling).
* Shape responses/requests specifically for API surface and versioning.
* Allow different serialization/validation rules than domain entities.

**Types:**

* Request DTOs (incoming payloads): `RegisterUserRequestDto`
* Response DTOs (what you return): `UserPublicDto`
* Event DTOs (for messages): `EmailConfirmationRequestedEventDto`

**Example:**

```csharp
public class RegisterUserRequestDto
{
    public string Email { get; set; } = default!;
    public string Password { get; set; } = default!;
}

public class UserPublicDto
{
    public Guid Id { get; set; }
    public string Email { get; set; } = default!;
    public bool IsEmailConfirmed { get; set; }
}
```

**Notes:**

* DTOs are the perfect place for input validation attributes and for shaping exactly what a client can see or provide.
* DTOs evolve independently of your DB schema, which helps with API versioning.

---

# View Models

**What:** Objects created for rendering UI (Razor pages, MVC views, or frontend models). A view model contains exactly what a view needs (fields for display and controls), often including properties for UI concerns (select lists, combined strings).

**Where used:** MVC controller actions that return views, Razor Pages, or UI-oriented APIs.

**Example (Razor/MVC):**

```csharp
public class UserEditViewModel
{
    public Guid Id { get; set; }
    [Display(Name="Email address")]
    public string Email { get; set; } = default!;
    public bool IsEmailConfirmed { get; set; }

    // UI convenience
    public IEnumerable<SelectListItem> AvailableRoles { get; set; } = Enumerable.Empty<SelectListItem>();
}
```

**Difference from DTOs:**

* ViewModels are UI-focused and can contain UI-only helpers (SelectList, formatted strings).
* DTOs are API-focused and intended for network transfer.

---

# Manual Mapping vs AutoMapper

Mapping = converting between Entities ↔ DTOs ↔ ViewModels.

## Manual Mapping

**What:** You write code to convert objects yourself.

**Why:** Full control, explicitness, easy to debug, no magic.

**Example:**

```csharp
// Entity -> DTO
public static UserPublicDto ToDto(User user) => new()
{
    Id = user.Id,
    Email = user.Email,
    IsEmailConfirmed = user.IsEmailConfirmed
};

// DTO -> Entity (for registration)
public static User ToEntity(RegisterUserRequestDto dto)
{
    return new User
    {
        Email = dto.Email,
        PasswordHash = HashPassword(dto.Password),
        IsEmailConfirmed = false
    };
}
```

**Pros:**

* Clear, explicit, easy to reason about.
* Small overhead, fast.
* Great for complex transformations with logic.

**Cons:**

* Boilerplate when many types/fields exist.
* Easy to forget a field when types evolve.

## AutoMapper

**What:** A popular library that uses configuration/profiles to map properties automatically by name, with extension points for custom mappings.

**Setup example:**

```csharp
// Install AutoMapper.Extensions.Microsoft.DependencyInjection

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<User, UserPublicDto>();
        CreateMap<RegisterUserRequestDto, User>()
            .ForMember(dest => dest.PasswordHash, opt => opt.Ignore()); // we handle password separately
    }
}

// Startup.cs / Program.cs
services.AddAutoMapper(typeof(MappingProfile));
```

**Usage:**

```csharp
var dto = _mapper.Map<UserPublicDto>(user);
var user = _mapper.Map<User>(registerDto);
```

**Pros:**

* Reduces repetitive mapping code.
* Centralized configuration.
* Supports custom resolvers and complex logic.

**Cons:**

* Hidden mapping logic can be surprising during debugging.
* Overhead in cold start; sometimes tricky with nested and collection mappings.
* For critical or security-sensitive transformations (e.g., password hashing) prefer explicit code.

**Recommendation:** Use AutoMapper for large APIs with many simple mappings. For sensitive fields or when you need exact control, prefer explicit mapping.

---

# API Versioning, Validation, and Data Annotations on DTOs

## API Versioning

**What:** Manage breaking changes in APIs by allowing multiple API versions to co-exist.

**ASP.NET Core approaches:**

* URL segment: `/api/v1/users`
* Query string: `/api/users?api-version=1.0`
* Header-based: `Api-Version: 1.0`

**Example (Microsoft.AspNetCore.Mvc.Versioning):**

```csharp
// Program.cs
services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
});
```

**Controller attribute example:**

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetV1() => Ok(...);
}

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("2.0")]
public class UsersV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult GetV2() => Ok(...); // different shape or behavior
}
```

**Guidance:**

* Use a clear versioning strategy (URL segment is the most explicit).
* Maintain small, incremental versions; document the contract changes.
* Keep DTOs versioned: `UserDto.v1`, `UserDto.v2` or namespaces like `Dtos.V1.UserDto`.

---

## Validation

**What:** Ensure incoming DTOs are valid (required fields, formats, length) before hitting business logic.

**Where:** Validate at the boundary (controller/model binding) — typically on DTO classes.

**Built-in approach:** Data Annotations + `[ApiController]` auto-validation.

**Example DTO with DataAnnotations:**

```csharp
public class RegisterUserRequestDto
{
    [Required, EmailAddress]
    public string Email { get; set; } = default!;

    [Required, MinLength(8)]
    public string Password { get; set; } = default!;
}
```

**Behavior:** With `[ApiController]`, invalid models cause automatic `400 Bad Request` with validation details.

**More advanced:** Use FluentValidation for richer rules and composition.

---

## Data Annotation

**Use:** Declarative validation on DTO properties via attributes from `System.ComponentModel.DataAnnotations`.

**Common attributes:**

* `[Required]`, `[EmailAddress]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[Compare]`, `[MaxLength]`, `[MinLength]`, `[Phone]`.

**Example:**

```csharp
public class CreateProductDto
{
    [Required]
    public string Name { get; set; } = default!;

    [Range(0.01, 100000)]
    public decimal Price { get; set; }

    [StringLength(500)]
    public string? Description { get; set; }
}
```

**Notes:**

* DataAnnotations are great for straightforward validation and for generating docs (Swagger).
* They are declarative and quick, but sometimes not expressive enough for complex rules.

---

## Custom Validation

When built-in attributes aren’t enough, create custom validation attributes or implement `IValidatableObject`.

### Custom `ValidationAttribute` example:

```csharp
public class NotDisposableEmailAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        var email = value as string;
        if (email == null) return ValidationResult.Success;

        var disposableDomains = new[] { "mailinator.com", "tempmail.com" };
        var domain = email.Split('@').LastOrDefault();
        if (domain != null && disposableDomains.Contains(domain.ToLower()))
            return new ValidationResult("Disposable email addresses not allowed.");

        return ValidationResult.Success;
    }
}

public class RegisterUserRequestDto
{
    [Required, EmailAddress, NotDisposableEmail]
    public string Email { get; set; } = default!;

    [Required, MinLength(8)]
    public string Password { get; set; } = default!;
}
```

### `IValidatableObject` example (for cross-field validation):

```csharp
public class CreateBookingDto : IValidatableObject
{
    public DateTime Start { get; set; }
    public DateTime End { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (End <= Start)
            yield return new ValidationResult("End must be after Start", new[] { nameof(End), nameof(Start) });
    }
}
```

**Where to put validation logic?**

* Simple format/required checks: DataAnnotations on DTOs.
* Cross-field rules: `IValidatableObject` or custom attribute on the class.
* Complex domain rules: Prefer domain-level validation on entities/services (e.g., business invariants like “user must be 18+” that should be enforced regardless of API).

---

# Putting it together — Typical flow in an ASP.NET Core app

1. **Request**: client POSTs `RegisterUserRequestDto` to `POST /api/v1/auth/register`.
2. **Model binding + validation**: `[ApiController]` auto-validates DTO using DataAnnotations. A `400` is returned if invalid.
3. **Controller**: maps DTO → Entity (manual or AutoMapper). If passwords present, hash here (explicitly).
4. **Repository**: save `User` entity to DB.
5. **Event**: publish domain event (optional).
6. **Response**: return `UserPublicDto` (entity mapped → DTO) or `202 Accepted` if asynchronous flow.

Example controller using manual mapping and validation:

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/auth")]
[ApiVersion("1.0")]
public class AuthController : ControllerBase
{
    [HttpPost("register")]
    public async Task<IActionResult> Register(RegisterUserRequestDto dto)
    {
        // ModelState is automatically validated by [ApiController]
        var user = new User { Email = dto.Email, PasswordHash = HashPassword(dto.Password) };
        await _userRepo.AddAsync(user);
        var resultDto = new UserPublicDto { Id = user.Id, Email = user.Email, IsEmailConfirmed = user.IsEmailConfirmed };
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, resultDto);
    }
}
```

---

# Interview-ready talking points (short)

* **Why DTOs?** Prevents leaking DB structure, allows API-specific shapes, and simplifies versioning.
* **When to use AutoMapper?** Good for simple, repetitive mappings; avoid for critical fields (hashing, security).
* **Where to validate?** At API boundary (DTOs) for input, at domain level for invariants.
* **Versioning approach:** Use route-versioning (URL segments) for clarity; version DTOs, not entities.
* **Idempotency & mapping:** Always design mapping to be deterministic and consider idempotency for requests that may be retried.

---

# Common pitfalls & how to avoid them

* **Putting UI-only properties on entities** → leaks presentation concerns; use ViewModels/DTOs instead.
* **Relying on AutoMapper for security logic** (e.g., expecting AutoMapper to hash password) → do hashing explicitly.
* **Validating only in the controller and not in domain** → domain invariants should be enforced even if the API changes.
* **Changing entity schema breaks API** → version DTOs independently of entities.

---