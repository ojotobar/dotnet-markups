Ah, the **meat and potatoes** of programming.
This is where beginners stop “watching code” and start *thinking* like programmers. Old school, chalk-on-the-board stuff that actually sticks.

I’ll give you **progressive exercises** — from “warm-up” to “this hurts a little (good)”.
All examples are **console-based**, no distractions.

---

# EXERCISES SET

*(Data Types, Operators, Conditionals, Loops)*

> Rule for students:
> **No copy-paste. Type everything. Fingers teach the brain.**

---

## PART 1 — DATA TYPES (FOUNDATION)

### Exercise 1: Identify the Data Type

Ask students to say **which type fits best** and why.

1. Age of a person
2. Price of a product
3. Number of students in a class
4. A yes/no answer
5. A single letter grade (A, B, C)
6. Full name of a user

👉 Goal: make them **choose intentionally**, not randomly.

---

### Exercise 2: Declare & Print

Write a program that declares:

* An `int` for age
* A `double` or `decimal` for price
* A `string` for name
* A `bool` for isStudent

Print them to the console in a sentence.

💡 Emphasize:

> Data types describe **meaning**, not just storage.

---

### Exercise 3: Type Conversion

1. Ask the user to enter their age as a string.
2. Convert it to an integer.
3. Add 5 and print the result.

Discuss:

* Why conversion is needed
* What happens if input is invalid (no try/catch yet)

---

## PART 2 — OPERATORS (THINKING IN EXPRESSIONS)

### Exercise 4: Arithmetic Operators

Write a program that:

* Takes two numbers
* Prints:

  * Sum
  * Difference
  * Product
  * Quotient
  * Remainder

Introduce:

* `+ - * / %`

Ask:

> When would remainder (`%`) be useful in real life?

---

### Exercise 5: Comparison Operators

Given two numbers:

* Print whether:

  * First is greater than second
  * They are equal
  * First is less than second

Use:

* `> < >= <= == !=`

---

### Exercise 6: Logical Operators

Given:

```csharp
int age = 20;
bool hasID = true;
```

Write expressions that check:

* Is age ≥ 18 AND has ID?
* Is age < 18 OR has ID?
* NOT has ID

Explain:

* `&& || !`
* Truth tables (briefly, not academically)

---

## PART 3 — IF / ELSE (DECISION MAKING)

### Exercise 7: Even or Odd

1. Ask the user for a number.
2. Print whether it’s even or odd.

Hint:

* Use `%`

---

### Exercise 8: Grade Checker

Input a score (0–100).

Rules:

* 90–100 → A
* 80–89 → B
* 70–79 → C
* Below 70 → F

Use `if / else if / else`.

Discuss:

> Order matters. The computer is literal.

---

### Exercise 9: Login Simulation

Given:

```csharp
string correctPassword = "1234";
```

Ask user to enter password.

* If correct → “Access granted”
* Else → “Access denied”

Talk about:

* Equality vs assignment
* Case sensitivity

---

## PART 4 — SWITCH (CLEAR INTENT)

### Exercise 10: Day of the Week

Input a number (1–7).
Print the day name.

Example:

* 1 → Monday
* 7 → Sunday
* Default → Invalid day

Explain:

* Why switch is cleaner than many `if`s

---

### Exercise 11: Simple Menu

Display:

```
1. Deposit
2. Withdraw
3. Check Balance
```

User selects an option.
Print what they chose using `switch`.

---

## PART 5 — LOOPS (REPETITION WITH PURPOSE)

### Exercise 12: Count Up

Use a `for` loop to print numbers 1–10.

Ask:

* Where does the loop start?
* When does it stop?

---

### Exercise 13: Count Down

Print numbers from 10 to 1.

Explain:

* Increment vs decrement

---

### Exercise 14: Sum of Numbers

Ask user for a number `n`.
Use a loop to calculate the sum from `1` to `n`.

Example:

* Input: 5
* Output: 15

---

### Exercise 15: While Loop Input Validation

Ask user to enter a number **greater than 0**.
Keep asking until they do.

Introduce:

* `while`
* Sentinel conditions

---

### Exercise 16: Multiplication Table

Ask user for a number.
Print its multiplication table from 1–10.

---

## PART 6 — MINI CHALLENGES (COMBINE EVERYTHING)

### Challenge 1: Number Guessing Game

* Program picks a number (hardcode it for now)
* User guesses
* Tell them:

  * Too high
  * Too low
  * Correct

Use:

* `while`
* `if/else`
* Comparison operators

---

### Challenge 2: Simple ATM

Start with balance = 1000.

Menu:

1. Check Balance
2. Deposit
3. Withdraw
4. Exit

Rules:

* Cannot withdraw more than balance
* Loop until user exits

---

## Teaching Advice (Old but Gold)

* Let them **struggle a bit**
* Don’t correct immediately
* Ask *why* before *how*
* Mistakes are part of the lesson

> Loops teach patience.
> Conditionals teach judgment.
> Operators teach precision.


Perfect choice. This is *exactly* the kind of exercise that shows whether the basics have landed or not.

Let’s walk it **slowly, clearly, and the traditional way** — one `Main`, no classes yet, no fancy tricks. Just logic.

---

## 🏦 SIMPLE ATM PROGRAM (Console, C#)

### Requirements recap

* Start with `balance = 1000`
* Show a menu
* Let the user choose
* Loop until they choose **Exit**
* Prevent overdraft

---

## FULL WORKING CODE

```csharp
using System;

class Program
{
    static void Main()
    {
        decimal balance = 1000m;
        bool running = true;

        while (running)
        {
            Console.WriteLine("\n--- ATM MENU ---");
            Console.WriteLine("1. Check Balance");
            Console.WriteLine("2. Deposit");
            Console.WriteLine("3. Withdraw");
            Console.WriteLine("4. Exit");
            Console.Write("Choose an option: ");

            string input = Console.ReadLine();

            switch (input)
            {
                case "1":
                    Console.WriteLine($"Your balance is: {balance}");
                    break;

                case "2":
                    Console.Write("Enter deposit amount: ");
                    decimal deposit = decimal.Parse(Console.ReadLine());

                    if (deposit > 0)
                    {
                        balance += deposit;
                        Console.WriteLine("Deposit successful.");
                    }
                    else
                    {
                        Console.WriteLine("Invalid deposit amount.");
                    }
                    break;

                case "3":
                    Console.Write("Enter withdrawal amount: ");
                    decimal withdraw = decimal.Parse(Console.ReadLine());

                    if (withdraw <= 0)
                    {
                        Console.WriteLine("Invalid withdrawal amount.");
                    }
                    else if (withdraw > balance)
                    {
                        Console.WriteLine("Insufficient funds.");
                    }
                    else
                    {
                        balance -= withdraw;
                        Console.WriteLine("Withdrawal successful.");
                    }
                    break;

                case "4":
                    Console.WriteLine("Thank you for using the ATM.");
                    running = false;
                    break;

                default:
                    Console.WriteLine("Invalid option. Please try again.");
                    break;
            }
        }
    }
}
```

---

## 🧠 HOW THIS WORKS (THE IMPORTANT PART)

### 1️⃣ Balance

```csharp
decimal balance = 1000m;
```

* `decimal` is used for money (important habit)
* `m` tells C# “this is a decimal literal”

---

### 2️⃣ Loop Until Exit

```csharp
bool running = true;

while (running)
{
    ...
}
```

Old-school logic:

> As long as `running` is true, the ATM keeps running.

Exit flips it to `false`.

---

### 3️⃣ Menu Input

```csharp
string input = Console.ReadLine();
```

Why `string`?

* Console input is **always text**
* We decide later how to interpret it

---

### 4️⃣ switch (Clear Intent)

```csharp
switch (input)
```

Each `case` maps directly to a menu option.
This is cleaner than a pile of `if/else`.

---

### 5️⃣ Deposit Logic

```csharp
if (deposit > 0)
{
    balance += deposit;
}
```

Rules enforced:

* No zero
* No negative deposits

---

### 6️⃣ Withdraw Logic (THE KEY RULE)

```csharp
else if (withdraw > balance)
{
    Console.WriteLine("Insufficient funds.");
}
```

This is the **business rule**:

> You can’t take what you don’t have.

---

## 🚨 COMMON BEGINNER ERRORS (WATCH FOR THESE)

* Using `int` for money ❌
* Forgetting to update `balance`
* Infinite loop (never setting `running = false`)
* Parsing input without checking
* Mixing up `=` and `==`

---

## 🧪 EXTENSION EXERCISES (FOR STUDENTS)

Ask them to:

1. Add a **minimum withdrawal limit**
2. Show balance after every transaction
3. Count how many transactions occurred
4. Prevent crashing on invalid input (no try/catch yet — use logic)

---

## TEACHING NOTE (IMPORTANT)

Don’t rush this exercise.

If a student can:

* Explain **why** the loop exists
* Explain **why** withdrawal is checked
* Explain **why** `decimal` is used