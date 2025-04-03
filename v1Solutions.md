# **V1 Solution using CTE** 

---

```sql
WITH ResidencyPeriods AS (
    SELECT 
        id,
        first_name,
        last_name,
        city,
        start_date,
        LEAD(start_date) OVER (
            PARTITION BY first_name, last_name 
            ORDER BY start_date
        ) AS next_start_date
    FROM residency_history
)
SELECT 
    id,
    CONCAT(last_name, ', ', first_name) AS "Full Name",
    city,
    start_date,
    CASE 
        WHEN next_start_date IS NULL THEN NULL 
        ELSE DATEADD(DAY, -1, next_start_date) 
    END AS end_date,
    CONCAT(
        FORMAT(start_date, 'MMMM dd, dddd, yyyy'), ' - ',
        COALESCE(FORMAT(DATEADD(DAY, -1, next_start_date), 'MMMM dd, dddd, yyyy'), 'Now')
    ) AS period
FROM ResidencyPeriods
ORDER BY first_name, last_name, start_date;
```

---

## **Explanation**
This solution uses a **Common Table Expression (CTE)** to get the next start_date for each person using the `LEAD()` function.
The query then calculates the `end_date` by subtracting one day from the `next_start_date`. If there is no next residency, the `end_date` will be `NULL`, indicating the person is still living in that city.
The `period` is formatted as specified in the requirements.
