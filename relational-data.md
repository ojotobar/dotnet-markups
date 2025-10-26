## 🧩 Introduction to Entity Framework Core (EF Core)

### 1️⃣ What is Entity Framework Core?

Entity Framework Core (**EF Core**) is Microsoft’s modern **Object-Relational Mapper (ORM)** for .NET applications.
It allows developers to interact with a relational database using **C# classes** instead of raw SQL queries.

With EF Core, you can:

* Map .NET **classes** to **database tables**
* Use **LINQ** to query data
* Automatically **generate and update** the database schema using migrations
* Track changes and persist updates with `SaveChanges()`

In short — EF Core bridges the gap between your **object-oriented code** and the **relational world** of databases.

---

### 2️⃣ How EF Core Fits into the Application Architecture

In a layered or clean architecture, EF Core sits in the **Data Access Layer (DAL)** and acts as the communication bridge between your application and the database.

**Typical layering:**

```
Presentation Layer (Controllers / UI)
          ↓
Business Layer (Services)
          ↓
Data Access Layer (Repositories using EF Core)
          ↓
Database (SQL Server, PostgreSQL, etc.)
```

Each layer has its own responsibility:

* The **presentation layer** handles HTTP requests or UI events.
* The **business layer** implements logic and rules.
* The **data access layer** interacts with EF Core to query or persist data.

This separation makes your system **maintainable, testable, and scalable**.

---

### 3️⃣ Database Providers

EF Core supports multiple database systems using **providers**.
Each provider adapts EF Core to a specific database.

| Provider       | Package                                   | Notes                          |
| -------------- | ----------------------------------------- | ------------------------------ |
| **SQL Server** | `Microsoft.EntityFrameworkCore.SqlServer` | Default and most common        |
| **PostgreSQL** | `Npgsql.EntityFrameworkCore.PostgreSQL`   | Popular open-source choice     |
| **MySQL**      | `Pomelo.EntityFrameworkCore.MySql`        | Common for web apps            |
| **SQLite**     | `Microsoft.EntityFrameworkCore.Sqlite`    | Great for local or testing use |
| **Cosmos DB**  | `Microsoft.EntityFrameworkCore.Cosmos`    | For NoSQL database usage       |

You can easily swap databases by changing the provider and connection string — EF Core takes care of the rest.

---

### 4️⃣ Demo: Getting Started

#### 🧱 Step 1: Setup

1. Install EF Core packages using the .NET CLI:

   ```bash
   dotnet add package Microsoft.EntityFrameworkCore
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.Tools
   ```

2. Add a connection string to `appsettings.json`:

   ```json
   "ConnectionStrings": {
     "DefaultConnection": "Server=.;Database=KwikNestaDb;Trusted_Connection=True;TrustServerCertificate=True;"
   }
   ```

3. Register the DbContext in your `Program.cs`:

   ```csharp
   builder.Services.AddDbContext<AppDbContext>(options =>
       options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
   ```

---

#### 🧩 Step 2: Creating the DbContext

`DbContext` is the **heart of EF Core** — it represents a session with the database and provides access to entities through `DbSet<T>`.

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<Property> Properties { get; set; }
    public DbSet<Location> Locations { get; set; }
}
```

Each `DbSet<T>` corresponds to a table in the database, and each entity class (`Property`, `Location`, etc.) corresponds to a row definition.

---

#### 🏗️ Step 3: Creating Entity Models

Example entity:

```csharp
public class Property
{
    public Guid Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public ListingStatus Status { get; set; } = ListingStatus.Available;

    public Guid LocationId { get; set; }
    public Location Location { get; set; } = null!;
}
```

---

#### 🗃️ Step 4: Running Migrations

Migrations allow you to create or update your database schema automatically from your models.

Commands:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

These will:

* Generate a `Migrations` folder with SQL scripts.
* Create the database and tables in SQL Server.

---

### 5️⃣ CRUD Operations Using EF Core

EF Core provides built-in support for **Create, Read, Update, and Delete** operations.

#### ➕ Create

```csharp
var property = new Property { Title = "Luxury Villa", Price = 750000 };
_context.Properties.Add(property);
await _context.SaveChangesAsync();
```

#### 🔍 Read

```csharp
var allProperties = await _context.Properties.ToListAsync();
var singleProperty = await _context.Properties.FindAsync(propertyId);
```

#### ✏️ Update

```csharp
var prop = await _context.Properties.FindAsync(propertyId);
if (prop != null)
{
    prop.Price = 800000;
    _context.Properties.Update(prop);
    await _context.SaveChangesAsync();
}
```

#### ❌ Delete

```csharp
var prop = await _context.Properties.FindAsync(propertyId);
if (prop != null)
{
    _context.Properties.Remove(prop);
    await _context.SaveChangesAsync();
}
```

---

### 6️⃣ Relationships and Navigation Properties

In EF Core, **relationships** define how entities are connected:

* **One-to-One**
* **One-to-Many**
* **Many-to-Many**

#### 🏘️ Example: One-to-Many Relationship

Each `Location` can have many `Properties`.

```csharp
public class Location
{
    public Guid Id { get; set; }
    public string City { get; set; } = string.Empty;
    public List<Property> Properties { get; set; } = new();
}

public class Property
{
    public Guid Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public Guid LocationId { get; set; }
    public Location Location { get; set; } = null!;
}
```

EF Core automatically detects this relationship using conventions:

* `LocationId` is the **foreign key**.
* `Location` is the **navigation property**.

To load related data:

```csharp
var property = await _context.Properties
    .Include(p => p.Location)
    .FirstOrDefaultAsync(p => p.Id == id);
```

---

### 🧭 Key Takeaways

* EF Core helps bridge C# objects and database tables.
* It supports multiple databases via providers.
* `DbContext` is the entry point for all database operations.
* Migrations keep your database schema in sync with your code.
* You can manage relationships easily with navigation properties and LINQ queries.

---

## ⚙️ **Dapper**

### 1. Overview

Dapper is a **micro ORM** developed by the Stack Overflow team.
It sits between **raw ADO.NET** and **full ORMs like EF Core** — it maps SQL query results directly to C# objects but **you write the SQL yourself**.

It’s:

* Extremely **fast** and **lightweight**
* Best for **read-heavy applications**
* Requires **manual schema and relationship handling**

---

### 2. Key Concepts

| Concept               | Description                                               |
| --------------------- | --------------------------------------------------------- |
| **IDbConnection**     | Represents the database connection (e.g., SqlConnection). |
| **Dapper Extensions** | Adds helper methods for CRUD operations.                  |
| **Query & Execute**   | Core Dapper methods for retrieving and modifying data.    |

---

### 3. Dapper Code Example

#### Repository Interface

```csharp
public interface IDapperRepository
{
    Task<IEnumerable<T>> QueryAsync<T>(string sql, object? parameters = null);
    Task<T?> QuerySingleAsync<T>(string sql, object? parameters = null);
    Task<int> ExecuteAsync(string sql, object? parameters = null);
}
```

#### DapperRepository.cs

```csharp
using Dapper;
using System.Data;

public class DapperRepository : IDapperRepository
{
    private readonly IDbConnection _connection;

    public DapperRepository(IDbConnection connection)
    {
        _connection = connection;
    }

    public async Task<IEnumerable<T>> QueryAsync<T>(string sql, object? parameters = null)
        => await _connection.QueryAsync<T>(sql, parameters);

    public async Task<T?> QuerySingleAsync<T>(string sql, object? parameters = null)
        => await _connection.QuerySingleOrDefaultAsync<T>(sql, parameters);

    public async Task<int> ExecuteAsync(string sql, object? parameters = null)
        => await _connection.ExecuteAsync(sql, parameters);
}
```

---

### 4. Usage Example

#### Query

```csharp
var sql = "SELECT * FROM Properties WHERE Status = @status";
var properties = await _dapperRepository.QueryAsync<Property>(sql, new { status = "Approved" });
```

#### Insert

```csharp
var insertSql = "INSERT INTO Properties (Id, Title, Status) VALUES (@Id, @Title, @Status)";
await _dapperRepository.ExecuteAsync(insertSql, new
{
    Id = Guid.NewGuid(),
    Title = "New Apartment",
    Status = "Pending"
});
```

---

## ⚖️ EF Core vs Dapper

| Feature              | EF Core                 | Dapper                                |
| -------------------- | ----------------------- | ------------------------------------- |
| **Performance**      | Slightly slower         | Blazing fast                          |
| **Ease of Use**      | Easier for beginners    | Requires SQL knowledge                |
| **Control Over SQL** | Limited (generated SQL) | Full control                          |
| **Change Tracking**  | Yes                     | No                                    |
| **Migrations**       | Yes                     | No                                    |
| **Best For**         | CRUD-heavy apps         | Read-heavy apps, reporting, analytics |

---

## 🧩 **Pro Tip for Students**

Use **EF Core** for:

* Business apps with frequent data modifications
* Complex relationships
* Migrations and domain modeling

Use **Dapper** for:

* High-performance queries
* Read-only reporting
* Mixed use (EF for writes, Dapper for reads)