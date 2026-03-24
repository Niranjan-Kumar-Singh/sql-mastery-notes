# 🌌 Topic 2.9: Super Keys

Before you pick a Candidate Key or a Primary Key, you are technically dealing with **Super Keys**.

---

## 1. Definition

**Super Key:** A set of one or more columns that can uniquely identify a record in a table. It is the most "general" type of key.

---

## 2. Super Key vs. Candidate Key (PRO LEVEL)

This is a common interview "trap" question. The difference lies in **Minimality**.

*   **Super Key:** Any combination of columns that is unique. (e.g., `Employee_ID + Employee_Name`). This is unique, but it contains "extra" information (`Name`) that isn't strictly needed for uniqueness.
*   **Candidate Key:** A **Minimal Super Key**. It is the smallest possible set of columns that can uniquely identify a row. (e.g., `Employee_ID` on its own).

---

## 3. Visualizing the Hierarchy

```text
SUPER KEYS (All unique combinations)
    └── CANDIDATE KEYS (Minimal unique combinations)
             └── PRIMARY KEY (The one chosen by the DBA)
```

---

## 4. Why This Concept Exists

Super Keys are the starting point of database design. You identify all unique combinations (Super Keys), trim them down to the smallest unique versions (Candidate Keys), and then select your official identifier (Primary Key).

---

## 5. Mini Practice Tasks

*   **Task 1:** If `ID` is unique, is `(ID, Name)` a Super Key? Is it a Candidate Key?
*   **Task 2:** "All Candidate Keys are Super Keys, but not all Super Keys are Candidate Keys." Explain why this statement is true.

---
