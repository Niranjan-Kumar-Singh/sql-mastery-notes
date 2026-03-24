# 🧱 Topic 16.2: 1st Normal Form (1NF)

The first step in cleaning a database is ensuring its atoms are "Unbreakable." This is the core of **1st Normal Form (1NF)**.

---

## 1. Definition

A table is in **1NF** if:
1.  Each cell contains only **Atoms** (Atomic/Single values).
2.  Each column contains data of the same type.
3.  Each column has a unique name.
4.  The order in which data is stored does not matter.

---

## 2. The Great Rule: NO Repeating Groups

Imagine a `Users` table with a column called `Hobbies`.
- **User A:** "Cycling, Cooking, SQL"
- **User B:** "Running"

This is **NOT in 1NF** because User A has 3 values in one cell. This makes it impossible to run a query like `SELECT * WHERE Hobby = 'Cooking'` easily (you'd have to use slow `LIKE %...%` filters).

---

## 3. Why This Concept Exists

1NF is the absolute minimum requirement for a relational database. It ensures that every "fact" in your table is directly searchable and indexable by the SQL engine.

---

## 4. How It Works (Atomic Decomposition - PRO LEVEL)

To move a table to 1NF:
1. Identify columns with multiple values.
2. Split them into separate rows or separate tables.

**Before 1NF:**
| Student | Subjects |
|---|---|
| John | Math, Science |

**After 1NF:**
| Student | Subject |
|---|---|
| John | Math |
| John | Science |

---

## 5. Tips & Best Practices (Pro-Level)

**Don't use ID1, ID2, ID3:**
A common "cheat" for 1NF is creating multiple columns like `Hobby1`, `Hobby2`, `Hobby3`. 
**Why this is a failure:** What if a user has 4 hobbies? You have to change the table structure. What if they have 0? You waste space with NULLs.
**Standard Fix:** Move hobbies to a separate `User_Hobbies` table.

---

## 6. Mini Practice Tasks

*   **Task 1:** What does it mean for a value in a database to be "Atomic"?
*   **Task 2:** Convert the following non-1NF data into 1NF: `(ID: 1, Name: "Niranjan", Phone: "9876543210, 1122334455")`.

---
