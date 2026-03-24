# 🗝️ Topic 2.6: Composite Key — When One Column Isn't Enough

A student with ID `101` can enroll in Course `'MATH101'`. The same student can also enroll in `'PHY101'`. There is no single column that makes each enrollment row unique — but the *combination* of `Student_ID + Course_ID` is always unique. This is a **Composite Key**.

---

## 1. Definition

A **Composite Key** (also called a **Compound Key** or **Composite Primary Key**) is a Primary Key made up of **two or more columns** that together uniquely identify each row in a table.

No single column alone is unique, but the **combination** of the selected columns is always unique.

---

## 2. Why This Concept Exists

Some entities by their very nature don't have a single natural identifier. Consider:

- **Enrollment:** A student can enroll in many courses. A course can have many students. The unique "fact" is the specific `(Student + Course)` pairing.
- **Flight Seat:** Seat `14A` exists on every flight. Seat `14A` on `Flight_Number 'AI202'` on `2024-12-01` is unique. Three columns together form the key.

---

## 3. When to Use It

Use a Composite Key when you are modeling **Many-to-Many relationships** (junction/bridge tables):

- Students ↔ Courses → `Enrollments` table
- Products ↔ Orders → `Order_Items` table
- Users ↔ Roles → `User_Roles` table

---

## 4. Syntax / Implementation

```sql
-- Junction table for a student-course enrollment
CREATE TABLE Enrollments (
    Student_ID  INT     NOT NULL,
    Course_ID   VARCHAR(20) NOT NULL,
    Enroll_Date DATE,
    Grade       CHAR(1),

    -- The COMBINATION of both columns is the Primary Key
    PRIMARY KEY (Student_ID, Course_ID),

    -- Plus the Foreign Key links
    FOREIGN KEY (Student_ID) REFERENCES Students(Student_ID),
    FOREIGN KEY (Course_ID)  REFERENCES Courses(Course_ID)
);
```

**What this enforces:**
- A student can enroll in many courses. ✅
- A course can have many students. ✅
- The same student **cannot** enroll in the **same course twice**. ❌ (Duplicate PK blocked)

---

## 5. Real-Life Example

**The `Order_Items` table in an e-commerce store:**

| `Order_ID` | `Product_ID` | `Quantity` | `Price` |
|---|---|---|---|
| 1001 | 5 | 2 | 29.99 |
| 1001 | 8 | 1 | 49.99 |
| 1002 | 5 | 3 | 29.99 |

Here, `Order_ID` alone is not unique (Order 1001 has 2 rows). `Product_ID` alone is not unique (Product 5 appears twice). But the combination of `(Order_ID, Product_ID)` is always unique — a composite primary key is perfect here.

---

## 6. Composite Key vs. Surrogate Key

A common design debate:

| Approach | Pros | Cons |
|---|---|---|
| **Composite PK** (Natural) | No extra column needed; enforces business rule at DB level. | Harder to reference as a FK in other tables. |
| **Surrogate PK** (Artificial `INT AUTO_INCREMENT`) | Simple single-column FK references. | Requires a separate `UNIQUE` constraint to enforce business uniqueness. |

**Senior Engineer Recommendation:** For simple junction tables, a Composite PK is clean and sufficient. For complex hierarchies where the junction table itself is heavily referenced by other tables, add an `AUTO_INCREMENT` surrogate and add the natural columns as `UNIQUE`.

---

## 7. Common Mistakes

- **Order matters for performance:** In a Composite PK `(A, B)`, MySQL creates an index starting with `A` first. Queries filtering on `A` alone will use the index. Queries filtering on `B` alone will NOT use the index. Put the most frequently filtered column first.
- **Updating a composite PK:** Since the PK is used in Foreign Key references, updating it cascades across multiple tables — this is rarely a good idea. Prefer using a surrogate key if updates are frequent.

---

## 8. Mini Practice Tasks

- **Task 1:** Design and write the SQL for a `User_Roles` junction table linking `Users(User_ID)` and `Roles(Role_ID)`. Use a Composite Primary Key.
- **Task 2:** Can a Composite Key column contain `NULL` values? (Hint: think about what a Primary Key requires.)

---
