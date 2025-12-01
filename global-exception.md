# ✅ What you wrote (the old-school way)

Your method:

```csharp
app.UseExceptionHandler(builder =>
{
    builder.Run(async context =>
    {
        var contextFeature = context.Features.Get<IExceptionHandlerFeature>();
        ...
    });
});
```

### ✅ What it does RIGHT

* Centralized exception handling ✔
* Status-code mapping ✔
* Structured API response ✔
* Logging ✔

### ❌ Why it’s outdated

| Issue                           |
| ------------------------------- |
| Uses terminal pipeline (`Run`)  |
| Hard to test                    |
| No DI                           |
| No ProblemDetails               |
| Can’t reuse logic               |
| No standard HTTP error contract |
| Not extensible                  |

It works — but it’s not the future.

---

# 🚀 Modern Replacement (Exact Behavior — Clean Version)

Let’s build the **modern equivalent** using:

✅ `IExceptionHandler`
✅ Full DI
✅ Status-code mapping
✅ Unified JSON response
✅ No `Run()`
✅ No feature plumbing
✅ No spaghetti

---

# ✅ Step 1: Create Custom Exceptions

```csharp
public sealed class BadRequestException : Exception
{
    public BadRequestException(string message) : base(message) {}
}

public sealed class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) {}
}

public sealed class ForbiddenException : Exception
{
    public ForbiddenException(string message) : base(message) {}
}
```

---

# ✅ Step 2: Your API Result Type

```csharp
public record ApiResult<T>(T Message, int StatusCode);
```

---

# ✅ Step 3: The Modern Global Handler

Create: `GlobalExceptionHandler.cs`

```csharp
using Microsoft.AspNetCore.Diagnostics;
using System.Net;
using System.Text.Json;

public sealed class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext context,
        Exception exception,
        CancellationToken ct)
    {
        _logger.LogError(exception, "An error occurred");

        var (statusCode, message) = exception switch
        {
            BadRequestException => (HttpStatusCode.BadRequest, exception.Message),
            NotFoundException => (HttpStatusCode.NotFound, exception.Message),
            ForbiddenException => (HttpStatusCode.Forbidden, exception.Message),
            _ => (HttpStatusCode.InternalServerError, "An internal server error occurred")
        };

        var response = new ApiResult<string>(message, (int)statusCode);

        context.Response.ContentType = "application/json";
        context.Response.StatusCode = (int)statusCode;

        await context.Response.WriteAsync(JsonSerializer.Serialize(response), ct);

        return true;
    }
}
```

💥 This does **everything your code does** — but:

* Cleaner
* Injectable
* Testable
* Extensible
* No pipeline hijack
* No feature digging

---

# ✅ Step 4: Register It Properly (Program.cs)

```csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
```

Then:

```csharp
app.UseExceptionHandler();
```

✅ That’s all.

---

# ✅ Step 5: Controller Usage

```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    if (id <= 0)
        throw new BadRequestException("Invalid user id");

    if (id == 99)
        throw new ForbiddenException("Access denied");

    if (id == 42)
        throw new NotFoundException("User not found");

    return Ok("Success");
}
```

---

# ✅ Response Examples

### `/users/42`

```json
{
  "message": "User not found",
  "statusCode": 404
}
```

### `/users/0`

```json
{
  "message": "Invalid user id",
  "statusCode": 400
}
```

### Unexpected error:

```json
{
  "message": "An internal server error occurred",
  "statusCode": 500
}
```

---

# ✅ Why this wins over your extension method

| Old Extension                   | New Handler         |
| ------------------------------- | ------------------- |
| `UseExceptionHandler` + `Run()` | `IExceptionHandler` |
| Static method                   | Injectable service  |
| Feature plumbing                | Direct access       |
| Rigid                           | Extensible          |
| Hard to unit test               | Unit testable       |
| Manual JSON                     | Standard flow       |

---

# 🧠 Tutor Insight (Tell Your Students This)

> “We don’t write pipeline hacks anymore —
> We plug into the framework properly.”

This modern pattern is:

* used in enterprise ASP.NET Core
* supported by Microsoft
* forward compatible
* clean architecture friendly