## 🧱 1. Unnormalized Example (0NF)

Let’s say you start with a table like this:

```sql
CREATE TABLE StudentsRaw (
    Id INT PRIMARY KEY IDENTITY(1,1),
    StudentName VARCHAR(100),
    Courses VARCHAR(200)  -- stores multiple courses in one cell, comma-separated
);

INSERT INTO StudentsRaw (StudentName, Courses) VALUES
('Ada', 'Math, Physics, Chemistry'),
('Tunde', 'Physics, English');
```

❌ Problems:

* Multiple courses in one field (violates **1NF**).
* Hard to query courses or add new ones.
* Can’t easily join or enforce foreign keys.

---

## ✅ 2. Apply First Normal Form (1NF)

Break out the courses into separate rows.

```sql
CREATE TABLE Students (
    Id INT PRIMARY KEY IDENTITY(1,1),
    StudentName VARCHAR(100)
);

CREATE TABLE Courses (
    Id INT PRIMARY KEY IDENTITY(1,1),
    CourseName VARCHAR(100)
);

CREATE TABLE StudentCourses (
    StudentId INT,
    CourseId INT,
    PRIMARY KEY (StudentId, CourseId),
    FOREIGN KEY (StudentId) REFERENCES Students(Id),
    FOREIGN KEY (CourseId) REFERENCES Courses(Id)
);
```

Now insert sample data:

```sql
INSERT INTO Students (StudentName) VALUES ('Ada'), ('Tunde');
INSERT INTO Courses (CourseName) VALUES ('Math'), ('Physics'), ('Chemistry'), ('English');

-- Link students to courses
INSERT INTO StudentCourses (StudentId, CourseId) VALUES
(1, 1), (1, 2), (1, 3), -- Ada: Math, Physics, Chemistry
(2, 2), (2, 4);         -- Tunde: Physics, English
```

✅ **Now it’s in 1NF and 2NF**, since all non-key columns depend on the full key.

---

## ✅ 3. Apply Third Normal Form (3NF)

Let’s say we expand the design and add **Departments**.

```sql
CREATE TABLE Departments (
    Id INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName VARCHAR(100)
);

ALTER TABLE Students ADD DepartmentId INT;
ALTER TABLE Students 
ADD FOREIGN KEY (DepartmentId) REFERENCES Departments(Id);
```

Now every department name is stored **once**, and students reference it.
No duplication, no dependency on non-key columns.

---

## 📚 4. Demonstrating ACID Properties

Let’s simulate a simple **transaction** using SQL.

### 🧩 Scenario

Ada wants to transfer ₦200 from her **Savings** to **Tuition** account.

```sql
CREATE TABLE Accounts (
    Id INT PRIMARY KEY IDENTITY(1,1),
    AccountName VARCHAR(100),
    Balance DECIMAL(10,2)
);

INSERT INTO Accounts (AccountName, Balance) VALUES
('Savings', 500.00),
('Tuition', 100.00);
```

---

### 🧨 Transaction with COMMIT (Atomicity + Consistency)

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 200 WHERE AccountName = 'Savings';
UPDATE Accounts SET Balance = Balance + 200 WHERE AccountName = 'Tuition';

COMMIT TRANSACTION;
```

✅ Both updates succeed → COMMIT makes them permanent.

If you run:

```sql
SELECT * FROM Accounts;
```

You’ll get:

```
1 | Savings | 300.00
2 | Tuition | 300.00
```

---

### ⚠️ Transaction with ROLLBACK (Atomicity in action)

Now let’s force a failure to see rollback in action.

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 200 WHERE AccountName = 'Savings';

-- Simulate an error (invalid account)
UPDATE Accounts SET Balance = Balance + 200 WHERE AccountName = 'UnknownAccount';

ROLLBACK TRANSACTION;
```

❌ Second update fails, so we `ROLLBACK`.
Now check again:

```sql
SELECT * FROM Accounts;
```

Balances are **unchanged**, meaning the first update was undone.
That’s **Atomicity** at work — “all or nothing.”

---

### ⚙️ Isolation Example (Two users at once)

Imagine two users both trying to update Ada’s record.
Depending on the **isolation level**, SQL Server will handle locks differently.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION;
-- Read Ada’s balance here, do some calculations
UPDATE Accounts SET Balance = Balance - 50 WHERE AccountName = 'Savings';
COMMIT TRANSACTION;
```

Other transactions can’t read uncommitted data until you `COMMIT`.

This ensures **Isolation** — concurrent users can’t see “half-done” updates.

---

### 💾 Durability Example

Once you `COMMIT`, SQL Server writes changes to the **transaction log (.ldf)**.
Even if the server crashes, it replays those logs on restart to restore committed changes.

That’s **Durability** — your COMMIT survives power cuts.

---

## ⚡ Summary

| Concept         | Purpose                           | Example                            |
| --------------- | --------------------------------- | ---------------------------------- |
| **1NF**         | Eliminate repeating groups        | Split courses into separate rows   |
| **2NF**         | Eliminate partial dependencies    | Move StudentName to Students table |
| **3NF**         | Eliminate transitive dependencies | Separate Department info           |
| **Atomicity**   | All-or-nothing operations         | ROLLBACK on error                  |
| **Consistency** | Keep data valid                   | FK ensures Student must exist      |
| **Isolation**   | Prevent conflicts                 | Transaction locks rows             |
| **Durability**  | Keep committed data safe          | Transaction logs                   |
