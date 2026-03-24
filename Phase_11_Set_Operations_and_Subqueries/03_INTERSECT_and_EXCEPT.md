# ✂️ Topic 11.3: INTERSECT and EXCEPT

While `UNION` adds sets together, **`INTERSECT`** finds what they have in common, and **`EXCEPT`** (also known as `MINUS`) finds what makes them different.

*Note: For many years, MySQL did not support these natively. Starting in version 8.0.31, native support was added. We will cover both the native syntax and the "Classic Simulation" for older versions.*

---

## 1. Definition

*   **`INTERSECT`:** Returns only the rows that appear in **both** Query A and Query B.
*   **`EXCEPT` (or `MINUS`):** Returns rows from Query A that **do not appear** in Query B.

---

## 2. Why This Concept Exists

*   **`INTERSECT`:** "Find users who are both a Customer and a Supplier."
*   **`EXCEPT`:** "Find customers who have registered but have NEVER placed an order." (Excluded rows).

---

## 3. The MySQL Simulation Patterns (PRO LEVEL)

If you are on a MySQL version older than 8.0.31, you must simulate these using Joins or Subqueries. Every professional should know these patterns:

### Simulating INTERSECT (Using INNER JOIN or IN)
```sql
-- Find common IDs
SELECT ID FROM TableA
INNER JOIN TableB USING (ID);

-- OR
SELECT ID FROM TableA WHERE ID IN (SELECT ID FROM TableB);
```

### Simulating EXCEPT (Using LEFT JOIN or NOT IN)
```sql
-- Find IDs in A but not in B
SELECT a.ID FROM TableA a
LEFT JOIN TableB b ON a.ID = b.ID
WHERE b.ID IS NULL;

-- OR
SELECT ID FROM TableA WHERE ID NOT IN (SELECT ID FROM TableB);
```

---

## 4. Native Syntax (MySQL 8.0.31+)

```sql
-- Finding common emails
SELECT Email FROM Newsletter_Subscribers
INTERSECT
SELECT Email FROM Customer_List;

-- Finding subscribers who are NOT customers
SELECT Email FROM Newsletter_Subscribers
EXCEPT
SELECT Email FROM Customer_List;
```

---

## 5. How It Works (The Hash Match Algorithm)

Internally, `INTERSECT` and `EXCEPT` typically use a **Hash Set**:
1.  MySQL builds a Hash Table for the results of the second query.
2.  It then iterates through the first query.
3.  **For INTERSECT:** If the row exists in the hash table, it returns it.
4.  **For EXCEPT:** If the row does **NOT** exist in the hash table, it returns it.

---

## 6. Common Mistakes

*   **Assuming EXCEPT is symmetric:** `A EXCEPT B` is **NOT** the same as `B EXCEPT A`. The first one finds "A without B," the second finds "B without A."
*   **Handling NULLs:** In Set Operations, `NULL` is treated as a value. Two rows with `NULL` in the same column are considered "Equal" for an `INTERSECT` or `EXCEPT` check, unlike in a Join where `NULL = NULL` is unknown.

---

## 7. Tips & Best Practices (Pro-Level)

**EXCEPT vs. NOT EXISTS:**
While `EXCEPT` is cleaner to read, `NOT EXISTS` (Topic 11.9) is often faster for complex exclusion logic because it allows the optimizer to "short-circuit" as soon as it finds one match in the second table.

---

## 8. Mini Practice Tasks

*   **Task 1:** Using the simulation pattern, write a query to find all `Product_IDs` that exist in the `Inventory` table but NOT in the `Daily_Sales` table.
*   **Task 2:** What is the fundamental difference in how `NULL` is handled between a `JOIN` and an `INTERSECT` operation?

---
