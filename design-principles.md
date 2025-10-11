# Introduction to Design Principles

## Introduction
### How to think like a developer — not just code like one

Before building APIs, a software engineer must understand why we design software a certain way. This week focuses on the thinking behind clean, maintainable code — abstraction, separation of concerns, and SOLID design principles. We'll learn to organize their thoughts before organizing their folders.

## Why it Matters
### Theory
- The cost of messy code (hard to test, hard to change, easy to break).
- What is abstraction? → Hiding complexity and exposing intent.
- Examples of abstraction in real life (car pedals, TV remotes).
- How classes, interfaces, and methods help us model real-world concepts.

### Example
```csharp
// Without abstraction
public class Portfolio
{
    public void SaveToFile() { /* ... */ }
    public void SendEmailToUser() { /* ... */ }
    public void CalculatePortfolioScore() { /* ... */ }
}
```

```csharp
// With abstraction
public interface IPortfolioRepository
{
    void Save(Portfolio portfolio);
}

public class PortfolioService
{
    private readonly IPortfolioRepository _repo;

    public PortfolioService(IPortfolioRepository repo)
    {
        _repo = repo;
    }

    public void SubmitPortfolio(Portfolio portfolio)
    {
        _repo.Save(portfolio);
    }
}
```

## Separation of Concerns (SoC)
### Splitting code responsibilities so each class does one thing and does it well.
- Definition: Each component should have one reason to change.
- Analogy: In a restaurant — chef cooks, waiter serves, cashier collects money.
- Benefits: Reusability, testability, clarity.
- Introduction to layers — Controller, Service, Repository.

### Example
```csharp
// Controller handles request
public class ProfilesController : ControllerBase
{
    private readonly ProfileService _service;
    public ProfilesController(ProfileService service) => _service = service;

    [HttpPost]
    public IActionResult Create(ProfileDto dto)
    {
        _service.CreateProfile(dto);
        return Ok();
    }
}

// Service handles business logic
public class ProfileService
{
    private readonly ProfileRepository _repo;
    public ProfileService(ProfileRepository repo) => _repo = repo;

    public void CreateProfile(ProfileDto dto)
    {
        var profile = new Profile { Name = dto.Name };
        _repo.Add(profile);
    }
}

// Repository handles data
public class ProfileRepository
{
    public void Add(Profile profile) { /* EF or file storage */ }
}
```

## SOLID Principles Overview
### The 5 foundational principles that make software easy to maintain and extend.
- **S** – Single Responsibility Principle (SRP): One class = one job. (Your toaster doesn’t also make coffee.)
- **O** – Open/Closed Principle (OCP): Open for extension, closed for modification. (Add new features without breaking old ones.)
- **L** – Liskov Substitution Principle (LSP): Subclasses should behave like their parent class promises. (If it walks like a duck…)
- **I** – Interface Segregation Principle (ISP): Don’t force classes to depend on methods they don’t use. (Don’t make a Bird class implement IFly if it’s a penguin.)
- **D** – Dependency Inversion Principle (DIP): Depend on abstractions, not concretes. (Use IEmailService instead of new GmailService().)

### Example
```csharp
// Applying DIP and SRP
public interface INotificationService
{
    void Notify(string message);
}

public class EmailNotificationService : INotificationService
{
    public void Notify(string message) => Console.WriteLine($"Email: {message}");
}

public class PortfolioService
{
    private readonly INotificationService _notifier;

    public PortfolioService(INotificationService notifier)
    {
        _notifier = notifier;
    }

    public void SubmitPortfolio(Portfolio portfolio)
    {
        // business logic
        _notifier.Notify("Portfolio submitted successfully!");
    }
}
```

## KISS — Keep It Simple, Stupid
### Concept: 
KISS reminds us that simplicity beats cleverness. The simpler your design, the fewer bugs, misunderstandings.

```csharp
// ❌ Overengineered
public class MathService
{
    public Func<int, int, int> OperationFactory(string op)
    {
        return op switch
        {
            "add" => (a, b) => a + b,
            "sub" => (a, b) => a - b,
            _ => throw new InvalidOperationException("Unsupported op")
        };
    }
}

// ✅ KISS
public class MathService
{
    public int Add(int a, int b) => a + b;
}
```

## DRY — Don’t Repeat Yourself
### Concept:
Every piece of knowledge or logic in a system should live in one and only one place. Repetition leads to inconsistency — and inconsistency leads to bugs.

### Example
```csharp
// ❌ Violating DRY
public class ProjectService
{
    public decimal CalculateTotal(decimal amount, decimal tax) => amount + (amount * tax);
}

public class InvoiceService
{
    public decimal CalculateTotal(decimal amount, decimal tax) => amount + (amount * tax);
}
```

```csharp
// ✅ DRY
public static class CalculationHelper
{
    public static decimal CalculateTotal(decimal amount, decimal tax) => amount + (amount * tax);
}
```

## YAGNI — You Ain’t Gonna Need It
### Concept:
Don’t build features just because you might need them someday. Write only what’s needed right now. Future needs can be handled when they’re real, not imaginary.

### Example:
```csharp
// ❌ Violating YAGNI
public class ProfileService
{
    public void Create(ProfileDto dto)
    {
        // Planning ahead for future AI recommendations 🤦‍♂️
        var aiFeature = new AIPredictor();
        aiFeature.PreAnalyze(dto);

        // Just save the profile, dude.
    }
}
```

```csharp
// ✅ YAGNI
public class ProfileService
{
    public void Create(ProfileDto dto)
    {
        // Simple and necessary
        SaveToDatabase(dto);
    }
}
```