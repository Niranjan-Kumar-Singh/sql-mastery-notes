# 📛 Topic 4.8: RENAME Table

You have spent 7 topics learning how to construct and modify the internal structure of a table. But what if the problem is not inside the table — what if the table itself has a bad name?

In large enterprise codebases, table naming conventions evolve. A table named `data_tbl_old` needs to be surgically renamed to `Customer_Profiles` without disturbing a single byte of the millions of rows inside it. This is the job of **`RENAME TABLE`**.

---

## 1. Definition

**`RENAME TABLE`** is a DDL command that changes the publicly visible **name** of an existing table in the database dictionary. It does not touch the underlying data, schema structure, constraints, or indexes inside the table.

---

## 2. Why This Concept Exists

Because software codebases mature. An early-stage startup might carelessly name tables like `stuff` or `data2`. As the company grows and brings on a team of 30 engineers, those naming conventions become a professional hazard causing query confusion, production bugs, and code misreads.

---

## 3. Why We Use It

*   **Refactoring Architecture:** When multiple teams agree on a standardized naming convention and retroactively enforce it across all existing tables.
*   **Zero-Data-Loss Renaming:** It is dramatically safer than the amateur approach of `CREATE TABLE new_name ... INSERT INTO new_name SELECT * FROM old_name ... DROP TABLE old_name`, which is slow, expensive, and risky for large tables.

---

## 4. How It Works (Physical Dictionary Rewrite - PRO LEVEL)

A table rename is one of the fastest DDL operations in MySQL. 

When you execute `RENAME TABLE old_name TO new_name;`, MySQL does not rebuild the physical `.ibd` file on the hard drive. It simply locates the table's **metadata entry** in InnoDB's internal Data Dictionary (stored in special system tables), atomically updates that single text string from `old_name` to `new_name`, and the operation completes in *microseconds* regardless of how many billions of rows are inside the table.

---

## 5. Syntax / Implementation (Cheat Sheet)

There are two valid syntaxes that accomplish the same task:

```sql
-- Method 1: The dedicated RENAME TABLE command
RENAME TABLE old_table_name TO new_table_name;

-- Method 2: The ALTER TABLE approach (same result, slightly more verbose)
ALTER TABLE old_table_name RENAME TO new_table_name;
```

**Power Move — Renaming Multiple Tables in One Atomic Transaction:**
```sql
-- This renames both tables simultaneously! If one fails, neither is renamed.
RENAME TABLE 
    Users TO Customers,
    Products TO Inventory_Items;
```

---

## 6. Real-Life Examples

**The Company Rebrand:**
Your startup "Grab-It" was acquired by "Amazon". All internal database tables previously named `grabit_orders` and `grabit_users` must be professionally renamed to `amazon_orders` and `amazon_users` to match the new global coding standard, without touching a single order record.

```sql
RENAME TABLE 
    grabit_orders TO amazon_orders,
    grabit_users TO amazon_users;
```

---

## 7. SQL Examples (MySQL Execution)

```sql
-- We started with a horribly named table from a junior developer
CREATE TABLE tbl123 (id INT PRIMARY KEY, info VARCHAR(100));

-- A senior architect decides to rename it professionally
RENAME TABLE tbl123 TO Customer_Profiles;

-- The table is now Customer_Profiles with all its data perfectly intact!
SELECT * FROM Customer_Profiles;
```

---

## 8. Common Mistakes

*   **Forgetting to Update Application Code:** If your Python or Node.js application code has hundreds of `SELECT * FROM tbl123` statements hardcoded in it, renaming the database table will cause every single one of those backend queries to crash with "Table Not Found" errors. Always **globally find and replace** all references to the old table name in your codebase before renaming it in production!
*   **Renaming Across Databases:** MySQL allows you to combine a `RENAME TABLE` with a database prefix to move a table between two different databases on the same server!
    ```sql
    -- Moving the Orders table from Database A to Database B atomically!
    RENAME TABLE DatabaseA.Orders TO DatabaseB.Orders;
    ```

---

## 9. Tips & Best Practices (Pro-Level)

**The Zero-Downtime Blue-Green Rename Strategy:**
In enterprise CI/CD deployments, engineers use renames to achieve completely zero-downtime "Blue-Green deployments". 

Here is the exact technique used by companies like GitHub:
1.  Build the new `Users_v2` table with the updated schema in the background.
2.  Migrate all existing data from `Users` into `Users_v2` while the system is still live.
3.  Perform one atomic `RENAME TABLE Users TO Users_old, Users_v2 TO Users;` — This swaps the two tables instantaneously without a single millisecond of downtime!
4.  Verify the production system is stable for 24 hours.
5.  `DROP TABLE Users_old;` — Clean up.

---

## 10. Mini Practice Tasks

*   **Task 1:** You have a table named `emp_data_old`. Write the SQL command to professionally rename it to `Employees`.
*   **Task 2:** Your company launches in Canada, and you need to simultaneously rename `US_Orders` to `Global_Orders` and `US_Customers` to `Global_Customers` atomically (so if one fails, neither changes). Write the single SQL statement to safely accomplish this.

---
