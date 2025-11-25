# 🧠 What Are Bitwise Operators?

They operate on integers **bit-by-bit**.
Think of numbers as 32-bit patterns like:

```
5  = 00000000 00000000 00000000 00000101
10 = 00000000 00000000 00000000 00001010
```

---

# 🔥 The Operators (with examples)

## 1. `&` — AND

Both bits must be **1** to get **1**.

```
5  = 0101
10 = 1010
------------
&  = 0000   → 0
```

C#:

```csharp
int result = 5 & 10; // result = 0
```

Another example:

```
1101
1011
----
1001
```

---

## 2. `|` — OR

If **either** bit is 1 → result is 1.

```
5  = 0101
10 = 1010
-----------
|  = 1111  → 15
```

C#:

```csharp
int result = 5 | 10; // result = 15
```

---

## 3. `^` — XOR

1 if the bits are **different**, 0 if they’re the same.

```
5  = 0101
10 = 1010
-----------
^  = 1111  → 15
```

Example with different pattern:

```
1100
1010
----
0110 → 6
```

C#:

```csharp
int result = 5 ^ 10; // 15
```

---

## 4. `~` — NOT (bitwise complement)

Flips **every** bit (0 → 1, 1 → 0).

```
5 = 0000 0101
~5= 1111 1010 → -6   // two’s complement result
```

C#:

```csharp
int result = ~5; // -6
```

---

# 🏎️ Shift Operators

## 5. `<<` — Left Shift

Shifts bits left, filling with zeros on the right.
Equivalent to multiplying by 2 for each shift.

Example:

```
5 = 0101
5 << 1 = 1010 → 10
```

C#:

```csharp
int result = 5 << 1; // 10
```

---

## 6. `>>` — Right Shift (Arithmetic)

Shifts bits right, preserving the sign bit.

```
20 = 0001 0100
20 >> 2 = 0000 0101 → 5
```

Negative example:

```
-4 = 1111 1100
-4 >> 1 = 1111 1110 → -2   // sign preserved
```

C#:

```csharp
int result = 20 >> 2; // 5
```

---

## 7. `>>>` — Unsigned Right Shift

(C# **does not** have this operator directly; Java does.)

But in C#, you can simulate for `uint`.

Example with `uint`:

```csharp
uint x = 0xF0000000;
uint y = x >> 1; // logical shift because it's unsigned
```

---

# 🎯 Practical Use Cases

## ⚡ 1. Make an int positive

```csharp
int positive = value & 0x7FFFFFFF;
```

## ⚡ 2. Check if a number is even

```csharp
bool isEven = (x & 1) == 0;
```

## ⚡ 3. Combine flags

```csharp
[Flags]
enum Permissions { Read=1, Write=2, Delete=4 }

var perm = Permissions.Read | Permissions.Write; // combine
bool hasWrite = (perm & Permissions.Write) != 0;
```

## ⚡ 4. Quickly multiply by powers of 2

```csharp
int doubled = x << 1;
int quadrupled = x << 2;
```

## ⚡ 5. Extract certain bits

```csharp
int lower4 = value & 0xF;  // 1111
```