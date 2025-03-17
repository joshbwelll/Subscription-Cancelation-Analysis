# Subscription-Cancelation-Analysis


```sql
with yearly as (SELECT
    date_trunc('year', cancel_date::date) as cancel_year,
    cancelation_reason,
    count(*) as num_reason
FROM junk.all_cancel_reasons
GROUP BY 1,2
)
SELECT
    cancel_year,
    cancelation_reason,
    num_reason,
    SUM(num_reason)OVER(PARTITION BY cancel_year) as year_total,
    num_reason * 100 / year_total as perc_reas_ann
FROM yearly
ORDER BY cancel_year ASC
```
