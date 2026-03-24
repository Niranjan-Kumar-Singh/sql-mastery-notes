# 🖼️ Topic 13.8: Window Framing (ROWS vs. RANGE)

Standard `PARTITION BY` aggregates the whole group. But sometimes you need the aggregation to "slide" row-by-row. This is called **Window Framing**.

---

## 1. Definition

**The "Frame"** is a specified subset of the current partition. It defines exactly which neighbors of the current row should be included in the calculation.

---

## 2. Syntax: The Frame Clause

The frame is defined at the end of the `OVER()` clause:
`ROWS BETWEEN <start> AND <end>`

**The Keywords:**
- `UNBOUNDED PRECEDING`: From the very beginning of the partition.
- `n PRECEDING`: From `n` rows before current.
- `CURRENT ROW`: The row the pointer is on.
- `n FOLLOWING`: To `n` rows after current.
- `UNBOUNDED FOLLOWING`: To the very end of the partition.

---

## 3. Why This Concept Exists

*   **Moving Averages:** *"Look at 3 rows before and 3 rows after."*
*   **Running Totals:** *"Look from the start up to the current row."*
*   **Delta Checks:** *"Compare me to only the row immediately before."*

---

## 4. ROWS vs. RANGE (PRO LEVEL)

This is the most advanced concept in Windowing:
- **`ROWS`:** Treats everything based on the **Row Number**. It doesn't care about values. (Physical distance).
- **`RANGE`:** Treats everything based on the **Logical Value**. (Logical distance).

**Example:** If two transactions have the same `Date`. 
- `ROWS BETWEEN 1 PRECEDING AND CURRENT ROW`: Only includes the single physical row before it.
- `RANGE BETWEEN 1 PRECEDING AND CURRENT ROW`: Includes **all** rows that share that date value.

**Performance Fact:** `ROWS` is faster because MySQL just counts rows. `RANGE` requires value comparison.

---

## 5. Standard Presets

If you don't define a frame, but you HAVE an `ORDER BY`:
Default = `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

**Senior Pro-Tip:** This default often causes poor performance. Explicitly defining `ROWS` is often better for predictability.

---

## 6. Real-Life Examples

**Calculating a 7-Day Revenue Smooth:**
```sql
SELECT 
    Date, Revenue,
    AVG(Revenue) OVER(ORDER BY Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS 7Day_Smooth
FROM Sales;
```

**Finding the Max Price since the "Start of Time":**
```sql
SELECT 
    Product_Name, Price,
    MAX(Price) OVER(ORDER BY Price DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
FROM Products;
```

---

## 7. Mini Practice Tasks

*   **Task 1:** What is the difference between `ROWS` and `RANGE` in a window frame?
*   **Task 2:** Write a frame clause that includes the current row and the two rows that follow it.

---
