# **WEEK 10 — Basic Architectural Structures**
### *“Good code in the wrong place is still bad design.”*

Structure is *how to organize* growing project — how each part of the codebase should know its place and mind its business.

## **Understanding Software Architecture & Layered Design**
Understanding what software architecture is and why layering helps organize responsibilities and reduce chaos.

### **What is Software Architecture?**
Architecture is the **blueprint** of your software — it defines how the parts of your system talk to each other, who knows what, and who does what.
> “Think of it as city planning for your code — not every building needs to know what’s inside the others.”
**In simpler words:**
It’s the structure that helps a small project grow *without turning into a spaghetti bowl.*

---

### **Analogy:**

Using the **“Restaurant Model”**:

* **Customer (Controller):** takes the order (HTTP Request)
* **Waiter (Service Layer):** passes order to kitchen (Business Logic)
* **Chef (Repository/Database Layer):** prepares data (Database)
* **Menu (DTOs):** defines what customers can order (Data Contract)

Each has a clear role, no one does everything.

---

### **Layered Architecture Overview**

#### Common 3-Layer Pattern:

1. **Presentation (API Layer)** – Handles HTTP requests/responses.
2. **Application (Service Layer)** – Contains business logic.
3. **Infrastructure (Data Layer)** – Talks to databases or external services.

```csharp
// Controller (Presentation)
public class ProfilesController : ControllerBase
{
    private readonly ProfileService _service;
    public ProfilesController(ProfileService service) => _service = service;

    [HttpPost]
    public IActionResult Create(ProfileDto dto)
    {
        _service.Create(dto);
        return Ok();
    }
}

// Service (Application)
public class ProfileService
{
    private readonly IProfileRepository _repo;
    public ProfileService(IProfileRepository repo) => _repo = repo;

    public void Create(ProfileDto dto)
    {
        var profile = new Profile { Name = dto.Name };
        _repo.Add(profile);
    }
}

// Repository (Infrastructure)
public class ProfileRepository : IProfileRepository
{
    public void Add(Profile profile)
    {
        // Database code here
    }
}
```
---

## **Clean Architecture Principle & Dependency Flow**
Clean Architecture enhances layered design by emphasizing **dependency direction** and **loose coupling**.

---

### **What is Clean Architecture?**

Clean Architecture (by Uncle Bob Martin) emphasizes that:

* **Business rules (core logic)** should not depend on **frameworks, databases, or UI.**
* **Dependencies point inward**, not outward.

#### Visual:

```
[External Layer]  →  [Application Layer]  →  [Domain Layer]
```

* Outer layers depend on inner layers.
* Inner layers know nothing about the outside world.

> “Your domain should be pure — no EF, no HTTP, no JSON, no drama.”

---

### **Dependency Flow in Practice**

In your Portfolio API:

| Layer       | Knows About | Should Not Know About |
| ----------- | ----------- | --------------------- |
| API         | Application | Infrastructure        |
| Application | Domain      | Database, Frameworks  |
| Domain      | Nothing     | Everything else       |

```csharp
// Domain
public class Profile
{
    public string Name { get; set; }
}

// Application
public interface IProfileRepository
{
    void Add(Profile profile);
}

// Infrastructure
public class EfProfileRepository : IProfileRepository
{
    public void Add(Profile profile)
    {
        // EF logic here
    }
}
```

> “The Application layer defines *what* needs to happen; the Infrastructure layer decides *how* it happens.”

---

### **Loose Coupling**
Loose coupling means components are independent — changing one doesn’t break others.

**Example:**

```csharp
// Instead of this (tight coupling)
var repo = new ProfileRepository();

// Use dependency injection (loose coupling)
private readonly IProfileRepository _repo;
public ProfileService(IProfileRepository repo) => _repo = repo;
```
---

## **Connecting Layers with Interfaces & Applying Clean Architecture**
**interfaces** connect layers and how to structure a Clean Architecture project.

---

### **Interfaces as Contracts**

* **Interfaces** define what a service *can do*, not *how* it does it.
* They live in the **Application layer**, implemented by the **Infrastructure** layer.

```csharp
// Application Layer
public interface IProjectRepository
{
    void Add(Project project);
}

// Infrastructure Layer
public class EfProjectRepository : IProjectRepository
{
    public void Add(Project project)
    {
        // EF Core logic
    }
}
```
---

### **Project Structure Example**

```
src/
 ├── Portfolio.Api/             --> Presentation Layer
 ├── Portfolio.Application/     --> Business Logic (Interfaces, Services)
 ├── Portfolio.Domain/          --> Entities & Core Logic
 └── Portfolio.Infrastructure/  --> EF Core, Repositories
```

---

### **Applying Clean Architecture**

* Keep **Entities** (Profile, Project, Skill) in **Domain**.
* Keep **Interfaces & Services** in **Application**.
* Keep **EF Core Repositories** in **Infrastructure**.
* Keep **Controllers** in **API**.

#### Example Folder Crosswalk:

| Folder         | Contains                                 |
| -------------- | ---------------------------------------- |
| Domain         | Profile.cs, Project.cs                   |
| Application    | IProfileRepository.cs, ProfileService.cs |
| Infrastructure | EfProfileRepository.cs, DbContext.cs     |
| Api            | ProfilesController.cs                    |