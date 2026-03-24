# 📅 Topic 8.4: Date & Time Functions

In Phase 3, we learned about Date/Time storage types. Now we learn how to **interact** with them. Whether it's calculating an employee's age, finding orders from the last 30 days, or determining the day of the week, **Date Functions** are the bridge between raw timestamps and human-readable time logic.

---

## 1. Definition

**Date & Time Functions:** Built-in SQL functions that manipulate `DATE`, `DATETIME`, and `TIMESTAMP` columns to calculate durations, extract components (like month or year), and handle timezone-aware interval arithmetic.

---

## 2. The Core Date Functions Reference

| Function | Description | Example | Result |
|---|---|---|---|
| `NOW()` | Current date + time | `NOW()` | '2024-03-15 14:30:10' |
| `CURDATE()` | Current date only | `CURDATE()` | '2024-03-15' |
| `CURTIME()` | Current time only | `CURTIME()` | '14:30:10' |
| `YEAR(d)` | Extracts the year | `YEAR('2024-03-15')`| 2024 |
| `MONTH(d)` | Extracts the month | `MONTH('2024-03-15')`| 03 |
| `DAY(d)` | Extracts the day | `DAY('2024-03-15')`| 15 |
| `DATEDIFF(a, b)` | Difference in **days** | `DATEDIFF('2024-03-20', '2024-03-15')` | 5 |
| `DATE_ADD(d, INTERVAL x)`| Adds time | `DATE_ADD('2024-03-15', INTERVAL 1 MONTH)` | '2024-04-15' |
| `DATE_SUB(d, INTERVAL x)`| Subtracts time | `DATE_SUB('2024-03-15', INTERVAL 7 DAY)` | '2024-03-08' |

---

## 3. Why This Concept Exists

Time is the most common dimension in data analysis ("Sales this week vs last week," "Overdue invoices," "User retention over 30 days"). Because months have different lengths and leap years exist, you can't just add numbers to dates. You need an engine that understands the calendar.

---

## 4. How It Works (TIMESTAMP vs DATETIME & Timezones - PRO LEVEL)

This is a critical architectural choice:

**`NOW()` vs `SYSDATE()`**
- **`NOW()`** returns the time at which the statement **began** executing. In a long query with 1 million rows, `NOW()` will be the same for the first and last row.
- **`SYSDATE()`** returns the **exact moment** it is called. In a long query, `SYSDATE()` will gradually increase as it processes rows. Senior engineers almost always use `NOW()` for consistency.

**Timezone Shift:**
`CURRENT_TIMESTAMP` and `NOW()` rely on the `time_zone` setting of the MySQL session.
```sql
SELECT @@session.time_zone; -- Check current timezone
SET time_zone = '+05:30';    -- Change to IST on the fly
```

---

## 5. Real-Life Examples

**Example 1: The "New Customers" Filter**
Find all users who signed up in the last 7 days:
```sql
SELECT Username, Created_At
FROM Users
WHERE Created_At >= DATE_SUB(NOW(), INTERVAL 7 DAY);
```

**Example 2: Overdue Invoices**
Calculate how many days an invoice is overdue:
```sql
SELECT 
    Invoice_ID, 
    Due_Date, 
    DATEDIFF(CURDATE(), Due_Date) AS Days_Overdue
FROM Invoices
WHERE Due_Date < CURDATE();
```

---

## 6. SQL Examples (MySQL Execution)

```sql
-- 1. Extracting Month and Year for Reports
SELECT 
    YEAR(Order_Date) AS Sales_Year,
    MONTH(Order_Date) AS Sales_Month,
    SUM(Total_Amount) AS Monthly_Revenue
FROM Orders
GROUP BY YEAR(Order_Date), MONTH(Order_Date);

-- 2. Finding Day Name
SELECT DAYNAME('2024-03-15'); -- 'Friday'

-- 3. Calculating Age from Date of Birth
SELECT 
    First_Name, 
    FLOOR(DATEDIFF(CURDATE(), Date_of_Birth) / 365.25) AS Age
FROM Employees;
```

---

## 7. Common Mistakes

*   **`DATEDIFF` argument order:** `DATEDIFF(date1, date2)` returns `date1 - date2`. If `date1` is older than `date2`, the result is negative.
*   **Assuming `DATEDIFF` handles hours:** `DATEDIFF` ignores the time component and only calculates whole days. If you need the difference in seconds or hours, use `TIMESTAMPDIFF(HOUR, date1, date2)`.

---

## 8. Tips & Best Practices (Pro-Level)

**Interval Keyword Flexibility:**
The `INTERVAL` keyword works with many units: `MICROSECOND`, `SECOND`, `MINUTE`, `HOUR`, `DAY`, `WEEK`, `MONTH`, `QUARTER`, `YEAR`. You can even combine them:
```sql
SELECT DATE_ADD(NOW(), INTERVAL '1:30' HOUR_MINUTE);
```

---

## 9. Mini Practice Tasks

*   **Task 1:** Write a query that finds all employees whose `Hire_Date` is exactly 5 years ago from today.
*   **Task 2:** Use `DATE_ADD` to calculate the "Subscription Expiry Date" if the subscription started on `'2024-01-01'` and lasts for 6 months.

---
