# 📦 Topic 13.3: NTILE Bucketing

Ranking gives you a specific number. But sometimes you want to "Band" your data into broad groups — like **Top 25%**, **Middle 50%**, etc. This is called **Bucketing**.

---

## 1. Definition

**`NTILE(n)`:** A window function that divides the sorted rows in a partition into `n` roughly equal groups (tiles) and assigns a group number (1 to `n`) to each row.

---

## 2. Why This Concept Exists

*   **Quartile Analysis:** "Find customers in the top 25% of spenders (Quartile 1)."
*   **Segmentation:** Dividing a list of 1000 leads into 10 groups of 100 for a marketing experiment.
*   **Performance Testing:** Benchmarking the slowest 10% of database queries.

---

## 3. How It Works (The Division Logic - PRO LEVEL)

How does MySQL handle "remainders" (e.g., putting 10 rows into 3 buckets)?
1.  MySQL calculates `Rows / N`.
2.  If there is a remainder, it distributes the extra rows starting from **Bucket 1** downwards.
3.  **Example:** 10 rows into 3 buckets.
    - Bucket 1: 4 rows.
    - Bucket 2: 3 rows.
    - Bucket 3: 3 rows.

---

## 4. Syntax / Implementation

```sql
SELECT 
    Name, 
    Exam_Score,
    NTILE(4) OVER(ORDER BY Exam_Score DESC) AS Quartile
FROM Students;
-- Result:
-- Top 25% get Quartile 1
-- Next 25% get Quartile 2...
```

---

## 5. Real-Life Examples

**The Sales Performance Banding:**
Assign every sales rep to a "Performance Tier" (Tiers 1, 2, 3).
```sql
SELECT 
    Rep_Name, 
    Total_Sales,
    NTILE(3) OVER(ORDER BY Total_Sales DESC) AS Performance_Tier
FROM Sales_Team;
```

---

## 6. Common Mistakes

*   **Confusing `NTILE` with `RANK`:** `RANK` is based on the **value**. `NTILE` is based on **row count**. 
    - If everyone has the same score, `RANK` puts them all at #1. 
    - If everyone has the same score, `NTILE(2)` still splits them into two halves based on their row position.
*   **Invalid Argument:** `NTILE` must take a literal positive integer. You cannot pass a dynamic column or a negative number.

---

## 7. Tips & Best Practices (Pro-Level)

**Percentile Calculation:**
While MySQL doesn't have a native `PERCENTILE` function as simply as others, you can use `NTILE(100)` to get the **Percentile Rank** of any row within your dataset.

---

## 8. Mini Practice Tasks

*   **Task 1:** Use `NTILE(4)` to divide a list of `Employees` into 4 salary groups (Quartiles).
*   **Task 2:** If you have 7 rows and you use `NTILE(3)`, how many rows will be in each bucket?

---
