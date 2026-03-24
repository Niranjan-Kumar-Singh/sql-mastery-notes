# 🔑 Topic 2.3: Primary Keys vs. Candidate Keys

In a database with millions of rows, how do you find the *exact* one you're looking for? You use **Keys**.

---

## 1. Candidate Key

A **Candidate Key** is a set of one or more columns that can *uniquely* identify a record in a table. A table can have multiple Candidate Keys.

*Example:* In a `Users` table, both `Employee_ID` and `Social_Security_Number` can uniquely identify a person. Both are Candidate Keys.

---

## 2. Primary Key (PK)

The **Primary Key** is the "Chosen One." It is the single Candidate Key that you select as the official identifier for the table.

*Rules for a Primary Key:*
1.  **Unique:** No two rows can have the same PK.
2.  **NOT NULL:** It cannot be empty.
3.  **Static:** Its value should never change.

---

## 3. The Difference (PRO LEVEL)

*   **Candidate Keys** are all the "potential" identifiers.
*   **Primary Key** is the one you actually use to build relationships (Foreign Keys) with other tables.

**Senior Tip:** Every table **must** have a Primary Key. Without it, your data becomes a disorganized pile that is impossible to join or update safely.

---

## 4. SQL Implementation

```sql
CREATE TABLE Employees (
    Emp_ID INT PRIMARY KEY,      -- The Chosen Primary Key
    National_ID VARCHAR(20) UNIQUE -- Another Candidate Key (Unique but not the PK)
);
```

---

## 5. Mini Practice Tasks

*   **Task 1:** If a table has `Email`, `Username`, and `ID`, and all are unique, how many Candidate Keys are there?
*   **Task 2:** Can a table have more than one Primary Key? (Careful: Don't confuse this with Composite Keys!).

---
