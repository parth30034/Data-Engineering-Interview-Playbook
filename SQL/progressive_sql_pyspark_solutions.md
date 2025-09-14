# Progressive SQL + PySpark Solutions

This file contains reference solutions for the progressive problem set.  

---

## 🟦 SQL Solutions

### Level 1 (3rd highest salary per department)
```sql
SELECT emp_id, emp_name, dept_id, salary
FROM (
    SELECT emp_id, emp_name, dept_id, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 3;
```

---

### Level 2 (Latest order per customer)
```sql
SELECT cust_id, order_id, amount, order_date
FROM (
    SELECT o.*, 
           ROW_NUMBER() OVER (PARTITION BY cust_id ORDER BY order_date DESC) AS rn
    FROM orders o
) t
WHERE rn = 1;
```

---

### Level 3 (Website Logs Aggregation)
```sql
SELECT 
    log_date,
    COUNT(DISTINCT user_id) AS daily_active_users,
    100.0 * COUNT(DISTINCT CASE WHEN user_type='premium' THEN user_id END) / COUNT(DISTINCT user_id) AS pct_premium_users,
    SUM(COUNT(DISTINCT user_id)) OVER (ORDER BY log_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_3day_users
FROM web_logs
GROUP BY log_date
ORDER BY log_date;
```

---

### Level 4 (Incremental Processing)
```sql
SELECT transaction_id, updated_at, amount
FROM (
    SELECT t.*, 
           ROW_NUMBER() OVER (PARTITION BY transaction_id ORDER BY updated_at DESC) AS rn
    FROM transactions t
) x
WHERE rn = 1;
```

---

### Level 5 (Sessionization)
```sql
WITH logs AS (
  SELECT user_id, ts,
         LAG(ts) OVER (PARTITION BY user_id ORDER BY ts) AS prev_ts
  FROM web_logs
),
flagged AS (
  SELECT user_id, ts,
         CASE WHEN prev_ts IS NULL OR ts - prev_ts > INTERVAL '30 minutes' THEN 1 ELSE 0 END AS new_session
  FROM logs
),
sessions AS (
  SELECT user_id, ts,
         SUM(new_session) OVER (PARTITION BY user_id ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_id
  FROM flagged
)
SELECT user_id, session_id, MIN(ts) AS session_start, MAX(ts) AS session_end
FROM sessions
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

---

## 🟩 PySpark Solutions

### Level 1 (3rd highest salary per department)
```python
from pyspark.sql import Window
from pyspark.sql.functions import dense_rank, desc

w = Window.partitionBy("dept_id").orderBy(desc("salary"))
df_result = employees_df.withColumn("rnk", dense_rank().over(w)).filter("rnk = 3")
```

---

### Level 2 (Latest order per customer)
```python
from pyspark.sql.functions import row_number

w = Window.partitionBy("cust_id").orderBy(desc("order_date"))
df_result = orders_df.withColumn("rn", row_number().over(w)).filter("rn = 1")
```

---

### Level 3 (Website Logs Aggregation)
```python
from pyspark.sql.functions import col, countDistinct, when, sum as _sum

w = Window.orderBy("log_date").rowsBetween(-2, 0)
df_result = logs_df.groupBy("log_date")     .agg(
        countDistinct("user_id").alias("daily_active_users"),
        (countDistinct(when(col("user_type")=="premium", col("user_id"))) / countDistinct("user_id") * 100).alias("pct_premium_users")
    )     .withColumn("rolling_3day_users", _sum("daily_active_users").over(w))
```

---

### Level 4 (Incremental Processing)
```python
from pyspark.sql.functions import row_number

w = Window.partitionBy("transaction_id").orderBy(desc("updated_at"))
df_result = transactions_df.withColumn("rn", row_number().over(w)).filter("rn = 1")
```

---

### Level 5 (Sessionization)
```python
from pyspark.sql.functions import lag, when, sum as _sum, col
from pyspark.sql import Window

w = Window.partitionBy("user_id").orderBy("ts")

# Step 1: lag previous timestamp
df_lag = logs_df.withColumn("prev_ts", lag("ts").over(w))

# Step 2: flag new sessions
df_flagged = df_lag.withColumn("new_session",
    when((col("prev_ts").isNull()) | ((col("ts").cast("long") - col("prev_ts").cast("long")) > 1800), 1).otherwise(0)
)

# Step 3: cumulative sum to assign session ids
df_sessions = df_flagged.withColumn("session_id", _sum("new_session").over(w.rowsBetween(Window.unboundedPreceding, 0)))

# Step 4: aggregate session start and end
df_result = df_sessions.groupBy("user_id", "session_id")     .agg(
        F.min("ts").alias("session_start"),
        F.max("ts").alias("session_end")
    )     .orderBy("user_id", "session_id")
```

---
