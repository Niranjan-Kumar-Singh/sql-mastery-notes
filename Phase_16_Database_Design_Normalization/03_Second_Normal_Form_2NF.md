# 🏗️ Topic 16.3: 2nd Normal Form (2NF)

Once your table is in 1NF (all atomic values), the next question is: *Do all columns actually belong together?* This is the heart of **2nd Normal Form (2NF)**.

---

## 1. Definition

A table is in **2NF** if:
1. It is already in **1NF** (all atomic values, no repeating groups).
2. Every non-key column is **Fully Functionally Dependent** on the **entire Primary Key** — not just a part of it.

**The Key Insight:** 2NF is only a concern when your table has a **Composite Primary Key** (a PK made of two or more columns). If your table has a single-column PK, it is automatically in 2NF (as long as it's in 1NF).

---

## 2. What is a "Partial Dependency"?

A **Partial Dependency** occurs when a non-key column depends on only **one part** of a Composite Primary Key, not the whole thing.

**Example — the problem:**

Imagine a table tracking student course enrollments:

| `Student_ID` (PK) | `Course_ID` (PK) | `Student_Name` | `Course_Name` | `Grade` |
|---|---|---|---|---|
| 101 | MATH1 | Ayesha | Mathematics | A |
| 101 | PHY1 | Ayesha | Physics | B |
| 102 | MATH1 | Ravi | Mathematics | C |

**Composite PK:** `(Student_ID, Course_ID)`

**The Problem:**
- `Student_Name` only depends on `Student_ID`. If we know the student ID, we know their name — we don't need the Course_ID. This is a **Partial Dependency**.
- `Course_Name` only depends on `Course_ID`. Same problem.
- `Grade` depends on **both** `Student_ID` AND `Course_ID` — this is correct! A grade is specific to *that student in that course*.

---

## 3. Why Does This Matter?

Partial dependencies cause three serious problems called **Database Anomalies**:

1. **Update Anomaly:** Ayesha changes her name to "Aisha". You have to update it in 2 rows (one for MATH, one for PHY). If you miss one, your data is inconsistent.
2. **Insert Anomaly:** You can't add a new course to the system until at least one student enrolls in it (because `Student_ID` is part of the PK).
3. **Delete Anomaly:** If you delete Ravi's enrollment in MATH1, you permanently lose the information that MATH1 is called "Mathematics" (if it's the last enrollment).

---

## 4. The Fix: Decomposition

Move the partially dependent columns into their own separate tables.

**After 2NF (Decomposed):**

**Table 1: `Students`**
| `Student_ID` (PK) | `Student_Name` |
|---|---|
| 101 | Ayesha |
| 102 | Ravi |

**Table 2: `Courses`**
| `Course_ID` (PK) | `Course_Name` |
|---|---|
| MATH1 | Mathematics |
| PHY1 | Physics |

**Table 3: `Enrollments` (Junction Table)**
| `Student_ID` (FK, PK) | `Course_ID` (FK, PK) | `Grade` |
|---|---|---|
| 101 | MATH1 | A |
| 101 | PHY1 | B |
| 102 | MATH1 | C |

Now, every non-key column in each table depends on the *whole* primary key of that table. The database is clean!

---

## 5. SQL Implementation

```sql
CREATE TABLE Students (
    Student_ID   INT          PRIMARY KEY,
    Student_Name VARCHAR(100) NOT NULL
);

CREATE TABLE Courses (
    Course_ID   VARCHAR(10)  PRIMARY KEY,
    Course_Name VARCHAR(100) NOT NULL
);

CREATE TABLE Enrollments (
    Student_ID INT NOT NULL,
    Course_ID  VARCHAR(10) NOT NULL,
    Grade      CHAR(1),
    PRIMARY KEY (Student_ID, Course_ID),
    FOREIGN KEY (Student_ID) REFERENCES Students(Student_ID),
    FOREIGN KEY (Course_ID)  REFERENCES Courses(Course_ID)
);
```

---

## 6. Quick Rule to Check for 2NF

Ask yourself: "Could any non-key column be determined by only PART of my primary key?"
- If **Yes** → Partial Dependency exists → Not in 2NF → Decompose.
- If **No** → Table is in 2NF ✅

---

## 7. Mini Practice Tasks

- **Task 1:** A table has PK `(Order_ID, Product_ID)`. It also has columns `Customer_Name`, `Product_Price`, and `Quantity`. Which of these columns likely have Partial Dependencies? Which one doesn't?
- **Task 2:** Is 2NF relevant for a table with a single-column Primary Key? Why or why not?

---
