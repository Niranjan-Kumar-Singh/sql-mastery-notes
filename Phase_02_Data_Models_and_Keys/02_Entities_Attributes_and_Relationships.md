# 🧱 Topic 2.2: Entities, Attributes, and Relationships

These are the three fundamental "bricks" used to build any ER Model. If you master these, you can map any business in the world into a database.

---

## 1. Definitions

1.  **Entity (Rectangle):** A real-world *Noun*. (e.g., Student, Teacher, Course).
2.  **Attribute (Oval):** An *Adjective* or property of an entity. (e.g., Name, Date_of_Birth).
3.  **Relationship (Diamond):** A *Verb* showing how entities interact. (e.g., Student *Enrolls* in Course).

---

## 2. Types of Attributes (PRO LEVEL)

Not all attributes are the same. MySQL engineers distinguish between:

*   **Key Attribute (Underlined):** Uniquely identifies a record (e.g., `_ID_`).
*   **Composite Attribute:** Can be broken down into sub-parts (e.g., `Address` splits into `Street`, `City`, `Zip`).
*   **Multi-valued Attribute (Double Oval):** Can have multiple values for one record (e.g., `Phone_Numbers`). **Pro Hint:** In a real DB, these usually become their own table!
*   **Derived Attribute (Dashed Oval):** Calculated from others (e.g., `Age` comes from `DOB`). **Pro Hint:** We don't store these in MySQL; we calculate them in queries!

---

## 3. Why This Concept Exists

By separating these three, we ensure that data doesn't get "mixed up." For example, a `Student's Address` belongs to the `Student` entity, whereas the `Course Name` belongs to the `Course` entity. This leads to **Normalization**.

---

## 4. Mapping to SQL

*   **Entity** ➔ `CREATE TABLE`
*   **Simple Attribute** ➔ Column (e.g., `VARCHAR`, `INT`)
*   **Relationship** ➔ `FOREIGN KEY` connection

---

## 5. Real-Life Example

**Hospital Scenario:**
*   `[ Patient ]` is the **Entity**.
*   `( Blood_Type )` is an **Attribute**.
*   `< Stays in >` is the **Relationship** connecting them to the `[ Ward ]` Entity.

---

## 6. Mini Practice Tasks

*   **Task 1:** Draw an ER diagram for a "Library" with two entities (`Book`, `Member`) and their relationship.
*   **Task 2:** Is `Total_Price` a stored attribute or a derived attribute if you already have `Quantity` and `Unit_Price`? Should it be saved in the database?

---
