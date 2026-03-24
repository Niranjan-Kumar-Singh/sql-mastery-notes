# ⏭️ Topic 13.4: LEAD and LAG (Positional Functions)

Before MySQL 8.0, comparing a row to its neighboring row required a self-join or a complex correlated subquery — both slow and hard to read. **`LEAD`** and **`LAG`** solve this in a single, clean pass.

---

## 1. Definition

- **`LAG(column, offset, default)`:** Returns the value from a column **N rows behind** the current row within the same window/partition. Looks "backward."
- **`LEAD(column, offset, default)`:** Returns the value from a column **N rows ahead** of the current row. Looks "forward."

Parameters:
- `column` — the column value to retrieve
- `offset` — how many rows back/forward (default is 1)
- `default` — value to return when there's no previous/next row (default is NULL)

---

## 2. Why This Concept Exists

**Time-series and sequential analysis** constantly requires comparing a value to its adjacent rows:
- "How much did revenue grow compared to yesterday?"
- "How long did this session last? (current logout - previous login)"
- "Did the stock price go up or down from the last trading day?"
- "What was the previous status of this order before it changed?"

Before LEAD/LAG, you'd do an ugly self-join like: `ON t1.ID = t2.ID + 1` — which breaks whenever IDs have gaps.

---

## 3. How It Works — Pointer-Based Access (PRO LEVEL)

LEAD and LAG don't perform a new search or join. They work on the **already-sorted window partition in memory**:

1. MySQL sorts the rows within the partition (as specified by `ORDER BY` inside `OVER()`).
2. For each row, it moves a **memory pointer** N positions backward (LAG) or forward (LEAD).
3. It reads the value at that pointer position.

Because it's just pointer arithmetic on an already-sorted result, LEAD and LAG are extremely **fast** — O(1) per row after the initial sort.

---

## 4. Syntax / Implementation — Full Cheat Sheet

```sql
-- Basic LEAD and LAG
SELECT 
    Date,
    Daily_Revenue,
    LAG(Daily_Revenue)  OVER(ORDER BY Date) AS Yesterday_Revenue,
    LEAD(Daily_Revenue) OVER(ORDER BY Date) AS Tomorrow_Revenue
FROM Daily_Sales;

-- With explicit offset (look 2 rows back instead of 1)
SELECT 
    Quarter,
    Revenue,
    LAG(Revenue, 2) OVER(ORDER BY Quarter) AS Revenue_2_Quarters_Ago
FROM Quarterly_Revenue;

-- With default value (show 0 instead of NULL on first/last rows)
SELECT 
    Month,
    Revenue,
    LAG(Revenue, 1, 0) OVER(ORDER BY Month) AS Last_Month_Revenue,
    Revenue - LAG(Revenue, 1, 0) OVER(ORDER BY Month) AS Growth_Amount
FROM Monthly_Sales;

-- With PARTITION BY: separate LAG per entity
SELECT 
    Store_ID,
    Sale_Date,
    Daily_Sales,
    LAG(Daily_Sales) OVER(PARTITION BY Store_ID ORDER BY Sale_Date) AS Prev_Day_Sales,
    Daily_Sales - LAG(Daily_Sales) OVER(PARTITION BY Store_ID ORDER BY Sale_Date) AS Day_Change
FROM Store_Sales;
```

---

## 5. Real-Life Examples

**Example 1 — Month-over-Month Revenue Growth (Finance):**
```sql
SELECT 
    Month_Year,
    Revenue,
    LAG(Revenue) OVER(ORDER BY Month_Year) AS Last_Month,
    Revenue - LAG(Revenue) OVER(ORDER BY Month_Year) AS Dollar_Change,
    ROUND(
        (Revenue - LAG(Revenue) OVER(ORDER BY Month_Year)) / 
        NULLIF(LAG(Revenue) OVER(ORDER BY Month_Year), 0) * 100, 
    2) AS Pct_Change
FROM Monthly_Revenue
ORDER BY Month_Year;
```

**Example 2 — Stock Price Daily Change:**
```sql
SELECT 
    Trade_Date,
    Ticker,
    Closing_Price,
    LAG(Closing_Price) OVER(PARTITION BY Ticker ORDER BY Trade_Date) AS Prev_Close,
    CASE 
        WHEN Closing_Price > LAG(Closing_Price) OVER(PARTITION BY Ticker ORDER BY Trade_Date) THEN '📈 Up'
        WHEN Closing_Price < LAG(Closing_Price) OVER(PARTITION BY Ticker ORDER BY Trade_Date) THEN '📉 Down'
        ELSE '➡️ Unchanged'
    END AS Direction
FROM Stock_Prices;
```

**Example 3 — User Session Duration (LEAD for logout time):**
```sql
SELECT 
    User_ID,
    Event_Type,
    Event_Time,
    LEAD(Event_Time) OVER(PARTITION BY User_ID ORDER BY Event_Time) AS Next_Event_Time,
    TIMESTAMPDIFF(MINUTE, Event_Time, 
        LEAD(Event_Time) OVER(PARTITION BY User_ID ORDER BY Event_Time)
    ) AS Minutes_Until_Next_Event
FROM User_Activity_Log
WHERE Event_Type IN ('Login', 'Logout');
```

---

## 6. Handling NULLs on Boundary Rows

The first row has no "previous" row → `LAG` returns `NULL`.
The last row has no "next" row → `LEAD` returns `NULL`.

```sql
-- Without default: First row shows NULL for LAG
SELECT Date, Revenue, LAG(Revenue) OVER(ORDER BY Date) AS Prev;
-- Result: 2024-01-01 | 5000 | NULL  ← NULL because nothing before it

-- With default value 0:
SELECT Date, Revenue, LAG(Revenue, 1, 0) OVER(ORDER BY Date) AS Prev;
-- Result: 2024-01-01 | 5000 | 0  ← 0 instead of NULL

-- Use COALESCE for custom formatting:
SELECT Date, Revenue, COALESCE(LAG(Revenue) OVER(ORDER BY Date), 'N/A (First Month)') AS Prev;
```

---

## 7. Common Mistakes

- **Missing PARTITION BY in multi-entity reports:** Without `PARTITION BY Store_ID`, LAG for Store B will look at the last row of Store A — completely incorrect data. Always partition when you have multiple entities.
- **Assuming ORDER BY and PARTITION match the outer query sort:** The `ORDER BY` inside `OVER()` is independent of the query's final `ORDER BY`. They can be different.
- **Not handling NULLs from boundary rows:** If you compute `Revenue - LAG(Revenue)` and the first row returns NULL for LAG, the result is NULL (not 0). Use a default value or COALESCE.

---

## 8. Tips & Best Practices

- **Index the column used in `OVER(ORDER BY ...)`** to avoid a full sort (FileSort) on large tables.
- **Use LEAD/LAG over Correlated Subqueries:** The old way of "look at previous row" required `WHERE id = (SELECT MAX(id) FROM t WHERE id < current_id)` — complex and O(N²). LAG/LEAD is O(N log N) — dramatically faster.
- **Store LAG expressions in a CTE** if you need to reuse the same value multiple times:
```sql
WITH Sales_With_Lag AS (
    SELECT Date, Revenue, LAG(Revenue) OVER(ORDER BY Date) AS Prev_Revenue
    FROM Monthly_Sales
)
SELECT Date, Revenue, Prev_Revenue,
       Revenue - Prev_Revenue AS Change,
       ROUND((Revenue - Prev_Revenue)/Prev_Revenue*100, 1) AS Pct
FROM Sales_With_Lag;
```

---

## 9. Mini Practice Tasks

- **Task 1:** Write a query that shows each `Order_Date` and `Total_Amount` for a customer, alongside a `Previous_Order_Amount` column showing the amount from the immediately preceding order (for the SAME customer, using `PARTITION BY`).
- **Task 2:** What happens when `LAG()` is used on the **first row** of a partition? How do you handle it gracefully?
- **Task 3:** Why is `LAG(Revenue, 1, 0) OVER(...)` better than `COALESCE(LAG(Revenue) OVER(...), 0)` in terms of readability? (They produce the same result — which is more idiomatic SQL?)

---
