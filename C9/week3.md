Perfect — thanks for the constraint.
**3 days a week, 2 hours per day** is actually a *good* teaching rhythm if you respect attention span and muscle memory. Old-school apprenticeships worked like this for a reason.

Below is a **full, in-depth teaching note** you can literally read from, adapt, or turn into slides.
This is written for **absolute beginners**, but without insulting their intelligence.

I’ll structure each day as:

* **Concepts**
* **Mental model**
* **Code**
* **Things to emphasize verbally**
* **Common beginner mistakes**

No structs. No clever tricks. Just foundations that don’t rot.

---

# WEEK PLAN (FOUNDATIONS OF OOP IN C#)

> **Guiding principle:**
> *Objects first. Abstractions later. The brain needs something concrete to hang ideas on.*

---

# DAY 1–2 (Mon + Wed)

## CLASSES & OBJECTS

*(This is the backbone — do not rush this)*

### 1. What a Class Models

#### Concept

A **class** is a **blueprint** that describes:

* What something **has** (data)
* What something **does** (behavior)

It does **not** represent a real thing by itself.
It represents the *idea* of a thing.

```text
Class  →  Blueprint
Object →  Actual thing built from the blueprint
```

#### Mental model

* A **class** is like a *house plan*
* An **object** is an *actual house built from that plan*
* You can build many houses from the same plan

#### Code

```csharp
class Car
{
    public string Brand;
    public int Speed;

    public void Drive()
    {
        Console.WriteLine("The car is driving");
    }
}
```

Explain:

* `Car` is a **type**
* It defines **data** (`Brand`, `Speed`)
* It defines **behavior** (`Drive()`)

---

### 2. Creating Objects (Instances)

#### Concept

An **object** is created using `new`.

```csharp
Car myCar = new Car();
```

* `myCar` is a **reference**
* The object lives in **memory**
* Multiple variables can point to different objects

#### Example

```csharp
Car car1 = new Car();
Car car2 = new Car();

car1.Brand = "Toyota";
car2.Brand = "Honda";
```

These are **two different cars**, even though they come from the same class.

#### Emphasize verbally

> “Changing `car1` does NOT change `car2`.
> Same class, different objects.”

---

### 3. Fields (State)

#### Concept

**Fields** store data inside an object.

```csharp
public string Name;
public int Age;
```

Fields represent:

* State
* Characteristics
* Memory

#### Example – User

```csharp
class User
{
    public string Username;
    public int Age;
}
```

Usage:

```csharp
User user = new User();
user.Username = "Toba";
user.Age = 30;
```

---

### 4. Methods (Behavior)

#### Concept

Methods describe **what the object can do**.

```csharp
public void Greet()
{
    Console.WriteLine("Hello!");
}
```

Methods:

* Operate on the object’s data
* Represent actions

#### Example

```csharp
class User
{
    public string Username;

    public void Greet()
    {
        Console.WriteLine($"Hello, my name is {Username}");
    }
}
```

Usage:

```csharp
User user = new User();
user.Username = "Ada";
user.Greet();
```

---

### 5. Simple Real-World Examples

#### Account

```csharp
class Account
{
    public decimal Balance;

    public void Deposit(decimal amount)
    {
        Balance += amount;
    }
}
```

#### Car

```csharp
class Car
{
    public int Speed;

    public void Accelerate()
    {
        Speed += 10;
    }
}
```

---

### Common Beginner Mistakes (Day 1–2)

🚫 Forgetting `new`
🚫 Thinking class == object
🚫 Trying to access fields without an instance
🚫 Putting everything in `Main`

---

### End-of-Day Wisdom

> **Objects come first.**
> If students don’t *feel* objects, everything later becomes memorization.

---

# DAY 3 (Friday)

## PROPERTIES (CONTROLLED ACCESS)

> This is where we teach **discipline**, not syntax.

---

### 1. Fields vs Properties

#### Fields (raw access)

```csharp
public int Age;
```

Problem:

* Anyone can set invalid values
* No control
* No rules

```csharp
user.Age = -50; // allowed 🤦‍♂️
```

---

### 2. Properties (Safe Access)

#### Concept

A **property** controls how data is read and written.

```csharp
public int Age { get; set; }
```

This looks simple, but it’s powerful:

* You can **validate**
* You can **block changes**
* You can **hide implementation**

---

### 3. Validation with Properties

```csharp
private int _age;

public int Age
{
    get { return _age; }
    set
    {
        if (value < 0)
            throw new Exception("Age cannot be negative");

        _age = value;
    }
}
```

Explain slowly:

* `_age` is the real storage
* `Age` is the gatekeeper
* Rules live here

---

### 4. Read-Only Properties

```csharp
public string Id { get; }
```

Set once:

```csharp
public User(string id)
{
    Id = id;
}
```

Why this matters:

* Protects identity
* Prevents accidental changes

---

### 5. Why Properties Exist (The WHY)

Properties exist to:

* Protect data
* Enforce rules
* Allow change without breaking code

Tell them this:

> “Fields are about storage.
> Properties are about **intent and safety**.”

---

### Common Beginner Mistakes (Day 3)

🚫 Thinking properties are “just syntax”
🚫 Making everything public
🚫 Skipping validation
🚫 Not understanding backing fields

---

# DAY 4 (Monday)

## INSTANCE VS STATIC

> This is where confusion lives. Go slow. Break things on purpose.

---

### 1. Instance Members

Belong to an **object**.

```csharp
class User
{
    public string Name;
}
```

Usage:

```csharp
User u = new User();
u.Name = "John";
```

You **need an object**.

---

### 2. Static Members

Belong to the **class itself**.

```csharp
class Config
{
    public static string AppName = "MyApp";
}
```

Usage:

```csharp
Console.WriteLine(Config.AppName);
```

No object needed.

---

### 3. Key Difference (Say This Clearly)

| Instance          | Static          |
| ----------------- | --------------- |
| Belongs to object | Belongs to type |
| Needs `new`       | No `new`        |
| Many copies       | One shared copy |

---

### 4. Real Examples

#### Math

```csharp
Math.Sqrt(16);
```

Why static?

* No need for a Math object
* Same behavior everywhere

---

#### Counter Example

```csharp
class Counter
{
    public static int Total;
}
```

Every object shares `Total`.

---

### 5. Break Things on Purpose

Show this error:

```csharp
static void Main()
{
    Name = "John"; // ❌
}
```

Explain:

* Static context cannot access instance members
* Why? No object exists

This lesson sticks.

---

### Common Beginner Mistakes (Day 4)

🚫 Trying to access instance members statically
🚫 Making everything static
🚫 Not understanding shared state

---

# DAY 5 (Wednesday)

## PRACTICE & REVIEW (MOST IMPORTANT DAY)

---

### Tiny Project: Bank Account

#### Step 1 – Start with Fields

```csharp
class BankAccount
{
    public string Owner;
    public decimal Balance;
}
```

---

### Step 2 – Add Methods

```csharp
public void Deposit(decimal amount)
{
    Balance += amount;
}
```

---

### Step 3 – Refactor Fields → Properties

```csharp
public string Owner { get; set; }

public decimal Balance { get; private set; }
```

---

### Step 4 – Add Validation

```csharp
public void Deposit(decimal amount)
{
    if (amount <= 0)
        throw new Exception("Invalid amount");

    Balance += amount;
}
```

---

### Step 5 – Introduce ONE Static Member

```csharp
public static int TotalAccounts;
```

Increment in constructor.

Explain:

* Why static makes sense here
* Why balance is NOT static

---

### Final Review Questions

* What is a class?
* What is an object?
* Why use properties?
* When should something be static?

If they can answer these, you’ve done your job.

---
