# **Dependency Injection: Wiring and Decoupling Layers**
---
**Dependency Injection** is a *design pattern* that means:

> “Instead of a class creating the things it needs (its dependencies), it gets them handed to it from the outside.”

---

### **Breaking it Down**

Imagine you’re a mechanic (the *class*).
You need tools (your *dependencies*) — like a wrench, screwdriver, etc.

* **Without DI**: you go buy tools yourself every time you fix a car.
* **With DI**: someone gives you a full toolbox when you arrive — you just use what’s provided.

That’s dependency injection.
You don’t worry about how the tools were made or stored — you just use them.

---

### **Without DI (Tightly Coupled Code)**

```csharp
public class ProfileService
{
    private readonly ProfileRepository _repo;

    public ProfileService()
    {
        _repo = new ProfileRepository(); // creates its own dependency
    }

    public void AddProfile(Profile profile)
    {
        _repo.Add(profile);
    }
}
```

Problems:

* Hard to replace `ProfileRepository` (what if we switch to MongoDB later?)
* Hard to test — can’t mock or substitute dependencies easily
* Violates **Dependency Inversion Principle** (from SOLID)

---

### **With DI (Loosely Coupled Code)**

```csharp
public class ProfileService
{
    private readonly IProfileRepository _repo;

    // Dependency is injected through the constructor
    public ProfileService(IProfileRepository repo)
    {
        _repo = repo;
    }

    public void AddProfile(Profile profile)
    {
        _repo.Add(profile);
    }
}
```

Now `ProfileService` doesn’t *create* or *control* the repository — it just *uses* it.
The actual repository (e.g. EF, mock, or in-memory) is supplied from the outside — typically by ASP.NET Core’s **dependency injection container**.

---

### **In ASP.NET Core**

You tell the framework what to inject where:

```csharp
builder.Services.AddScoped<IProfileRepository, EfProfileRepository>();
builder.Services.AddScoped<ProfileService>();
```

Now when the controller asks for a `ProfileService`, ASP.NET:

1. Creates a `ProfileService`
2. Automatically gives it an `EfProfileRepository`
3. Handles the lifecycle for you

```csharp
public class ProfilesController : ControllerBase
{
    private readonly ProfileService _service;

    public ProfilesController(ProfileService service)
    {
        _service = service; // injected automatically
    }
}
```

---

### 🧩 **Three Common Injection Methods**

1. **Constructor Injection** (most common)

   ```csharp
   public MyService(IMailer mailer) { ... }
   ```
2. **Property Injection**

   ```csharp
   public IMailer Mailer { get; set; }
   ```
3. **Method Injection**

   ```csharp
   public void SendEmail(IMailer mailer) { ... }
   ```

---

### **Why It Matters**

✅ Promotes **loose coupling**
✅ Makes code **testable**
✅ Simplifies **maintenance and refactoring**
✅ Enables **clean architecture** and **SOLID** design

---

### **In One Sentence**

> **Dependency Injection is about giving a class its tools, not making it build them.**
---

## **The Problem: Tight Coupling and Hardcoding Dependencies**
What *dependency* means, what tight coupling looks like, and why DI makes life easier.

---

### **What is a Dependency?**

A **dependency** is simply a class your class relies on.

```csharp
public class ProfileService
{
    private readonly ProfileRepository _repo = new ProfileRepository();
}
```

Here, `ProfileService` **depends** on `ProfileRepository`.

> “If your class can’t work without another class, that’s a dependency.”

---

### **The Problem with Tight Coupling**

* Code becomes **hard to test**
* Code becomes **hard to change**
* Code violates **Open/Closed** and **Dependency Inversion** principles

```csharp
// ❌ Tight coupling
public class ProfileService
{
    private readonly ProfileRepository _repo = new ProfileRepository();

    public void AddProfile(Profile profile)
    {
        _repo.Add(profile);
    }
}
```

Now if we want to replace `ProfileRepository` with a new EF version, we’ll have to edit the service.
That’s bad design.

---

### **Solution: Inversion of Control (IoC)**

We flip the responsibility — instead of the service creating its dependencies,
**we inject them from outside**.

> “Don’t bake your dependencies inside the cake — pass them as ingredients.”

---

### **Example:**

```csharp
public class ProfileService
{
    private readonly IProfileRepository _repo;
    public ProfileService(IProfileRepository repo)
    {
        _repo = repo;
    }

    public void AddProfile(Profile profile)
    {
        _repo.Add(profile);
    }
}
```

Now, `ProfileService` doesn’t care which repository implementation it gets — that’s dependency *inversion*.

---

## **Understanding Dependency Injection in ASP.NET Core**
ASP.NET Core automatically provides dependencies via its built-in DI container.

---

### **What is Dependency Injection (DI)?**

DI is a design pattern that **automatically provides dependencies** to a class at runtime, instead of creating them manually.

ASP.NET Core’s DI container:

* Creates objects for you
* Manages their lifetime
* Resolves all dependencies recursively

---

### **Registering Services**

#### In `Program.cs`:

```csharp
builder.Services.AddScoped<IProfileRepository, EfProfileRepository>();
builder.Services.AddScoped<ProfileService>();
```

#### In Controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProfilesController : ControllerBase
{
    private readonly ProfileService _service;

    public ProfilesController(ProfileService service)
    {
        _service = service;
    }

    [HttpPost]
    public IActionResult Create(ProfileDto dto)
    {
        _service.AddProfile(new Profile { Name = dto.Name });
        return Ok("Profile created!");
    }
}
```

ASP.NET Core automatically **injects** the right dependencies.

---

### **Dependency Lifetimes in ASP.NET Core**

Every time ASP.NET Core creates or resolves a service, it has to decide *how long* that instance should stay alive.

You configure this in your DI setup (usually in `Program.cs`):

```csharp
builder.Services.AddSingleton<ILogService, LogService>();
builder.Services.AddScoped<IProfileRepository, ProfileRepository>();
builder.Services.AddTransient<IEmailService, EmailService>();
```

---

### **Singleton** — *One instance for the entire application*

#### What it means

* Created **only once** — the very first time it’s requested.
* The *same instance* is reused **throughout the app’s lifetime** (until the app stops).
* Think of it as a *global object* managed safely by the DI container.

#### Analogy

> Imagine a **shared company printer** — every employee uses the same one. It’s set up once and stays running all day.

#### Example

```csharp
builder.Services.AddSingleton<ILogService, LogService>();
```

If 1,000 HTTP requests come in, they’ll all share **one** `LogService` instance.

#### When to Use

* Services that are **stateless** and **thread-safe**
  (e.g. logging, configuration readers, caching providers)

#### ⚠️ Avoid When

* The service holds **user-specific** or **request-specific** data — that would lead to data leaks or weird bugs.

---

### **Scoped** — *One instance per request*

#### What it means

* Created **once per HTTP request**.
* All parts of that request share the same instance.
* A new instance is created for the next request.

#### Analogy

> Think of it as a **shopping basket** at a supermarket — valid for *your* trip only. Once you check out, it’s cleared.

#### 💻 Example

```csharp
builder.Services.AddScoped<IProfileRepository, EfProfileRepository>();
```

If a controller and service both need `IProfileRepository`,
they’ll get **the same one** during the same HTTP request —
but a **different one** for the next request.

#### ✅ When to Use

* Database contexts (e.g. `DbContext`)
* Repositories or services tied to a **unit of work** pattern
* Anything that manages per-request operations

> ⚠️ Using `AddSingleton` with something like `DbContext` can cause database connection issues and concurrency nightmares.

---

### **Transient** — *A new instance every time*

#### 🔍 What it means

* Created **each time** it’s requested from the DI container.
* Lightweight, stateless services that don’t need to persist anything.

#### 🧠 Analogy

> Think of it like **disposable coffee cups** — used once and tossed away.

#### 💻 Example

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

If multiple controllers or classes need `IEmailService`, each gets **its own** fresh instance.

#### ✅ When to Use

* Short-lived, stateless, lightweight services (like email senders, formatters, helpers)
* When each consumer should have a completely separate copy

#### ⚠️ Avoid When

* The object is expensive to create (too many new objects = performance hit)

---

## 🧭 Summary Table

| Lifetime      | Created          | Shared              | Common Uses                           |
| ------------- | ---------------- | ------------------- | ------------------------------------- |
| **Singleton** | Once             | Across all requests | Logging, configuration, caching       |
| **Scoped**    | Once per request | Within same request | DbContext, repositories, unit of work |
| **Transient** | Every time       | Never shared        | Helpers, utilities, formatters        |

---

## 🧩 Example in the Portfolio API

```csharp
builder.Services.AddSingleton<ILoggerService, ConsoleLoggerService>();
builder.Services.AddScoped<IProfileRepository, EfProfileRepository>();
builder.Services.AddTransient<IEmailService, EmailService>();
```

* `ILoggerService` → one instance forever
* `EfProfileRepository` → one per HTTP request
* `EmailService` → new instance each time it’s needed

---

## 💬 Real-World Rule of Thumb

> “If you’re not sure — start with **Scoped**.
> Then move to **Singleton** if it’s stateless and thread-safe,
> or **Transient** if it’s lightweight and disposable.”

---

## **Applying Dependency Injection**

---

### **Wiring Layers Together**

```
API → Application → Infrastructure → Domain
```

Now we’ll *connect* those using DI.

#### Example Setup:

```csharp
// In Portfolio.Api (Program.cs)
builder.Services.AddScoped<IProfileRepository, EfProfileRepository>();
builder.Services.AddScoped<ProfileService>();
```

* `EfProfileRepository` is from **Infrastructure**
* `ProfileService` is from **Application**
* `IProfileRepository` interface connects them

---

### **Testing the Flow**

```csharp
// Controller
[HttpPost]
public IActionResult AddProfile(ProfileDto dto)
{
    _service.AddProfile(new Profile { Name = dto.Name });
    return Ok("Profile saved!");
}
```

* Controller gets `ProfileService`
* `ProfileService` gets `IProfileRepository`
* `EfProfileRepository` talks to database

That’s Clean Architecture — powered by DI.

---

### **Mocking with DI (Intro to Testing Concepts)**

DI makes unit testing easier:

```csharp
var mockRepo = new Mock<IProfileRepository>();
var service = new ProfileService(mockRepo.Object);
```

No need for real database — perfect for testing logic in isolation.

---

### **Common Mistakes to Watch For**

* Forgetting to register services in `Program.cs`
* Registering wrong lifetime (using Singleton for DbContext)
* Injecting concrete types instead of interfaces

---

### **Visual Recap**

```
Controller
   ↓
ProfileService
   ↓
IProfileRepository → EfProfileRepository
   ↓
Database
```

Each dependency injected at runtime. No hardcoded connections.

---