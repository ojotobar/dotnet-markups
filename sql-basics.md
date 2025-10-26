# 🏛 **Comprehensive SQL and Relational Database Notes**

## 1️⃣ Introduction to Databases

A **database** is an organized collection of data that can be easily accessed, managed, and updated.

### Common Types of Databases

| Type                   | Description                                             | Example                       |
| ---------------------- | ------------------------------------------------------- | ----------------------------- |
| **Relational (RDBMS)** | Stores data in tables with relationships                | SQL Server, MySQL, PostgreSQL |
| **NoSQL**              | Stores data in documents, key-value pairs, graphs, etc. | MongoDB, Redis, Cassandra     |

SQL (Structured Query Language) is used to **interact with relational databases**.

---

## 2️⃣ Understanding Database Objects

| Object               | Description                                          |
| -------------------- | ---------------------------------------------------- |
| **Database**         | Container for all data and objects                   |
| **Table**            | Stores data in rows and columns                      |
| **Column**           | Defines the type of data (e.g., `Name VARCHAR(100)`) |
| **Row / Record**     | A single data entry                                  |
| **View**             | A saved query that acts like a virtual table         |
| **Stored Procedure** | A saved set of SQL statements (like a function)      |
| **Trigger**          | Code that runs automatically when an event occurs    |
| **Index**            | Speeds up data lookup operations                     |
| **Schema**           | Logical grouping of tables and objects               |

---

## 3️⃣ Creating a Database

```sql
CREATE DATABASE SchoolDB;
GO

USE SchoolDB;
GO
```

### Delete a Database

```sql
DROP DATABASE SchoolDB;
```

---

## 4️⃣ Creating Tables (with Constraints)

```sql
CREATE TABLE Classes (
    ClassId INT PRIMARY KEY IDENTITY(1,1),
    ClassName VARCHAR(50) NOT NULL
);

CREATE TABLE Students (
    StudentId INT PRIMARY KEY IDENTITY(1,1),
    FullName VARCHAR(100) NOT NULL,
    Age INT CHECK (Age > 5),
    ClassId INT,
    CONSTRAINT FK_Students_Classes FOREIGN KEY (ClassId)
        REFERENCES Classes(ClassId)
);
```

### Explanation of Constraints:

| Constraint      | Purpose                          |
| --------------- | -------------------------------- |
| **PRIMARY KEY** | Uniquely identifies each record  |
| **FOREIGN KEY** | Links tables together            |
| **NOT NULL**    | Field must have a value          |
| **UNIQUE**      | Ensures all values are different |
| **CHECK**       | Validates data before insertion  |
| **DEFAULT**     | Provides a default value         |

---

## 5️⃣ Relationships Between Tables

| Type             | Description                                            | Example                                |
| ---------------- | ------------------------------------------------------ | -------------------------------------- |
| **One-to-One**   | One record in a table relates to one record in another | Each student has one profile           |
| **One-to-Many**  | One record relates to multiple others                  | One class has many students            |
| **Many-to-Many** | Multiple records relate to multiple others             | Students enrolled in multiple subjects |

### Many-to-Many Example

```sql
CREATE TABLE Subjects (
    SubjectId INT PRIMARY KEY IDENTITY(1,1),
    SubjectName VARCHAR(100)
);

CREATE TABLE StudentSubjects (
    StudentId INT,
    SubjectId INT,
    PRIMARY KEY (StudentId, SubjectId),
    FOREIGN KEY (StudentId) REFERENCES Students(Id),
    FOREIGN KEY (SubjectId) REFERENCES Subjects(Id)
);
```
---

## 🧱 **Code Explanation**

### 1️⃣ `Subjects` Table

```sql
CREATE TABLE Subjects (
    SubjectId INT PRIMARY KEY IDENTITY(1,1),
    SubjectName VARCHAR(100)
);
```

**Breakdown:**

* `SubjectId` is the **primary key** — unique for each subject.
* `IDENTITY(1,1)` means auto-increment from 1 upwards.
* `SubjectName` holds the name of the subject (e.g., "Mathematics", "Physics").

✅ Example Data:

| SubjectId | SubjectName |
| --------- | ----------- |
| 1         | Mathematics |
| 2         | Physics     |
| 3         | Chemistry   |

---

### 2️⃣ `StudentSubjects` Table

```sql
CREATE TABLE StudentSubjects (
    StudentId INT,
    SubjectId INT,
    PRIMARY KEY (StudentId, SubjectId),
    FOREIGN KEY (StudentId) REFERENCES Students(Id),
    FOREIGN KEY (SubjectId) REFERENCES Subjects(Id)
);
```

**This table is called a *junction* or *bridge* table.**
It represents the **many-to-many** relationship between students and subjects.

**Meaning:**

* A student can take many subjects.
* A subject can have many students.

✅ Example Data:

| StudentId | SubjectId |
| --------- | --------- |
| 1         | 1         |
| 1         | 2         |
| 2         | 1         |
| 3         | 3         |

This means:

* Student 1 takes Math and Physics
* Student 2 takes Math
* Student 3 takes Chemistry

---

## ⚙️ **Important Details**

### 🔹 Composite Primary Key

```sql
PRIMARY KEY (StudentId, SubjectId)
```

Ensures there are no duplicate pairs.
Example: You can’t accidentally insert `(1, 1)` twice.

### 🔹 Foreign Keys

```sql
FOREIGN KEY (StudentId) REFERENCES Students(StudentId),
FOREIGN KEY (SubjectId) REFERENCES Subjects(SubjectId)
```

These maintain referential integrity — you can’t add a subject enrollment for a student or subject that doesn’t exist.

---

## 🧮 **How to Insert Data**

### Add Subjects

```sql
INSERT INTO Subjects (SubjectName)
VALUES ('Mathematics'), ('Physics'), ('Chemistry');
```

### Add Student Enrollments

```sql
INSERT INTO StudentSubjects (StudentId, SubjectId)
VALUES (1, 1), (1, 2), (2, 1), (3, 3);
```

---

## 🔍 **How to Query It**

### List all Students and their Subjects

```sql
SELECT s.FullName, sub.SubjectName
FROM StudentSubjects ss
JOIN Students s ON ss.StudentId = s.StudentId
JOIN Subjects sub ON ss.SubjectId = sub.SubjectId;
```

### Find All Students Taking “Mathematics”

```sql
SELECT s.FullName
FROM StudentSubjects ss
JOIN Students s ON ss.StudentId = s.StudentId
JOIN Subjects sub ON ss.SubjectId = sub.SubjectId
WHERE sub.SubjectName = 'Mathematics';
```

---

## 💡 **Bonus Teaching Tip**

If your students struggle to visualize the relationships, use this analogy:

> **Students** = People
> **Subjects** = Classes
> **StudentSubjects** = Attendance sheet showing who takes what

---

## 6️⃣ Data Normalization

**Normalization** is the process of organizing data to reduce redundancy and improve data integrity.

| Normal Form                       | Definition                                                                           | Example                                                |
| --------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| **1NF (First Normal Form)**       | Each cell holds a single value (no repeating groups)                                 | ❌ “Phones: 0801, 0812” → ✅ Split into multiple rows    |
| **2NF (Second Normal Form)**      | No partial dependency — all non-key columns depend on the full primary key           | Avoid duplicating class names in student table         |
| **3NF (Third Normal Form)**       | No transitive dependency — non-key columns shouldn’t depend on other non-key columns | Move `TeacherName` from `Subjects` to `Teachers` table |
| **BCNF (Boyce-Codd Normal Form)** | Stricter version of 3NF — every determinant must be a candidate key                  |                                                        |

### Example of Poor Design (Unnormalized)

| StudentId | StudentName | ClassName | Teacher  |
| --------- | ----------- | --------- | -------- |
| 1         | Ada         | SS1       | Mr. John |
| 2         | John        | SS1       | Mr. John |

👉 Problem: Teacher name repeats unnecessarily.

### After Normalization

* `Students(StudentId, StudentName, ClassId)`
* `Classes(ClassId, ClassName, TeacherId)`
* `Teachers(TeacherId, TeacherName)`

Now each piece of data has **a single source of truth**.

---

## 7️⃣ Common SQL Operations (CRUD)

### Create

```sql
INSERT INTO Students (FullName, Age, ClassId)
VALUES ('Ada Obi', 16, 1);
```

### Read

```sql
SELECT * FROM Students;
```

### Update

```sql
UPDATE Students
SET Age = 17
WHERE StudentId = 1;
```

### Delete

```sql
DELETE FROM Students
WHERE StudentId = 1;
```

---

## 8️⃣ Filtering and Sorting

```sql
SELECT * FROM Students WHERE Age > 15;
SELECT * FROM Students ORDER BY FullName ASC;
SELECT * FROM Students WHERE Age BETWEEN 15 AND 18;
SELECT * FROM Students WHERE FullName LIKE 'A%';
```

---

## 9️⃣ Aggregation and Grouping

```sql
SELECT COUNT(*) AS TotalStudents FROM Students;
SELECT ClassId, AVG(Age) AS AvgAge FROM Students GROUP BY ClassId;
```

---

## 🔟 Joins Explained

| Type                | Purpose                                 |
| ------------------- | --------------------------------------- |
| **INNER JOIN**      | Only matching rows                      |
| **LEFT JOIN**       | All rows from left, matching from right |
| **RIGHT JOIN**      | All rows from right, matching from left |
| **FULL OUTER JOIN** | All rows from both sides                |
| **CROSS JOIN**      | Cartesian product of both tables        |

Example:

```sql
SELECT s.FullName, c.ClassName
FROM Students s
JOIN Classes c ON s.ClassId = c.ClassId;
```

---

## 11️⃣ Indexes

Indexes make lookups faster but slow down inserts/updates slightly.

```sql
CREATE INDEX IX_Students_FullName ON Students(FullName);
```

---

## 12️⃣ Views

A **view** is a saved query you can reuse as a virtual table.

```sql
CREATE VIEW vw_StudentClassInfo AS
SELECT s.FullName, c.ClassName
FROM Students s
JOIN Classes c ON s.ClassId = c.ClassId;
```

---

## 13️⃣ Stored Procedures

Reusable SQL scripts with parameters:

```sql
CREATE PROCEDURE GetStudentsByClass
    @ClassId INT
AS
BEGIN
    SELECT * FROM Students WHERE ClassId = @ClassId;
END;
```

Call it:

```sql
EXEC GetStudentsByClass @ClassId = 1;
```

---

## 14️⃣ Transactions

Used to ensure **atomicity** — all steps succeed or all fail.

```sql
BEGIN TRANSACTION;

UPDATE Students SET Age = 18 WHERE StudentId = 1;
DELETE FROM Students WHERE StudentId = 999; -- (invalid)

ROLLBACK TRANSACTION; -- undo everything if one fails
COMMIT TRANSACTION;   -- apply if all succeed
```

---

## 15️⃣ Interview Concepts to Know

| Concept                              | Explanation                                   |
| ------------------------------------ | --------------------------------------------- |
| **Normalization & Denormalization**  | Organizing vs combining for performance       |
| **Primary vs Foreign Key**           | Uniqueness vs relationship                    |
| **Clustered vs Non-Clustered Index** | Physical vs logical order of data             |
| **ACID Properties**                  | Atomicity, Consistency, Isolation, Durability |
| **Joins**                            | Ways to combine data                          |
| **Subqueries**                       | Query inside a query                          |
| **Views & Stored Procedures**        | Encapsulated reusable SQL logic               |
| **Transactions**                     | Ensures data integrity                        |
| **Data Types**                       | INT, VARCHAR, DATETIME, DECIMAL, etc.         |
| **NULL Handling**                    | Use `IS NULL` or `COALESCE()`                 |

---

## 16️⃣ Example Interview Questions

1. What is normalization and why is it important?
2. What are the types of joins and when do you use them?
3. Difference between primary key and unique key?
4. What’s a foreign key?
5. How do you improve query performance?
6. What is the difference between WHERE and HAVING?
7. What’s the purpose of indexes?
8. Explain ACID properties in databases.
9. What is a view?
10. What’s the difference between SQL and NoSQL?

---

## ⚖️ **11. EF Core and Dapper Context**
* **EF Core** automates these SQL operations behind the scenes.
* **Dapper** executes the SQL you write but maps the results to C# objects.

So SQL is **the foundation** — EF Core and Dapper are just **different ways to use it in code**.