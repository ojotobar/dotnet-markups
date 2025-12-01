# 1. Swagger / OpenAPI — advanced

## Goal

* Produce rich, meaningful OpenAPI docs for consumers.
* Include security, examples, XML comments, grouping, filters.

## Setup (Swashbuckle) — advanced config

```csharp
// Program.cs (or Startup)
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo {
        Title = "Attendance API",
        Version = "v1",
        Description = "QR attendance demo API",
        Contact = new OpenApiContact { Name = "Course Staff", Email = "staff@example.com" }
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlPath, includeControllerXmlComments: true);

    // JWT Bearer security
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer {token}'"
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement {
        {
            new OpenApiSecurityScheme { Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" } },
            new string[] {}
        }
    });

    // Example: Operation filter to add default response types
    c.OperationFilter<DefaultResponsesOperationFilter>();
});
```

## Operation filter example (add 401/403 responses automatically)

```csharp
public class DefaultResponsesOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        operation.Responses.TryAdd("401", new OpenApiResponse { Description = "Unauthorized" });
        operation.Responses.TryAdd("403", new OpenApiResponse { Description = "Forbidden" });
    }
}
```

## Add examples for request/response

Use `Swashbuckle.AspNetCore.Filters` to add `[SwaggerRequestExample]`/`[SwaggerResponseExample]` or directly use `OpenApiExample` in an OperationFilter to register schema examples — very useful to show consumers exactly what body to send.

---

# 2. Generating Live API Docs — writing useful examples

Swagger UI can do more than list endpoints. Teach students to:

* Provide concrete examples (request/response).
* Organize endpoints into tags (e.g., `Attendance`, `Users`).
* Show sample curl snippets via the UI.

Example: add example response with `ProducesResponseType(typeof(UserDto), 200)` and XML comments showing example content. Use schema filters to attach `example` JSON to schemas.

---

# 3. Writing Good Summaries & Response Type Annotations

## Why it matters

Clients need predictable contracts. Swagger + `ProducesResponseType` makes generated docs accurate and useful.

## Example: Controller with XML comments and response types

```csharp
/// <summary>
/// Registers a student for class and returns an attendance token.
/// </summary>
/// <param name="model">Registration request</param>
/// <returns>Created attendance token</returns>
[HttpPost("register")]
[ProducesResponseType(typeof(RegisterResponse), StatusCodes.Status201Created)]
[ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
public IActionResult Register([FromBody] RegisterRequest model)
{
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    var token = _service.CreateToken(model.StudentId);
    return CreatedAtAction(nameof(GetToken), new { id = token.Id }, new RegisterResponse { Token = token.Value });
}
```

## Use ProblemDetails for errors (RFC 7807)

```csharp
return Problem(detail: "Student ID not found", statusCode: 404, title: "NotFound");
```

Add global exception handling middleware to convert exceptions into `ProblemDetails`. This gives consistent error payloads.

---

# 4. Versioning: Why it’s important

* Breaking changes are inevitable.
* Versioning protects existing consumers.
* Allows iterative API improvements.

## Use Microsoft.AspNetCore.Mvc.Versioning

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
```

### Configure

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1,0);
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-API-Version")
    );
});
builder.Services.AddVersionedApiExplorer(options => {
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

---

# 5. Versioning Strategies: URL vs Header (detailed)

### URL versioning (simple, visible)

* Pros: easy to test, cache-friendly, human-friendly
* Cons: a lot of URLs to maintain

Controller example:

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV1Controller : ControllerBase { ... }

[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV2Controller : ControllerBase { ... }
```

### Header versioning (clean URL)

* Pros: cleaner URLs, easier to keep resource routes stable
* Cons: harder for users to discover version, caching proxies may need to include header

Client must set header:

```
X-API-Version: 2
```

### Teaching tip

Show them both, explain situations (public APIs prefer URL, internal APIs may prefer header). Demonstrate `ReportApiVersions` in responses (adds `api-supported-versions` header).

---

# 6. Server-side Validations — DataAnnotations and more

## DataAnnotations (built-in)

```csharp
public class RegisterRequest
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required, MinLength(6)]
    public string Password { get; set; }

    [Range(1, int.MaxValue)]
    public int CourseId { get; set; }
}
```

Controller pattern:

```csharp
[HttpPost]
public IActionResult Create([FromBody] RegisterRequest req)
{
    if (!ModelState.IsValid) return ValidationProblem(ModelState);
    ...
}
```

### Custom Validation Attribute

```csharp
public class NotReservedEmailAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        var email = value as string;
        if (email?.EndsWith("@reserved.com") == true)
            return new ValidationResult("Reserved domain not allowed");
        return ValidationResult.Success;
    }
}
```

### IValidatableObject for cross-field validation

```csharp
public class PasswordChange : IValidatableObject
{
    public string OldPassword { get; set; }
    public string NewPassword { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (OldPassword == NewPassword)
            yield return new ValidationResult("New must be different", new[] { nameof(NewPassword) });
    }
}
```

### Model binding pitfalls

* Null collections vs missing: account for `IList<T>` vs `IEnumerable<T>`
* Use `[FromBody]` vs default binding rules
* Large body sizes — set limits

---

# 7. FluentValidation — production usage

FluentValidation offers a clean DSL and is ideal for complex validation logic, reuse, and testability.

### Install & configure

```bash
dotnet add package FluentValidation.AspNetCore
```

```csharp
builder.Services.AddControllers()
    .AddFluentValidation(cfg => {
        cfg.RegisterValidatorsFromAssemblyContaining<Startup>();
    });
```

### Example validator with async rule and cascade control

```csharp
public class RegisterRequestValidator : AbstractValidator<RegisterRequest>
{
    public RegisterRequestValidator(IUserRepository userRepo)
    {
        RuleFor(x => x.Email)
            .NotEmpty().EmailAddress()
            .MustAsync(async (email, ct) => !await userRepo.ExistsAsync(email))
            .WithMessage("Email already in use");

        RuleFor(x => x.Password)
            .MinimumLength(8)
            .Matches("[A-Z]").WithMessage("Password must contain uppercase")
            .Matches("[0-9]").WithMessage("Password must contain digit");

        // conditional rule
        When(x => x.InviteCode != null, () =>
        {
            RuleFor(x => x.InviteCode).Length(6);
        });
    }
}
```

### Integrating into pipeline

FluentValidation integrates with the default model state behavior and automatically populates errors if validation fails.

### Testing validators

Teach students to unit test validators — they are small units of logic.

---

# 8. Paging, Filtering, Sorting — patterns & implementation

## Principles

* Use `IQueryable<T>` and apply paging/filtering/sorting at the DB level (avoid `ToList()` early).
* Return metadata (total count, page, pageSize) and navigational links (HATEOAS optionally).
* Support offset/limit and cursor-based pagination for large datasets.

## Offset paging (simple)

DTOs:

```csharp
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; }
    public int Total { get; set; }
    public int Page { get; set; }
    public int PageSize { get; set; }
}
```

Repository / controller:

```csharp
[HttpGet]
public async Task<PagedResult<UserDto>> GetUsers([FromQuery]int page = 1, [FromQuery]int pageSize = 20)
{
    var query = _dbContext.Users.AsQueryable();

    var total = await query.CountAsync();
    var items = await query
        .OrderBy(u => u.LastName)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(u => new UserDto { ... })
        .ToListAsync();

    return new PagedResult<UserDto> { Items = items, Total = total, Page = page, PageSize = pageSize };
}
```

## Cursor pagination (recommended for large datasets)

* Cursor = encoded last-seen value (e.g., timestamp + id).
* Pros: stable ordering, better performance.

Example approach:

* Client requests `?cursor=<base64>` or `?limit=50`.
* Server decodes cursor (e.g., lastCreatedAt + lastId) and `WHERE (CreatedAt < lastCreatedAt) OR (CreatedAt = lastCreatedAt AND Id < lastId)`.

## Filtering & Sorting combinatorics

Write reusable extension methods:

```csharp
public static IQueryable<T> ApplySorting<T>(this IQueryable<T> q, string sortBy, bool desc)
{
    return sortBy switch {
        "name" => desc ? q.OrderByDescendingDynamic("Name") : q.OrderByDynamic("Name"),
        _ => q
    };
}
```

Use `System.Linq.Dynamic.Core` for dynamic ordering or build switch/case mapping to expression trees for safer typing.

## Return metadata in headers

Return `X-Total-Count` header and `Link` header for pagination links (RFC 5988). Or include metadata in response body.

---

# 9. Structured Logging — make logs queryable

## Why structured logs

* Instead of free text, logs become JSON with fields the observability system can query.

## Use Serilog (example)

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Enrichers.CorrelationId
```

### Startup

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.json", rollingInterval: RollingInterval.Day, restrictedToMinimumLevel: LogEventLevel.Information, formatter: new Serilog.Formatting.Json.JsonFormatter())
    .CreateLogger();

builder.Host.UseSerilog();
```

### Enrichment & Correlation ID

Add middleware to generate a correlation id and add to log scope:

```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault() ?? Guid.NewGuid().ToString();
    context.Response.Headers["X-Correlation-ID"] = correlationId;
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

### Logging examples with structured properties

```csharp
_logger.LogInformation("User {UserId} scanned QR {QrId} for class {ClassId}", userId, qrId, classId);
```

This produces JSON fields: `UserId`, `QrId`, `ClassId`.

### Request logging & performance

Use `Serilog.AspNetCore` to log request durations, route, status code, and user id. Teach students to avoid logging sensitive fields (PII).

---

# 10. Git & GitHub Basics — beyond the basics

## Branching strategies

* **Git Flow**: `main` (stable), `develop` (integration), feature/release/hotfix branches — good for complex release cadence.
* **Trunk-based**: short-lived feature branches merged into `main` frequently (good for CI/CD).

Teach both and show tradeoffs: trunk-based + CI/CD enables faster delivery.

## Commit messages — conventional commits

Encourage `type(scope): subject` format:

```
feat(attendance): add QR generation endpoint
fix(auth): validate token expiry
chore(deps): bump Swashbuckle
```

This makes changelogs and automation easier.

## Pull Request checklist (teach students a PR template)

* Does it build?
* Have tests been added?
* Any breaking changes?
* Update docs/Swagger examples if required

## GitHub Actions CI example (build + test)

`.github/workflows/ci.yml`:

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: 7.0.x
      - name: Restore
        run: dotnet restore
      - name: Build
        run: dotnet build --no-restore --configuration Release
      - name: Test
        run: dotnet test --no-build --verbosity normal
```

## Protect branches, require PR review and passing CI before merge — enforce quality.

---

# Extra: Attendance live demo extensions (practical teaching points)

* After scanning, push attendance record into an in-memory cache or DB and broadcast via SignalR to the generator page for live updates.
* Show how to implement rate-limiting (prevent multiple scans in a short time) and anti-replay (store last token id per student).
* Token revocation example: store token IDs in Redis and check on scan.

---

# Pitfalls & Security Notes (must-teach)

* Never store secret keys in code. Use **Azure Key Vault / AWS Secrets Manager / environment variables**.
* Short-lived JWTs + token id blacklisting for instant revocation.
* Avoid logging tokens or PII.
* Validate at backend even if front-end validates.
* Use HTTPS always (camera + QR pages over HTTPS required on mobile browsers).

---

# Appendix — Useful code snippets to hand to students

1. **Global exception to ProblemDetails middleware** (short)

```csharp
app.UseExceptionHandler(a => a.Run(async context =>
{
    var feature = context.Features.Get<IExceptionHandlerPathFeature>();
    var ex = feature?.Error;
    var pd = new ProblemDetails { Title = "An error occurred", Detail = ex?.Message, Status = 500 };
    context.Response.ContentType = "application/problem+json";
    context.Response.StatusCode = 500;
    await context.Response.WriteAsJsonAsync(pd);
}));
```

2. **EF Core IQueryable extension for paging**

```csharp
public static async Task<PagedResult<T>> ToPagedResultAsync<T>(this IQueryable<T> q, int page, int pageSize, CancellationToken ct=default)
{
    var total = await q.CountAsync(ct);
    var items = await q.Skip((page-1)*pageSize).Take(pageSize).ToListAsync(ct);
    return new PagedResult<T>{ Items = items, Total = total, Page = page, PageSize = pageSize};
}
```