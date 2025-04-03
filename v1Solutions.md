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

# **V2 Solution using SubQuery** 

```sql
SELECT 
    r.id,
    CONCAT(r.last_name, ', ', r.first_name) AS "Full Name",
    r.city,
    r.start_date,
    CASE 
        WHEN next_residency.start_date IS NULL THEN NULL
        ELSE DATEADD(DAY, -1, next_residency.start_date) 
    END AS end_date,
    CONCAT(
        FORMAT(r.start_date, 'MMMM dd, dddd, yyyy'), ' - ',
        COALESCE(FORMAT(DATEADD(DAY, -1, next_residency.start_date), 'MMMM dd, dddd, yyyy'), 'Now')
    ) AS period
FROM residency_history r
LEFT JOIN residency_history next_residency
    ON r.first_name = next_residency.first_name
    AND r.last_name = next_residency.last_name
    AND r.start_date < next_residency.start_date
ORDER BY r.first_name, r.last_name, r.start_date;
```
## **Explanation**
This solution uses a self-join to the `residency_history` table, linking each residency record with the next one for the same person. The `CASE` statement handles the calculation of the `end_date` and the period column is formatted similarly to the first solution.
