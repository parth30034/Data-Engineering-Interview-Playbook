# Progressive SQL + PySpark Problem Set

This document contains progressively harder problems to practice SQL and PySpark, building step by step.  

---

## 🟦 SQL Problems

### Level 1 (Warm-up)
**Problem:** Find the **3rd highest salary per department** and return ties.  
- Hint: Use `DENSE_RANK()`.

---

### Level 2 (Join + Window Combo)
**Problem:** For each customer, find their **latest order amount**.  
- Hint: Use `ROW_NUMBER() OVER (PARTITION BY cust_id ORDER BY order_date DESC)`.

---

### Level 3 (Conditional Aggregation + Window)
**Problem:** Website Logs  
- For each day, compute:  
  1. Daily active users (distinct).  
  2. % of premium users.  
  3. Rolling 3-day active user count.  

- Hint: Use `COUNT(DISTINCT)`, `SUM(CASE…)`, window with `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`.

---

### Level 4 (Incremental Processing)
**Problem:** You have a `transactions` table with daily loads. Each day’s file may contain updates.  
- Requirement: Keep only the **latest record per transaction_id** based on `updated_at`.  
- Hint: Use `ROW_NUMBER()` over transaction_id ordered by updated_at DESC → filter `rn=1`.

---

### Level 5 (Analytical BI Problem)
**Problem:** Sessionization in Web Logs  
- Users generate events (user_id, ts). Define a new session when the gap between consecutive events > 30 minutes.  
- Output: user_id, session_id, session_start, session_end.  

- Hint: Use `LAG(ts)` window and a running sum of “is_new_session.”

---

## 🟩 PySpark Problems

### Level 1 (Warm-up)
**Problem:** Find the **3rd highest salary per department** and return ties.  
- Hint: Use `dense_rank().over(window)` and filter `rnk = 3`.

---

### Level 2 (Join + Window Combo)
**Problem:** For each customer, find their **latest order amount**.  
- Hint: Use Window partitioned by cust_id, ordered by order_date DESC.

---

### Level 3 (Conditional Aggregation + Window)
**Problem:** Website Logs  
- For each day, compute:  
  1. Daily active users (distinct).  
  2. % of premium users.  
  3. Rolling 3-day active user count.  

- Hint: Use `countDistinct`, `when`, and `Window.orderBy("log_date").rowsBetween(-2, 0)`.

---

### Level 4 (Incremental Processing)
**Problem:** You have a `transactions` DataFrame with daily loads. Each day’s file may contain updates.  
- Requirement: Keep only the **latest record per transaction_id** based on `updated_at`.  
- Hint: Use `row_number` window ordered by updated_at DESC.

---

### Level 5 (Analytical BI Problem)
**Problem:** Sessionization in Web Logs  
- Users generate events (user_id, ts). Define a new session when the gap between consecutive events > 30 minutes.  
- Output: user_id, session_id, session_start, session_end.  

- Hint: Use `lag`, `when`, and `sum().over()` to assign session ids.

---
