# 🔄 Topic 2.8: Alternate Key — The Runners-Up

You have 3 candidates for class president. Only one person wins. The others weren't bad — they just weren't chosen. In the database world, these "runners-up" identifiers are called **Alternate Keys**.

---

## 1. Definition

An **Alternate Key** is any **Candidate Key** that was **NOT chosen** as the Primary Key.

- If a table has 3 columns that can each uniquely identify a row (3 Candidate Keys), and you pick 1 as the Primary Key, the remaining 2 are **Alternate Keys**.

---

## 2. Recall: The Key Hierarchy

To understand Alternate Keys, remember the relationship between all key types:

```
All unique identifying columns → Candidate Keys
         ↓
  One is selected → Primary Key
         ↓
  The rest become → Alternate Keys
```

---

## 3. Why This Concept Exists

In the real world, entities often have more than one natural unique identifier:

| Table | Primary Key (chosen) | Alternate Keys (the others) |
|---|---|---|
| `Users` | `User_ID` | `Email`, `Username` |
| `Employees` | `Employee_ID` | `National_Insurance_No`, `Work_Email` |
| `Products` | `Product_ID` | `Barcode`, `SKU` |

Just because `Email` isn't the Primary Key doesn't mean duplicates should be allowed! We still need those values to be unique.

---

## 4. How Alternate Keys Are Enforced in MySQL

Alternate Keys are enforced using the **`UNIQUE` constraint**. This tells MySQL: *"This column isn't the PK, but it must still be unique across all rows."*

```sql
CREATE TABLE Users (
    User_ID     INT          PRIMARY KEY AUTO_INCREMENT, -- Primary Key (chosen)
    Username    VARCHAR(50)  NOT NULL UNIQUE,             -- Alternate Key #1
    Email       VARCHAR(100) NOT NULL UNIQUE,             -- Alternate Key #2
    Phone       VARCHAR(15)  UNIQUE,                      -- Alternate Key #3
    Password    VARCHAR(255) NOT NULL
);
```

In this example, `Username`, `Email`, and `Phone` are all **Alternate Keys** — they are unique, but `User_ID` was chosen as the official Primary Key because it's a simple, stable, auto-generated number.

---

## 5. Primary Key vs Alternate Key — Side-by-Side

| Property | Primary Key | Alternate Key |
|---|---|---|
| Can it be `NULL`? | ❌ Never | ✅ Sometimes (if `UNIQUE` allows `NULL`) |
| How many per table? | **Only one** | Multiple |
| Used for FK references? | ✅ Yes, commonly | Rarely (but possible) |
| Enforced by | `PRIMARY KEY` constraint | `UNIQUE` constraint |
| Is it indexed? | ✅ Always (Clustered Index) | ✅ Yes (Non-Clustered Index) |

---

## 6. Real-Life Example

**Bank Account System:**

| Account_ID (PK) | Account_Number (AK) | IBAN (AK) | Customer_ID |
|---|---|---|---|
| 1 | ACC-2024-001 | GB29NWBK60161331926819 | 501 |
| 2 | ACC-2024-002 | GB29NWBK60161331926820 | 502 |

Both `Account_Number` and `IBAN` uniquely identify the account. But neither was chosen as the Primary Key — the system uses an internal auto-incremented `Account_ID`. The bank's system can still look up accounts by their IBAN or Account Number because they are alternate keys enforced with `UNIQUE`.

---

## 7. Common Mistakes

- **Not enforcing Alternate Keys:** Many beginners pick a Primary Key and forget to add `UNIQUE` to the other unique columns. This silently allows duplicate emails or usernames to enter the database — a major security and data integrity bug.
- **Confusing Alternate Key with `UNIQUE` Index:** They are conceptually the same thing. A `UNIQUE` constraint in MySQL *creates* a Unique Index internally. The term "Alternate Key" is the academic/ER-diagram concept; `UNIQUE` is how you implement it in SQL.

---

## 8. Tips & Best Practices

- **Always enforce your Alternate Keys** with `UNIQUE` constraints. If a value *should* be unique based on business rules, your database should enforce it — not your application code.
- **Query performance:** Just like a Primary Key, a `UNIQUE` column automatically gets an index. This means lookups like `WHERE Email = 'john@gmail.com'` are lightning fast even on tables with millions of rows.

---

## 9. Mini Practice Tasks

- **Task 1:** In a `Products` table with columns `Product_ID`, `Barcode`, `SKU`, and `Name` — if `Product_ID` is the Primary Key and both `Barcode` and `SKU` should also be unique, what SQL constraint(s) do you add? Write the complete `CREATE TABLE` statement.
- **Task 2:** Can an Alternate Key column contain `NULL`? (Research MySQL's behavior: what happens if you insert `NULL` twice into a `UNIQUE` column?)

---
