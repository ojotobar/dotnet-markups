**Manual JWT Authentication + Profile Management API (No Identity)**

---

## 🧩 Project Setup

1. **Create the project**

```bash
dotnet new webapi -n ProfileAuthApi
cd ProfileAuthApi
```

2. **Add packages**

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.InMemory
dotnet add package Microsoft.AspNetCore.Identity
```

We’ll use the `PasswordHasher<T>` from Identity just for secure hashing — no Identity framework itself.

---

## Step 1: User Model

```csharp
public class User
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Email { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public string FullName { get; set; } = string.Empty;
}
```

---

## ⚙️ Step 2: AppDbContext

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<User> Users => Set<User>();
}
```

---

## Step 3: Auth Settings

Add this to **`appsettings.json`**:

```json
"Jwt": {
  "Key": "super_secret_key_here_change_me",
  "Issuer": "ProfileAuthApi",
  "Audience": "ProfileAuthApiUsers",
  "ExpiresInMinutes": 60
}
```

---

## Step 4: AuthService

Handles registration, login, and JWT generation.

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Tokens;

public class JwtOptions
{
    public string Key { get; set; } = string.Empty;
    public string Issuer { get; set; } = string.Empty;
    public string Audience { get; set; } = string.Empty;
    public int ExpiresInMinutes { get; set; } = 60;
}

public interface IAuthService
{
    Task<string?> RegisterAsync(string email, string password, string fullName);
    Task<string?> LoginAsync(string email, string password);
}

public class AuthService : IAuthService
{
    private readonly AppDbContext _db;
    private readonly IPasswordHasher<User> _hasher;
    private readonly JwtOptions _jwt;

    public AuthService(AppDbContext db, IPasswordHasher<User> hasher, IOptions<JwtOptions> jwt)
    {
        _db = db;
        _hasher = hasher;
        _jwt = jwt.Value;
    }

    public async Task<string?> RegisterAsync(string email, string password, string fullName)
    {
        if (_db.Users.Any(u => u.Email == email))
            return null;

        var user = new User
        {
            Email = email,
            FullName = fullName
        };

        user.PasswordHash = _hasher.HashPassword(user, password);
        _db.Users.Add(user);
        await _db.SaveChangesAsync();

        return GenerateJwt(user);
    }

    public async Task<string?> LoginAsync(string email, string password)
    {
        var user = _db.Users.FirstOrDefault(u => u.Email == email);
        if (user == null)
            return null;

        var result = _hasher.VerifyHashedPassword(user, user.PasswordHash, password);
        if (result == PasswordVerificationResult.Failed)
            return null;

        return GenerateJwt(user);
    }

    private string GenerateJwt(User user)
    {
        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim("FullName", user.FullName)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwt.Key));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _jwt.Issuer,
            audience: _jwt.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwt.ExpiresInMinutes),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

---

## ⚙️ Step 5: Register Services

In **`Program.cs`**:

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Db
builder.Services.AddDbContext<AppDbContext>(opt => opt.UseInMemoryDatabase("ProfileAuthDb"));

// JWT Options
builder.Services.Configure<JwtOptions>(builder.Configuration.GetSection("Jwt"));
var jwt = builder.Configuration.GetSection("Jwt").Get<JwtOptions>();

// Auth
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<IPasswordHasher<User>, PasswordHasher<User>>();

// Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt =>
    {
        opt.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwt.Issuer,
            ValidAudience = jwt.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwt.Key))
        };
    });

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## ⚙️ Step 6: AuthController

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;

    public AuthController(IAuthService authService)
    {
        _authService = authService;
    }

    [HttpPost("register")]
    public async Task<IActionResult> Register([FromBody] RegisterRequest request)
    {
        var token = await _authService.RegisterAsync(request.Email, request.Password, request.FullName);
        if (token == null) return BadRequest("User already exists");
        return Ok(new { Token = token });
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var token = await _authService.LoginAsync(request.Email, request.Password);
        if (token == null) return Unauthorized("Invalid credentials");
        return Ok(new { Token = token });
    }

    [Authorize]
    [HttpGet("me")]
    public IActionResult GetCurrentUser()
    {
        var claims = User.Claims.ToDictionary(c => c.Type, c => c.Value);
        return Ok(new
        {
            Id = claims[ClaimTypes.NameIdentifier],
            Email = claims[JwtRegisteredClaimNames.Email],
            FullName = claims["FullName"]
        });
    }
}

public record RegisterRequest(string Email, string Password, string FullName);
public record LoginRequest(string Email, string Password);
```

---

## ✅ Test Flow

1. `POST /api/auth/register` → returns JWT.
2. `POST /api/auth/login` → returns JWT.
3. Use that token in Authorization header:
   `Authorization: Bearer <token>`
   Then call `GET /api/auth/me` → returns current user info.