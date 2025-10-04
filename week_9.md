## 🧩 **What is an API?**

### 💡 Definition

An **API (Application Programming Interface)** is a **set of rules and tools** that allows one piece of software (like a web app, mobile app, or backend service) to **communicate** with another.

Think of it as a **messenger** between two systems — it takes a request, tells the system what you want to do, and returns the response back to you.

---

### 🖼 Real-world Analogy

Imagine a **restaurant** 🍽️:

* The **menu** is like an API specification — it tells you what you can order (available operations).
* You (the client) make an order (a request).
* The **waiter** (API) takes it to the kitchen (server).
* The **chef** prepares the food (processes your request).
* The waiter brings back the food (response).

You don’t go into the kitchen — you just use the menu and the waiter. That’s what APIs do between software systems.

---

### ⚙️ Technical Meaning

In programming, an API defines:

* **Endpoints (URLs)** where clients can send requests.
* **Methods (GET, POST, etc.)** that specify what action to perform.
* **Data formats** like JSON or XML for communication.

Example:

```bash
GET https://api.weather.com/v1/forecast?city=Lagos
```

Response (JSON):

```json
{
  "city": "Lagos",
  "temperature": 30,
  "condition": "Sunny"
}
```

Here:

* `GET` is the request method.
* The URL is the endpoint.
* The JSON is the response returned by the API.

---

## 🌐 **What is a Web API?**

A **Web API** is an API that can be accessed over the internet using the **HTTP protocol**.
For example:

* When you log into Instagram → the mobile app sends a request to Instagram’s **Web API**.
* When you check the weather → your app fetches data from a **Weather API**.

ASP.NET Web API is Microsoft’s framework for building **HTTP-based APIs** in C#.

---

## ☁️ **What is REST?**

### 💡 Definition

**REST** (Representational State Transfer) is an **architectural style** for designing networked APIs.

It defines a **set of principles** that make web services scalable, reliable, and easy to use.

---

### ⚖️ REST Principles

| Principle             | Description                                                                         |
| --------------------- | ----------------------------------------------------------------------------------- |
| **Client-Server**     | The client (frontend) and server (backend) are separate systems.                    |
| **Stateless**         | Each request is independent; the server doesn’t store client state.                 |
| **Cacheable**         | Responses can be cached to improve performance.                                     |
| **Uniform Interface** | Standard structure for communication — URLs represent resources.                    |
| **Layered System**    | APIs can have multiple layers (e.g., gateways, proxies) without the client knowing. |

---

### 🧱 Example of a RESTful API

| HTTP Method | URL               | Description             |
| ----------- | ----------------- | ----------------------- |
| **GET**     | `/api/products`   | Fetch all products      |
| **GET**     | `/api/products/1` | Fetch one product by ID |
| **POST**    | `/api/products`   | Add a new product       |
| **PUT**     | `/api/products/1` | Update a product        |
| **DELETE**  | `/api/products/1` | Remove a product        |

**Each URL** represents a **resource** (like a Product),
and **each HTTP verb** defines the **action** to perform.

---

### ⚙️ Example Request & Response

**Request**

```bash
GET https://api.example.com/products/2
```

**Response**

```json
{
  "id": 2,
  "name": "Wireless Mouse",
  "price": 25.99
}
```

**Rules followed:**

* Uses standard HTTP methods ✅
* Returns JSON ✅
* Resource-based URL ✅
* Stateless (no session tracking) ✅

---

### ⚙️ Summary Table

| Term        | Meaning                                     | Example                        |
| ----------- | ------------------------------------------- | ------------------------------ |
| **API**     | Interface for communication between systems | `GET /api/products`            |
| **Web API** | API accessed over the web using HTTP        | `https://api.github.com/users` |
| **REST**    | Architecture style for designing Web APIs   | RESTful endpoints for CRUD     |

---

# 🌐 HTTP Verbs & RESTful Standards

These are the **core building blocks** of how clients talk to servers in REST APIs.
When you build an **ASP.NET Web API**, you use these verbs (also called **HTTP methods**) to define what kind of action your endpoint performs.

---

## ⚙️ What Are HTTP Verbs?

An **HTTP Verb** tells the server **what action** you want to perform on a **resource** (a piece of data).

For example:

```bash
GET /api/products
```

means “Hey server, give me all the products.”

Each verb has a **specific purpose**, as shown below 👇

---

### 🧩 1. **GET**

Retrieve (read) data from the server.

#### Example:

```bash
GET /api/products
```

Response:

```json
[
  { "id": 1, "name": "Laptop" },
  { "id": 2, "name": "Mouse" }
]
```

**Usage:** Fetch lists or single items.
**Should not:** Change any data on the server (read-only).

---

### 🧩 2. **POST**

Create a new resource on the server.

#### Example:

```bash
POST /api/products
Content-Type: application/json

{
  "name": "Keyboard",
  "price": 45.00
}
```

Response:

```json
{
  "id": 3,
  "name": "Keyboard",
  "price": 45.00
}
```

**Usage:** For new records.
**Returns:** Usually a `201 Created` status and the new object.

---

### 🧩 3. **PUT**

Replace (update) an existing resource completely.

#### Example:

```bash
PUT /api/products/3
Content-Type: application/json

{
  "id": 3,
  "name": "Wireless Keyboard",
  "price": 55.00
}
```

**Usage:** Full updates (replace the old object).
**Returns:** Usually `200 OK` or `204 No Content`.

---

### 🧩 4. **PATCH**

Partially update an existing resource (only some fields).

#### Example:

```bash
PATCH /api/products/3
Content-Type: application/json

{
  "price": 50.00
}
```

**Usage:** Update just one or a few properties.
**Returns:** Updated object or `204 No Content`.

---

### 🧩 5. **DELETE**

Remove a resource.

#### Example:

```bash
DELETE /api/products/3
```

Response:

```
204 No Content
```

**Usage:** Permanently deletes the resource.
**Returns:** Usually `204 No Content` (no body).

---

## 🧠 Common HTTP Status Codes

| Code                      | Meaning                   | When to Use          |
| ------------------------- | ------------------------- | -------------------- |
| 200 OK                    | Success                   | GET, PUT, PATCH      |
| 201 Created               | Resource created          | POST                 |
| 204 No Content            | Success with no data      | DELETE               |
| 400 Bad Request           | Invalid input             | Missing/invalid data |
| 401 Unauthorized          | No or invalid credentials | JWT/Token issues     |
| 404 Not Found             | Resource doesn’t exist    | Wrong ID             |
| 500 Internal Server Error | Server crashed            | Unhandled exception  |

---

## 🧱 RESTful Standards and Conventions

To make APIs **consistent and predictable**, REST defines some **best practices**:

| Rule                                   | Description                                        | Example                                |
| -------------------------------------- | -------------------------------------------------- | -------------------------------------- |
| **Use Nouns, not Verbs in URLs**       | URL should represent a *resource*, not an *action* | ✅ `/api/products` ❌ `/api/getProducts` |
| **Use HTTP verbs for actions**         | Let verbs describe the operation                   | `GET /api/users`                       |
| **Use plural nouns**                   | Collections should be plural                       | `/api/products`, `/api/orders`         |
| **Use proper status codes**            | Don’t always return 200                            | Use 201, 404, 400 when appropriate     |
| **Stateless requests**                 | Each request must carry all info needed            | Include token or ID in every call      |
| **Support filtering, paging, sorting** | Use query strings                                  | `/api/products?sort=price&limit=10`    |

---

## 🧩 Example RESTful API Design

| Action           | HTTP Method | Endpoint          | Description |
| ---------------- | ----------- | ----------------- | ----------- |
| Get all products | GET         | `/api/products`   | Fetch all   |
| Get one product  | GET         | `/api/products/1` | Fetch by ID |
| Add new product  | POST        | `/api/products`   | Create      |
| Update product   | PUT         | `/api/products/1` | Replace     |
| Delete product   | DELETE      | `/api/products/1` | Remove      |

---

## 🧪 Mini Practice

Try designing your own REST API for **Students**:

| Operation              | HTTP Verb | URL |
| ---------------------- | --------- | --- |
| Get all students       | ?         | ?   |
| Add a student          | ?         | ?   |
| Get student by ID      | ?         | ?   |
| Update student details | ?         | ?   |
| Delete student         | ?         | ?   |

*(Try to fill in the question marks — this helps you remember the structure!)*

Perfect 👏 — let’s go hands-on!
You’re now going to **create your first ASP.NET Core Web API project** from scratch.
We’ll go step by step so you understand *not just what to do*, but *why* it’s done.

---

# 🧩 LESSON 3: Create a Web API Project (ASP.NET Core)
---

## 🧰 **Tools You’ll Need**

| Tool                                | Purpose                              |
| ----------------------------------- | ------------------------------------ |
| ✅ .NET SDK 6 or later               | Framework to build your API          |
| ✅ Visual Studio 2022 **or** VS Code | Code editor                          |
| ✅ Postman                           | For testing endpoints                |
| ✅ Swagger UI                        | Auto-generated API tester (built-in) |

---

## ⚙️ Step 1: Verify .NET Installation

Open your terminal (Command Prompt, PowerShell, or VS Code terminal):

```bash
dotnet --version
```

If you see a version number (e.g., `8.0.100`), you’re ready ✅
If not → install from: [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)

---

## ⚙️ Step 2: Create a New Web API Project

In your terminal:

```bash
dotnet new webapi -n ProductsApi
cd ProductsApi
```

✅ This command does 3 things:

1. Creates a new folder named **ProductsApi**
2. Sets up all necessary files and configurations
3. Includes a demo controller (`WeatherForecastController.cs`)

---

## 📁 Step 3: Explore the Folder Structure

Once created, open the folder in **Visual Studio** or **VS Code**.

Here’s the typical structure:

```
ProductsApi/
├── Controllers/
│   └── WeatherForecastController.cs
├── Properties/
│   └── launchSettings.json
├── appsettings.json
├── Program.cs
├── ProductsApi.csproj
```

### 🔍 Key Files:

| File                  | Purpose                                                  |
| --------------------- | -------------------------------------------------------- |
| `Program.cs`          | Entry point of the API, configures services & middleware |
| `appsettings.json`    | Stores configuration (DB strings, logging, etc.)         |
| `Controllers/`        | Where you define API endpoints                           |
| `launchSettings.json` | Development server settings                              |

---

## ⚙️ Step 4: Run the Project

In the terminal:

```bash
dotnet run
```

You’ll see something like:

```
Now listening on: https://localhost:7061
Now listening on: http://localhost:5061
```

Open the **HTTPS** link in your browser →
you’ll see **Swagger UI** with a default `/WeatherForecast` endpoint 🎉

---

## 🌐 Step 5: Test the Default Endpoint

Click **GET /WeatherForecast** → **Try it out** → **Execute**
You’ll see a JSON response like:

```json
[
  {
    "date": "2025-10-04",
    "temperatureC": 23,
    "temperatureF": 73,
    "summary": "Mild"
  }
]
```

✅ Congrats! You just built and ran your first Web API.

---

## ⚙️ Step 6: Understanding `Program.cs`

Open `Program.cs` — it might look like this:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services (like controllers, logging, etc.)
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers(); // Maps endpoints to controllers

app.Run();
```

### 🔍 What’s happening here:

| Line                    | Purpose                               |
| ----------------------- | ------------------------------------- |
| `AddControllers()`      | Adds MVC controller support           |
| `AddSwaggerGen()`       | Enables Swagger docs                  |
| `UseSwaggerUI()`        | Enables visual Swagger testing tool   |
| `UseHttpsRedirection()` | Redirects HTTP → HTTPS                |
| `MapControllers()`      | Maps routes from `[Route]` attributes |
| `app.Run()`             | Starts the server                     |

---

## ⚙️ Step 7: Add Your Own Controller

Let’s replace the default one with your own.

Create a file:
`Controllers/ProductsController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace ProductsApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        [HttpGet]
        public IActionResult GetProducts()
        {
            var products = new[]
            {
                new { Id = 1, Name = "Laptop", Price = 1500 },
                new { Id = 2, Name = "Mouse", Price = 50 }
            };
            return Ok(products);
        }
    }
}
```

---

## ⚙️ Step 8: Run and Test

Run again:

```bash
dotnet run
```

Go to:
👉 `https://localhost:7061/swagger`
You’ll now see a **ProductsController** with a **GET /api/products** endpoint.

Click **Try it out** → **Execute** → you should see:

```json
[
  { "id": 1, "name": "Laptop", "price": 1500 },
  { "id": 2, "name": "Mouse", "price": 50 }
]
```

Boom 💥 — you’ve just created your first real API endpoint.

---

## 🧠 What You’ve Learned

✅ How to create and run a Web API project
✅ How to navigate the folder structure
✅ What `Program.cs` does
✅ How to define and test a controller
✅ How to test with Swagger

---

## 🧪 Challenge Task

Try extending your `ProductsController`:

1. Add a new endpoint that returns **a single product by ID**:

   ```csharp
   [HttpGet("{id}")]
   public IActionResult GetProduct(int id)
   {
       var product = new { Id = id, Name = "Keyboard", Price = 70 };
       return Ok(product);
   }
   ```

2. Test it:
   `GET /api/products/1`

3. Bonus: Add `Console.WriteLine("Fetching product...")` to simulate logging.

---

## 🔹 1. Routing Overview

**Routing** in ASP.NET Core Web API determines how incoming HTTP requests are mapped to controller actions.

When a request comes in (like `GET /api/products`), the routing system matches that URL to a specific **controller** and **action method**.

### There are two main types of routing:

1. **Attribute Routing** (recommended)
2. **Convention-based Routing** (used in older ASP.NET versions)

#### Example of Attribute Routing:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAllProducts()
    {
        return Ok(new[] { "Laptop", "Phone", "Tablet" });
    }
}
```

➡️ The `Route("api/[controller]")` means:

* `[controller]` is replaced by the controller name without “Controller”.
* So `ProductsController` becomes `/api/products`.

➡️ The `[HttpGet]` tells ASP.NET Core that this method handles **GET** requests to `/api/products`.

---

## 🔹 2. Controllers

A **Controller** is a class that handles incoming HTTP requests, processes them, and returns a response.

### Key facts:

* Must inherit from `ControllerBase` or `Controller`.
* Contains **Action Methods** for each route.
* Decorated with `[ApiController]` to enable:

  * Automatic model validation
  * Automatic binding from request data
  * Consistent response formatting

#### Example:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts()
    {
        return Ok(new[] { "Mouse", "Keyboard", "Monitor" });
    }
}
```

---

## 🔹 3. Actions

An **Action** is a method inside a controller that responds to a specific HTTP request.

You can use attributes to define the HTTP method type:

| Attribute      | HTTP Verb | Description    |
| -------------- | --------- | -------------- |
| `[HttpGet]`    | GET       | Retrieve data  |
| `[HttpPost]`   | POST      | Create data    |
| `[HttpPut]`    | PUT       | Update data    |
| `[HttpDelete]` | DELETE    | Delete data    |
| `[HttpPatch]`  | PATCH     | Partial update |

#### Example:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(new[] { "TV", "Radio" });

    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok($"Product {id}");

    [HttpPost]
    public IActionResult Create([FromBody] string name) => Ok($"Created {name}");

    [HttpDelete("{id}")]
    public IActionResult Delete(int id) => Ok($"Deleted {id}");
}
```

✅ This controller supports multiple endpoints:

* `GET /api/products`
* `GET /api/products/1`
* `POST /api/products`
* `DELETE /api/products/1`

---

## 🔹 4. Quick Routing Flow Summary

1. Client sends a request → `GET /api/products/1`
2. ASP.NET Core routing checks for a matching controller + action.
3. The method `GetById(int id)` is matched.
4. The framework automatically passes `1` as the `id` parameter.
5. The action executes and returns a response.

---

## 🧠 Practice Exercise

**Goal:** Create your own `CustomersController`.

Steps:

1. Add a new controller:

   ```bash
   dotnet new controller -name CustomersController -output Controllers
   ```
2. Add routes:

   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class CustomersController : ControllerBase
   {
       [HttpGet]
       public IActionResult GetAll() => Ok(new[] { "John", "Jane" });

       [HttpGet("{id}")]
       public IActionResult GetById(int id) => Ok($"Customer {id}");
   }
   ```
3. Run the app and test in **Swagger** or **Postman**.


## 🔹 1. What Happens When You Return Data

When your Web API returns a result (like a string, object, or list), ASP.NET Core automatically **serializes** it into **JSON** (JavaScript Object Notation) — which is the standard format for web APIs.

Example:

```csharp
[HttpGet]
public IEnumerable<string> Get()
{
    return new[] { "Laptop", "Phone", "Tablet" };
}
```

✅ Response:

```json
[
  "Laptop",
  "Phone",
  "Tablet"
]
```

ASP.NET Core automatically sets:

```
Content-Type: application/json
Status Code: 200 OK
```

---

## 🔹 2. Common Return Types

### Option 1: `IActionResult`

* Gives **maximum flexibility**.
* Lets you return different HTTP responses (`Ok()`, `BadRequest()`, etc.).

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    if (id <= 0)
        return BadRequest("Invalid ID");

    var product = new { Id = id, Name = "Laptop" };
    return Ok(product);
}
```

✅ Output when successful:

```json
{
  "id": 1,
  "name": "Laptop"
}
```

❌ Output when invalid:

```json
"Invalid ID"
```

---

### Option 2: `ActionResult<T>`

* Combines the clarity of returning a **typed object** (`T`) with the flexibility of `IActionResult`.
* Recommended for modern ASP.NET Core APIs.

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetById(int id)
{
    if (id <= 0)
        return BadRequest("Invalid ID");

    var product = new Product { Id = id, Name = "Mouse" };
    return Ok(product);
}
```

This allows:

* `return product;` → returns JSON automatically (`200 OK`)
* `return NotFound();` → returns `404 Not Found`
* `return BadRequest();` → returns `400 Bad Request`

---

### Option 3: Plain object (e.g. `Product`)

If you simply return a POCO (Plain Old CLR Object), ASP.NET Core still converts it to JSON — but you lose control over status codes.

```csharp
[HttpGet]
public Product Get()
{
    return new Product { Id = 1, Name = "Monitor" };
}
```

✅ Always returns `200 OK`.

❌ You can’t easily return other codes like `404` or `400`.

---

## 🔹 3. Built-in Response Helpers

These methods come from `ControllerBase`:

| Method                 | HTTP Code | Usage                    |
| ---------------------- | --------- | ------------------------ |
| `Ok(object)`           | 200       | Success                  |
| `Created(uri, object)` | 201       | When creating a resource |
| `NoContent()`          | 204       | Success, no data         |
| `BadRequest(object)`   | 400       | Invalid input            |
| `Unauthorized()`       | 401       | Missing/invalid auth     |
| `Forbidden()`          | 403       | Access denied            |
| `NotFound(object)`     | 404       | Resource not found       |
| `Conflict(object)`     | 409       | Data conflict            |
| `StatusCode(int)`      | Custom    | Custom codes             |

Example:

```csharp
[HttpPost]
public IActionResult Create([FromBody] Product product)
{
    if (string.IsNullOrEmpty(product.Name))
        return BadRequest("Product name required");

    return Created($"/api/products/{product.Id}", product);
}
```

✅ Response:

```json
{
  "id": 5,
  "name": "Headphones"
}
```

Headers include:

```
Location: /api/products/5
Status Code: 201 Created
```

---

## 🔹 4. JSON Serialization

ASP.NET Core uses **System.Text.Json** by default to convert your C# objects into JSON.

You can customize it in `Program.cs`:

```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = null; // Keep PascalCase
    });
```

---

## 🔹 5. Practice: Try It Yourself 💪

Create this controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<Book>> GetAll()
    {
        var books = new[]
        {
            new Book { Id = 1, Title = "C# in Depth" },
            new Book { Id = 2, Title = "Clean Code" }
        };
        return Ok(books);
    }

    [HttpGet("{id}")]
    public ActionResult<Book> GetById(int id)
    {
        if (id <= 0)
            return BadRequest("Invalid ID");

        var book = new Book { Id = id, Title = "Design Patterns" };
        return Ok(book);
    }
}

public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
}
```

Test in **Swagger** or **Postman** — you’ll see neat JSON responses with correct status codes.
---

## 🔹 1. What Are Minimal APIs?

**Minimal APIs** are a lightweight way to build APIs in ASP.NET Core (introduced in .NET 6).

Instead of creating controllers, routes, and classes, you define endpoints **directly in `Program.cs`**.

They’re ideal for:

* Small or microservices
* Internal APIs
* Prototypes or demos
* Simpler backends (no MVC or Razor needed)

---

### 🧩 Example — Minimal API

**Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/api/products", () =>
{
    var products = new[] { "Laptop", "Phone", "Tablet" };
    return Results.Ok(products);
});

app.MapGet("/api/products/{id}", (int id) =>
{
    return Results.Ok($"Product {id}");
});

app.MapPost("/api/products", (Product product) =>
{
    return Results.Created($"/api/products/{product.Id}", product);
});

app.Run();

record Product(int Id, string Name);
```

✅ Output when calling `/api/products`:

```json
["Laptop", "Phone", "Tablet"]
```

---

## 🔹 2. What Are Controller-based APIs?

This is the **traditional** ASP.NET Web API approach:

* You create `Controller` classes.
* You organize code by **controllers** and **actions**.
* You use attributes like `[HttpGet]`, `[HttpPost]`, `[Route]`.

### 🧩 Example — Controller-based API

**Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();

app.MapControllers();
app.Run();
```

**Controllers/ProductsController.cs**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll()
        => Ok(new[] { "Laptop", "Phone", "Tablet" });

    [HttpPost]
    public IActionResult Create(Product product)
        => Created($"/api/products/{product.Id}", product);
}

public record Product(int Id, string Name);
```

---

## 🔹 3. Key Differences: Minimal APIs vs Controllers

| Feature              | **Minimal API**                | **Controller-based API**         |
| -------------------- | ------------------------------ | -------------------------------- |
| Structure            | Defined inline in `Program.cs` | Organized in controller classes  |
| Boilerplate          | Minimal (few lines)            | More setup (attributes, classes) |
| Flexibility          | Great for small services       | Better for large apps            |
| Filters & Middleware | Must be added manually         | Supported natively               |
| Model Binding        | Manual or lightweight          | Automatic                        |
| Validation           | Must be implemented manually   | Built-in via `[ApiController]`   |
| Swagger Integration  | Works, but needs extra setup   | Works automatically              |
| Versioning           | Manual setup                   | Supported via libraries          |

---

## 🔹 4. Which Should You Use?

| Use **Minimal APIs** when:                   |
| -------------------------------------------- |
| You’re building a microservice               |
| You need simple endpoints quickly            |
| You don’t need complex validation or filters |
| You prefer functional programming style      |

| Use **Controllers** when:                                      |
| -------------------------------------------------------------- |
| You’re building a full REST API                                |
| You need clear structure and separation of concerns            |
| You need model validation, filters, dependency injection, etc. |
| You plan to grow the project over time                         |

---

## 🔹 5. Practice: Build the Same Endpoint Both Ways

Try both approaches below — you’ll see the difference clearly.

**Minimal API:**

```csharp
app.MapGet("/api/customers", () => new[] { "John", "Jane" });
```

**Controller:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(new[] { "John", "Jane" });
}
```

---

✅ **Summary:**

* **Minimal APIs** → quick, concise, low overhead.
* **Controllers** → powerful, structured, scalable.

---

# 🧩 Practice Project — **Products Web API**

Goal: Build a simple, well-structured API with logging, CRUD endpoints, and JSON responses.

---

## 🧱 1. Project Setup

Create a new project:

```bash
dotnet new webapi -n ProductsApi
cd ProductsApi
```

Run it:

```bash
dotnet run
```

✅ Visit `https://localhost:5001/swagger` — you should see the default Swagger UI.

---

## 🧩 2. Define the Model

Inside the `Models` folder (create one if it doesn’t exist):

**Models/Product.cs**

```csharp
namespace ProductsApi.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = default!;
        public decimal Price { get; set; }
    }
}
```

---

## 🧩 3. Create a Simple Repository (In-Memory)

Inside a new `Services` folder:

**Services/ProductService.cs**

```csharp
using ProductsApi.Models;

namespace ProductsApi.Services
{
    public class ProductService
    {
        private readonly List<Product> _products = new()
        {
            new Product { Id = 1, Name = "Laptop", Price = 1200 },
            new Product { Id = 2, Name = "Mouse", Price = 25 }
        };

        public IEnumerable<Product> GetAll() => _products;

        public Product? GetById(int id) => _products.FirstOrDefault(p => p.Id == id);

        public Product Add(Product product)
        {
            product.Id = _products.Max(p => p.Id) + 1;
            _products.Add(product);
            return product;
        }

        public bool Delete(int id)
        {
            var product = GetById(id);
            if (product == null) return false;
            _products.Remove(product);
            return true;
        }
    }
}
```

---

## 🧩 4. Register Service in DI Container

**Program.cs**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// ✅ Add ProductService to dependency injection
builder.Services.AddSingleton<ProductsApi.Services.ProductService>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

## 🧩 5. Create Controller with Logging

**Controllers/ProductsController.cs**

```csharp
using Microsoft.AspNetCore.Mvc;
using ProductsApi.Models;
using ProductsApi.Services;

namespace ProductsApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly ProductService _service;
        private readonly ILogger<ProductsController> _logger;

        public ProductsController(ProductService service, ILogger<ProductsController> logger)
        {
            _service = service;
            _logger = logger;
        }

        [HttpGet]
        public ActionResult<IEnumerable<Product>> GetAll()
        {
            _logger.LogInformation("Fetching all products");
            return Ok(_service.GetAll());
        }

        [HttpGet("{id}")]
        public ActionResult<Product> GetById(int id)
        {
            _logger.LogInformation("Fetching product with ID {Id}", id);
            var product = _service.GetById(id);

            if (product == null)
            {
                _logger.LogWarning("Product with ID {Id} not found", id);
                return NotFound($"Product {id} not found");
            }

            return Ok(product);
        }

        [HttpPost]
        public ActionResult<Product> Create(Product product)
        {
            _logger.LogInformation("Creating new product: {@Product}", product);
            var newProduct = _service.Add(product);
            return CreatedAtAction(nameof(GetById), new { id = newProduct.Id }, newProduct);
        }

        [HttpDelete("{id}")]
        public IActionResult Delete(int id)
        {
            _logger.LogInformation("Deleting product with ID {Id}", id);
            var deleted = _service.Delete(id);

            if (!deleted)
            {
                _logger.LogWarning("Attempted to delete non-existing product ID {Id}", id);
                return NotFound($"Product {id} not found");
            }

            return NoContent();
        }
    }
}
```

---

## 🧩 6. How Logging Works Here

ASP.NET Core provides a built-in `ILogger<T>` service that automatically logs to the console by default.

You can log:

* `LogInformation()` for normal operations
* `LogWarning()` for potential issues
* `LogError()` for exceptions

Example console output:

```
info: ProductsApi.Controllers.ProductsController[0]
      Fetching all products
warn: ProductsApi.Controllers.ProductsController[0]
      Product with ID 5 not found
```

---

## 🧩 7. Test in Swagger

Run the app:

```bash
dotnet run
```

Open:

```
https://localhost:5001/swagger
```

Try these endpoints:

* `GET /api/products` → returns list
* `GET /api/products/1` → returns single product
* `POST /api/products` → create new
* `DELETE /api/products/1` → delete product

Watch your terminal to see the **logging in action**.

---

## 🧠 Optional Enhancements (Try later)

* Add `PUT` to update a product
* Implement proper exception handling middleware
* Replace in-memory list with EF Core + database
* Add validation with `[Required]` attributes