# 🤖 Topic 2.7: Surrogate Keys vs. Natural Keys

Every table needs an identifier. Should you use data that already exists (Natural) or create a "fake" ID (Surrogate)?

---

## 1. Natural Key

A **Natural Key** is a column that uniquely identifies a record based on real-world data that *already exists*.

*Examples:*
*   Social Security Number (SSN)
*   Email Address
*   Vehicle Identification Number (VIN)

---

## 2. Surrogate Key

A **Surrogate Key** is a "made-up" unique identifier (usually a number) that has no real-world meaning. It is generated automatically by the database.

*Example:* `User_ID` (1, 2, 3...) or a `UUID`.

---

## 3. The Showdown (PRO LEVEL)

**Why Professionals Prefer Surrogate Keys (Auto-increment IDs):**
1.  **Immutability:** Real-world data changes (people change emails, SSNs can be re-issued). If you change a Natural Key, you break all Foreign Key links in other tables. A Surrogate Key never changes.
2.  **Performance:** Comparing two small Integers (`INT`) is much faster for the CPU than comparing long strings (Emails).
3.  **Privacy:** You don't want to leak sensitive data like SSNs in URL parameters or logs.

---

## 4. SQL Implementation

```sql
-- Using a Surrogate Key (The Pro Way)
CREATE TABLE Users (
    User_ID INT AUTO_INCREMENT PRIMARY KEY, -- Surrogate Key
    Email VARCHAR(100) UNIQUE               -- The Natural Key remains as a unique constraint
);
```

---

## 5. Mini Practice Tasks

*   **Task 1:** Name one disadvantage of using an Email address as a Primary Key.
*   **Task 2:** What is the `AUTO_INCREMENT` keyword used for in MySQL?

---
