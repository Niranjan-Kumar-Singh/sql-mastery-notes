# 🆔 Topic 2.5: Unique Keys vs. Primary Keys

Both ensure that data isn't duplicated, but they serve completely different purposes in database architecture.

---

## 1. Definition

*   **Primary Key (PK):** The main identifier for a row. It is unique and **cannot be NULL**. You can only have ONE per table.
*   **Unique Key:** Ensures values in a column are unique, but it **can accept one (or more) NULL values**. You can have MANY Unique Keys per table.

---

## 2. The Difference Table

| Feature | Primary Key | Unique Key |
|---|---|---|
| **Limit** | Only 1 per table. | Multiple allowed. |
| **NULL values** | Strictly forbidden. | One NULL allowed (typically). |
| **Purpose** | Row identity. | Data uniqueness (Email, Nickname). |
| **Indexing** | Automatically creates a Clustered Index (data is stored in PK order). | Creates a Non-Clustered Index. |

---

## 3. Why This Concept Exists (PRO LEVEL)

You use a **Primary Key** for the "internal" logic (IDs, linking tables). You use a **Unique Key** for "business" constraints (ensuring no two users register with the same Email).

---

## 4. SQL Implementation

```sql
CREATE TABLE Users (
    User_ID INT PRIMARY KEY, -- Purpose: ID
    Email VARCHAR(100) UNIQUE, -- Purpose: Business constraint
    Username VARCHAR(50) UNIQUE -- Purpose: Business constraint
);
```

---

## 5. Mini Practice Tasks

*   **Task 1:** Can you have two `UNIQUE` columns in one table?
*   **Task 2:** Why is a `Primary Key` better for linking tables than a `Unique Key` like an Email address? (Hint: Think about what happens if a user changes their email).

---
