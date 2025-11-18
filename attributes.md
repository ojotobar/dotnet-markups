Alright, let’s walk through each of these attributes nice and easy—like a seasoned craftsman showing apprentices how tools have been used since the old days. And of course, with a sprinkle of humor so nobody sleeps off in class.

Below is what each attribute does, *plain English-style*, and then a clean usage example you can show your students.

---

# 🛠️ **C# Data Annotation Attributes Explained (With Examples)**

These attributes are usually applied in **model classes** to enforce validation rules. They work with ASP.NET Core model binding and validation—when the client sends data, the framework checks these rules automatically before your code runs.

---

## **1. `[Required]`**

**What it does:**
Ensures the property **must have a value**. No value? Validation error.
Perfect for fields your ancestors would insist on filling before leaving the house.

**Example:**

```csharp
[Required]
public string Name { get; set; }
```

---

## **2. `[EmailAddress]`**

**What it does:**
Checks if the value looks like a proper email format.
Doesn’t guarantee it’s your real email—just that it isn’t “thomas@@broken...com”.

**Example:**

```csharp
[EmailAddress]
public string Email { get; set; }
```

---

## **3. `[StringLength]`**

**What it does:**
Controls the *maximum* (and optionally *minimum*) allowable length of a string.

**Example:**

```csharp
[StringLength(50, MinimumLength = 3)]
public string Username { get; set; }
```

---

## **4. `[Range]`**

**What it does:**
Enforces that the value falls within a specific range.
Useful when someone wants to set their age to "900" like a vampire.

**Example:**

```csharp
[Range(18, 65)]
public int Age { get; set; }
```

---

## **5. `[RegularExpression]`**

**What it does:**
Allows you to validate input using a custom regex pattern.
Warning: Regex may cause headaches if stared at too long.

**Example:**

```csharp
[RegularExpression(@"^[A-Za-z0-9_]+$")]
public string Username { get; set; }
```

> Only letters, numbers, and underscores allowed.

---

## **6. `[Compare]`**

**What it does:**
Used to confirm matching values—mostly for passwords.

**Example:**

```csharp
[Required]
public string Password { get; set; }

[Compare("Password")]
public string ConfirmPassword { get; set; }
```

---

## **7. `[MaxLength]`**

**What it does:**
Sets the maximum allowed length of a string or array.
Similar to StringLength, but typically used for database schema generation or API input limits.

**Example:**

```csharp
[MaxLength(200)]
public string Bio { get; set; }
```

---

## **8. `[MinLength]`**

**What it does:**
Requires the string or array to be at least a certain length.

**Example:**

```csharp
[MinLength(8)]
public string Password { get; set; }
```

---

## **9. `[Phone]`**

**What it does:**
Validates that the value looks like a phone number.
May still accept those “+234-HELLO-YOU” fancy strings, but at least it tries.

**Example:**

```csharp
[Phone]
public string MobileNumber { get; set; }
```

---

# 📘 **Full Usage Example Model**

Here’s a single model combining all the attributes—something you can show the class:

```csharp
public class RegisterUserDto
{
    [Required]
    public string FullName { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [StringLength(20, MinimumLength = 3)]
    public string Username { get; set; }

    [Range(18, 100)]
    public int Age { get; set; }

    [RegularExpression(@"^[A-Za-z0-9_]+$")]
    public string Nickname { get; set; }

    [Required]
    [MinLength(8)]
    public string Password { get; set; }

    [Compare("Password")]
    public string ConfirmPassword { get; set; }

    [MaxLength(200)]
    public string Bio { get; set; }

    [Phone]
    public string PhoneNumber { get; set; }
}
```