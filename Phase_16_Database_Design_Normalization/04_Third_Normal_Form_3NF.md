# 🏛️ Topic 16.4: 3rd Normal Form (3NF)

Even if a table is in 2NF, it can still have "hidden" redundancy through **Transitive Dependencies**. **3rd Normal Form (3NF)** is the industry standard for "Clean enough for production."

---

## 1. Definition

A table is in **3NF** if:
1.  It is already in **2NF**.
2.  It has **No Transitive Functional Dependencies**.

**What is a Transitive Dependency?**
It occurs when Column A (PK) determines Column B, and Column B determines Column C. Effectively, A determines C through B. 
**Rule:** Non-key columns should only depend on the Primary Key, and nothing else!

---

## 2. Why This Concept Exists

Transitive dependencies create "Ghost Relationships."
- **Example:** In an `Employees` table: `Employee_ID` -> `Zip_Code` -> `City`.
- If you know the `Zip_Code`, you already know the `City`. So storing both next to the employee ID is redundant.

---

## 3. How It Works (The "Nothing but the Key" Rule - PRO LEVEL)

A famous developer slogan for 3NF: *"The data depends on the Key, the Whole Key, and **Nothing But The Key**, so help me Codd."*

**Before 3NF:**
| Emp_ID (PK) | Name | ZipCode | City |
|---|---|---|---|
| 1 | John | 10001 | New York |
| 2 | Sara | 10001 | New York |

**Fix (3NF):**
1. Move `ZipCode` and `City` to a separate `Geography` table.
2. Keep only `ZipCode` in the `Employees` table as a foreign key.

---

## 4. Real-Life Examples

**The "Product-Brand" Mess:**
| Product_ID (PK) | Model | Brand_ID | Brand_Phone_Support |
|---|---|---|---|
| 1 | iPhone 15 | Apple | 1-800-APPLE |

Here, `Brand_Phone_Support` depends on `Brand_ID`, not the product. Move it to a `Brands` table!

---

## 5. When is it 3NF? (The Checklist)

1. Is every row unique? (PK exists)
2. Are all columns atomic? (1NF)
3. Does every non-key column depend on the whole primary key? (2NF)
4. Do any non-key columns depend on other non-key columns? (If no, then 3NF!)

---

## 6. Mini Practice Tasks

*   **Task 1:** Explain the concept of "Transitive Dependency" using an example.
*   **Task 2:** If a `Cars` table has `Car_ID`, `Model_Name`, and `Manufacturer_Country`, which normal form does it likely violate?

---
