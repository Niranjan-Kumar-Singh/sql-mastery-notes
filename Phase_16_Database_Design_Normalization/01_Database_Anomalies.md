# 🕵️ Topic 16.1: Database Anomalies

Why can't we just store all our data in one giant table, like an Excel sheet? Because of **Database Anomalies**. An anomaly is a "glitch" that occurs when a table's design is logically flawed, leading to inconsistent or missing data.

---

## 1. The Three Deadly Anomalies

### A. Insertion Anomaly
You cannot add a piece of data because another piece of data is missing.
- **Example:** You have a table `Student_Courses`. You cannot record a new `Course` until you have at least one `Student` enrolled in it. (Because `Student_ID` is forced to be NOT NULL).

### B. Update Anomaly
You have to change a piece of data in multiple rows, risking a "Partial Update."
- **Example:** If a student's `Address` is stored in 50 different rows (one for each course they take), and they move house, you must update 50 rows. If you miss one, the student appears to live in two different places!

### C. Deletion Anomaly
Deleting one piece of information accidentally deletes another, unrelated piece of information.
- **Example:** If you delete the last student enrolled in the "Advanced SQL" course, you accidentally delete the record that the "Advanced SQL" course even exists!

---

## 2. Why This Concept Exists

Anomalies are the symptoms of **Redundancy** (storing the same data twice). Normalization is the "medicine" we use to cure these symptoms by splitting tables and building relationships.

---

## 3. How It Works (The Redundancy Trap - PRO LEVEL)

Redundancy consumes disk space, but more importantly, it creates **Cognitive Load** for the developer. 
- In a "Flat" table, the developer must remember to update every occurrence of a value.
- In a "Normalized" table, the value exists in **one place only**. Changing it once updates it for the whole system.

---

## 4. Real-Life Examples

**The "Employee-Dept" Mess:**
| Emp_Name | Dept_Name | Dept_Head |
|---|---|---|
| John | IT | Mr. Smith |
| Sara | IT | Mr. Smith |

If Mr. Smith retires, you must update 2 rows. If the IT department has 5,000 employees, you must update 5,000 rows. This is an **Update Anomaly**.

---

## 5. Mini Practice Tasks

*   **Task 1:** Name and briefly explain the three types of database anomalies.
*   **Task 2:** If deleting a row for a `Sale` causes you to lose the only record of a `Customer`'s phone number, which anomaly has occurred?

---
