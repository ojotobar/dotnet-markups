# 📘 CLASS NOTES – API DEVELOPMENT FUNDAMENTALS (C# / ASP.NET Core)

---

# 1. Swagger Documentation

## What is Swagger?

Swagger is like your API's **user manual that updates itself**.
It:

* Lists all your endpoints
* Shows request and response formats
* Allows live testing from the browser

In ASP.NET Core, Swagger is provided via **Swashbuckle**.

---

## Setup in ASP.NET Core

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

app.UseSwagger();
app.UseSwaggerUI();
```

Now open:

```
https://localhost:5001/swagger
```

Congratulations, you officially have live API documentation.

---

# 2. Generating Live API Docs

Swagger automatically reads:

* Controllers
* Routes
* Parameters
* Models

Example:

```csharp
[HttpGet("users/{id}")]
public IActionResult GetUser(int id)
{
    return Ok(new { Id = id, Name = "Toba" });
}
```

This instantly appears in Swagger.

---

# 3. Writing Good Summaries & Response Type Annotations

Your API should explain itself.

---

## ✅ XML Comments

Enable documentation:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

Controller:

```csharp
/// <summary>
/// Gets a user by ID
/// </summary>
/// <param name="id">User ID</param>
/// <returns>User details</returns>
[ProducesResponseType(200)]
[ProducesResponseType(404)]
[HttpGet("users/{id}")]
public IActionResult GetUser(int id)
{
    return Ok();
}
```

Boom 💥 — Swagger now shows professional documentation.

---

# 4. Versioning: Why It’s Important

APIs are like laws:

> Once people depend on them, changing them breaks lives.

Versioning allows:
✅ Backward compatibility
✅ Safe updates
✅ No breaking existing apps

---

# 5. Versioning Strategies

---

## ✅ URL Versioning (Most Popular)

```http
/api/v1/users
/api/v2/users
```

Controller:

```csharp
[Route("api/v1/users")]
public class UsersV1Controller : ControllerBase { }
```

---

## ✅ Header Versioning

Client sends:

```
X-API-Version: 2
```

Controller:

```csharp
[ApiVersion("2.0")]
[Route("api/users")]
public class UsersController : ControllerBase { }
```

---

## ✅ Which One to Teach?

| Method         | Recommendation              |
| -------------- | --------------------------- |
| URL versioning | ✅ Best for beginners        |
| Header         | Good for enterprise systems |

---

# 6. Server-side Validations

Never trust:

* Forms
* Mobile apps
* Web apps
* Frontend devs 😄

Validation must happen on the server.

---

# 7. Data Annotations

### Example Model:

```csharp
public class RegisterModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [MinLength(6)]
    public string Password { get; set; }
}
```

Controller:

```csharp
if (!ModelState.IsValid)
    return BadRequest(ModelState);
```

---

### Common Annotations

| Attribute      | Purpose                 |
| -------------- | ----------------------- |
| [Required]     | Field must not be empty |
| [Range]        | Number boundaries       |
| [EmailAddress] | Email validation        |
| [StringLength] | Length control          |

---

# 8. Fluent Validation (Professional Validation)

Instead of decorating models, you write validation as rules.

---

## Example:

```bash
dotnet add package FluentValidation.AspNetCore
```

Validator:

```csharp
public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(x => x.Email).NotEmpty().EmailAddress();
        RuleFor(x => x.Password).MinimumLength(6);
    }
}
```

Register:

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<UserValidator>();
```

---

## ✅ Data Annotations vs FluentValidation

| Feature        | Data Annotations | Fluent |
| -------------- | ---------------- | ------ |
| Simple         | ✅                | ✅      |
| Complex logic  | ❌                | ✅      |
| Reusable rules | ❌                | ✅      |

---

# 9. Paging, Filtering & Sorting

APIs with 50,000 rows? Paging saves lives.

---

## Paging Example

```http
GET /users?page=1&pageSize=10
```

Controller:

```csharp
var data = users
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToList();
```

---

## Filtering

```http
GET /users?role=admin
```

```csharp
query = query.Where(x => x.Role == role);
```

---

## Sorting

```http
GET /users?sort=name
```

```csharp
query = query.OrderBy(x => x.Name);
```

---

# 10. Structured Logging

Bad log:

```csharp
_logger.LogInformation("User failed");
```

Good log:

```csharp
_logger.LogInformation("Login failed for user {UserId}", userId);
```

---

## Why structured logs?

Because logs become:
✅ Searchable
✅ Queryable
✅ Dashboard friendly

---

## Example using Serilog

```bash
dotnet add package Serilog.AspNetCore
```

```csharp
Log.Information("Order {OrderId} failed for {User}", orderId, username);
```

---

# 11. Git & GitHub Basics

---

## What is Git?

Git is a **time machine with trust issues**.
It tracks:

* Code changes
* Authors
* History
* Rollbacks

---

## Basic Commands

```bash
git init
git status
git add .
git commit -m "Initial commit"
```

---

## Push to GitHub

```bash
git remote add origin <url>
git push -u origin main
```

---

## Very Important Commands

| Command      | Meaning       |
| ------------ | ------------- |
| git pull     | Get latest    |
| git push     | Send changes  |
| git branch   | Show branches |
| git checkout | Switch        |
| git merge    | Combine       |

---

## GitHub Workflow (Students Should Know This):

1. Clone repo
2. Create branch
3. Commit
4. Push
5. Pull request
6. Review
7. Merge

---

# ✅ BONUS: Teaching Strategy Suggestion (For Your 8 Weeks)

| Week | Topic         |
| ---- | ------------- |
| 1    | Git + GitHub  |
| 2    | API Basics    |
| 3    | Swagger       |
| 4    | Validation    |
| 5    | Paging        |
| 6    | Logging       |
| 7    | Versioning    |
| 8    | Final Project |