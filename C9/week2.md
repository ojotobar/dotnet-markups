Got it! Let’s make a **full beginner-friendly lesson guide** for the topics in your image: **If/Else, Switch, Loops, Basic Algorithms**. I’ll break it down clearly with examples and explanations.

---

# **Lesson Guide: Programming Fundamentals**

---

## **1️⃣ If / Else Statements**

### **Purpose:**

* Make decisions in code based on conditions.
* Execute different code depending on whether a condition is true or false.

### **Syntax Example (C# / Java / JavaScript-like):**

```csharp
int number = 10;

if (number > 0)
{
    Console.WriteLine("Positive number");
}
else if (number == 0)
{
    Console.WriteLine("Zero");
}
else
{
    Console.WriteLine("Negative number");
}
```

### **Key Points:**

* `if` checks a condition.
* `else if` checks another condition if the first is false.
* `else` executes if none of the above are true.
* Conditions must evaluate to **true** or **false**.

---

## **2️⃣ Switch Statements**

### **Purpose:**

* Simplifies multiple `if/else if` checks when comparing the same variable to different values.
* Often cleaner for **menu options** or **enumerated values**.

### **Syntax Example:**

```csharp
int day = 3;

switch(day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
        Console.WriteLine("Wednesday");
        break;
    default:
        Console.WriteLine("Other day");
        break;
}
```

### **Key Points:**

* `case` defines a value to match.
* `break` prevents “fall-through” to next case (required in most languages).
* `default` runs if no cases match.

---

## **3️⃣ Loops**

### **Purpose:**

* Repeat code multiple times.
* Useful for iterating over arrays, lists, or performing repeated tasks.

### **Types of Loops:**

#### a) **For Loop**

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine("Iteration " + i);
}
```

* Runs a fixed number of times.
* `i++` increases counter each iteration.

#### b) **While Loop**

```csharp
int i = 0;
while (i < 5)
{
    Console.WriteLine("Iteration " + i);
    i++;
}
```

* Repeats **while condition is true**.
* Useful when number of iterations is unknown.

#### c) **Do-While Loop**

```csharp
int i = 0;
do
{
    Console.WriteLine("Iteration " + i);
    i++;
} while (i < 5);
```

* Executes **at least once** before checking condition.

---

## **4️⃣ Basic Algorithms**

### **What is an Algorithm?**

* Step-by-step instructions to solve a problem.
* Think of it as a **recipe** for the computer.

### **Common Examples:**

#### a) **Finding the Largest Number in an Array**

```csharp
int[] numbers = {3, 7, 2, 9, 5};
int max = numbers[0];

for (int i = 1; i < numbers.Length; i++)
{
    if (numbers[i] > max)
        max = numbers[i];
}

Console.WriteLine("Largest number is: " + max);
```

#### b) **Sum of Numbers from 1 to N**

```csharp
int n = 10;
int sum = 0;

for (int i = 1; i <= n; i++)
{
    sum += i;  // sum = sum + i
}

Console.WriteLine("Sum = " + sum);
```

#### c) **Check if a Number is Prime**

```csharp
int num = 13;
bool isPrime = true;

for (int i = 2; i <= Math.Sqrt(num); i++)
{
    if (num % i == 0)
    {
        isPrime = false;
        break;
    }
}

Console.WriteLine(isPrime ? "Prime" : "Not Prime");
```

---

## **Tips for Beginners**

1. Start by writing small programs for each concept.
2. Test `if/else` and `switch` with different inputs.
3. Practice loops on arrays or numbers.
4. Solve simple algorithm problems like:

   * Factorial of a number
   * Reverse a string
   * Sum of even numbers
5. Read your code out loud — it helps understand flow control.

---

