# 📌 Topic 14.3: SQL Variables

In Database Programming, you often need to store a piece of information (like a "Current Max ID" or a "Tax Rate") and use it across multiple lines of your code. MySQL provides **Variables** to handle this session-level state.

---

## 1. Types of Variables

MySQL has two primary types of variables you will use:

| Type | Syntax | Usage |
|---|---|---|
| **User-Defined** | `@variable_name` | Storing values during a specific session/connection. |
| **System** | `@@variable_name` | Reading or changing server settings (e.g., `@@version`). |

---

## 2. Why This Concept Exists

*   **Parameterization:** Setting a `@cutoff_date` at the top of a script and referencing it in 10 different Queries below.
*   **Procedural Logic:** Storing the count of deleted rows to use in a print statement or audit log.
*   **Dynamic Queries:** Passing values into Stored Procedures.

---

## 3. How It Works (Session Scope - PRO LEVEL)

User-defined variables are **loosely typed** and stay alive until you disconnect from MySQL.
- If you set `@x = 10` in Workbench, and then 5 minutes later you run `SELECT @x`, it will still return 10.
- If a different user connects, they **cannot** see your `@x`. Each connection has its own "Private Memory" for variables.

---

## 4. Syntax / Implementation

### Assigning Values
There are two ways to set a variable:

```sql
-- Method 1: SET command
SET @my_var = 100;

-- Method 2: Assign inside a SELECT (Walrus Operator)
SELECT @max_val := MAX(Price) FROM Products;
-- Note: Use := for assignment inside a SELECT.
```

### Reading Values
```sql
SELECT * FROM Orders WHERE Amount > @max_val;
```

---

## 5. System Variables (The "Pro-Engine" View)

System variables control how the MySQL engine behaves.
- **Global:** Affect the whole server (e.g., `@@max_connections`).
- **Session:** Affect only your current connection (e.g., `@@sql_mode`).

```sql
-- Check your MySQL version
SELECT @@version;

-- Check the current timezone
SELECT @@time_zone;
```

---

## 6. Common Mistakes

*   **Using `=` instead of `:=`:** In a `SELECT` statement, `=` is for comparison. If you want to **assign** a value to a variable during a query, you MUST use `:=`.
*   **Expecting variables to persist after logout:** Variables are stored in RAM; they do not get saved to the database.

---

## 7. Tips & Best Practices (Pro-Level)

**Default Values:**
If you reference a variable that has never been set, MySQL returns `NULL` and does **not** throw an error. This can lead to silent logic bugs in your scripts. Always initialize your variables at the top of your scripts.

---

## 8. Mini Practice Tasks

*   **Task 1:** What is the difference between `@var` and `@@var`?
*   **Task 2:** Write a script that sets a variable `@standard_tax` to `0.18` and then uses it to calculate the tax for a product price from the `Products` table.

---
