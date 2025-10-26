## 🧩 Database Normalization

Normalization is the process of **organizing data** in a database to reduce **redundancy** (duplicate data) and improve **data integrity** (accuracy and consistency).

Imagine a badly designed student table:

| StudentId | StudentName | Course1 | Course2 | Course3   |
| --------- | ----------- | ------- | ------- | --------- |
| 1         | Ada         | Math    | Physics | Chemistry |
| 2         | Tunde       | Physics | NULL    | NULL      |

You see the problem? Adding or removing courses is a nightmare. We’re repeating course names in multiple columns.
Normalization solves this kind of mess.

There are several *normal forms*, each with stricter rules.

---

### **1️⃣ First Normal Form (1NF): No repeating groups or arrays**

**Rule:**

* Each column must hold **atomic** (indivisible) values.
* No repeating columns like `Course1`, `Course2`...
* Each record is unique (has a primary key).

✅ **Good example:**

| StudentId | StudentName | Course  |
| --------- | ----------- | ------- |
| 1         | Ada         | Math    |
| 1         | Ada         | Physics |
| 2         | Tunde       | Physics |

Each row holds a single value for each column — that’s atomic.

---

### **2️⃣ Second Normal Form (2NF): No partial dependency**

**Applies only if there’s a composite key (a key made of 2+ columns).**

**Rule:**

* The table must already be in 1NF.
* Every non-key column must depend on the **entire primary key**, not just part of it.

Example:
If a table’s primary key is `(StudentId, CourseId)` and there’s a column `StudentName`, that’s wrong — `StudentName` depends only on `StudentId`.

✅ Solution: Move `StudentName` to a separate `Students` table and keep only `StudentId` and `CourseId` here.

---

### **3️⃣ Third Normal Form (3NF): No transitive dependency**

**Rule:**

* Table must be in 2NF.
* No non-key column should depend on another non-key column.

Example:

| StudentId | StudentName | DepartmentId | DepartmentName |
| --------- | ----------- | ------------ | -------------- |
| 1         | Ada         | 10           | Science        |

Here, `DepartmentName` depends on `DepartmentId`, not directly on `StudentId`.
So we separate departments into their own table.

✅ Fix:

**Students**

| StudentId | StudentName | DepartmentId |
| --------- | ----------- | ------------ |
| 1         | Ada         | 10           |

**Departments**

| DepartmentId | DepartmentName |
| ------------ | -------------- |
| 10           | Science        |

---

### **4️⃣ Boyce-Codd Normal Form (BCNF)**

This is like **3NF on steroids** — it fixes some rare cases where:

* A non-primary key column can still determine another column.

Example:
If in a university, each **Course** is taught by **one Lecturer**, but each **Lecturer** can teach only one **Course**, we must ensure dependencies go only one way.

---

### ⚙️ Summary Table

| Normal Form | Rule Summary               | Goal                                                |
| ----------- | -------------------------- | --------------------------------------------------- |
| 1NF         | Atomic values only         | Eliminate repeating data                            |
| 2NF         | No partial dependency      | Eliminate redundant data depending on part of a key |
| 3NF         | No transitive dependency   | Eliminate indirect dependencies                     |
| BCNF        | Every determinant is a key | Handle rare edge cases                              |

---

## 💎 ACID Properties of Transactions

When we perform database operations (INSERT, UPDATE, DELETE), we want reliability — especially in systems that crash or run concurrently.

That’s where **ACID** comes in. It defines the four golden properties of a **transaction**.

---

### **A — Atomicity**

“All or nothing.”

* A transaction must either **fully succeed** or **fully fail**.
* If any part fails, everything rolls back.

🧠 Example:

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;

COMMIT TRANSACTION;
```

If the second update fails, the first one will roll back — no half-transfer.

---

### **C — Consistency**

“Data must always follow rules.”

* After a transaction, the database must remain valid (respect constraints, triggers, foreign keys, etc.).

🧠 Example:
You can’t insert a row with a `StudentId` that doesn’t exist in `Students` if there’s a foreign key constraint.

---

### **I — Isolation**

“Transactions shouldn’t interfere.”

* If two transactions run at the same time, they shouldn’t mess each other up.

SQL Server and others have isolation levels like:

* **Read Uncommitted:** can read uncommitted data (dirty read)
* **Read Committed:** reads only committed data (default)
* **Repeatable Read:** same query returns the same result during a transaction
* **Serializable:** most strict; fully isolated

---

### **D — Durability**

“Once committed, always committed.”

* Even if the system crashes, committed changes persist.
* Achieved via **transaction logs** — SQL Server’s `.ldf` file records all committed operations.

---

### ⚙️ ACID Summary Table

| Property    | Meaning                  | Example                                 |
| ----------- | ------------------------ | --------------------------------------- |
| Atomicity   | All or nothing           | Money transfer fully succeeds or fails  |
| Consistency | Data rules preserved     | No invalid foreign keys                 |
| Isolation   | Independent transactions | Two users editing different rows safely |
| Durability  | Changes survive crashes  | Power failure won’t undo a COMMIT       |

---