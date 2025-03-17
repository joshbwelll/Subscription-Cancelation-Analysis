# Subscription-Cancelation-Analysis

**Executive Summary:**
This is a project that was primarily completed in HEX. Facing a significant churn issue and major revenue loss, the company leadership wants deeper insights into the reasons behind subscription cancellations to inform future retention efforts. This analysis utilizing SQL and Power BI uses user-reported cancellation reasons to identify trends and provide business recommendations to support future retention.


**Business Problem:**
Company leadership has noticed a big churn problem this year which has had a large negative impact to revenue, so they're planning a company-wide retention effort. However, we don't currently have any insights or reporting into why people are churning, so the analytics team has decided to analyze the user-reported data collected in the cancelation workflow within the product to see if there are any trends as to why users are cancelling.


**Methodology:**
1. EDA - looking for patterns between the three cancelation reasons
2. Product Funnel Analysis
3. Data Visualization (Power BI)


**Skills:**
Data Visualization
Data Wrangling
Data Cleaning
Data Science Notebook
Snowflake Data warehouse
Power BI
SQL – CTEs, CASE, Union, View creation
SQL Code used:

Wanted to figure out the relation between number of subscriptions per cancelation reason
```sql
with reasons as (SELECT
    subscription_id,
    cancelation_reason1 as cancelation_reason,
    'reason 1' as reason_type

FROM public.cancelations

UNION

SELECT
    subscription_id,
    cancelation_reason2 as cancelation_reason,
    'reason 2' as reason_type
FROM public.cancelations

UNION

SELECT
    subscription_id,
    cancelation_reason3 as cancelation_reason,
    'reason 3' as reason_type
FROM public.cancelations
)

SELECT
    cancelation_reason,
    COUNT(subscription_id) as num_subs
FROM reasons
GROUP BY 1
```


Created a view to store cancelation reason data
```sql
CREATE or replace view junk.all_cancel_reasons AS
SELECT
    subscription_id,
    cancel_date,
    cancelation_reason1 as cancelation_reason,
    1 as reason_number

FROM public.cancelations

UNION

SELECT
    subscription_id,
    cancel_date,
    cancelation_reason2 as cancelation_reason,
    2 as reason_number
FROM public.cancelations

UNION

SELECT
    subscription_id,
    cancel_date,
    cancelation_reason3 as cancelation_reason,
    3 as reason_number
FROM public.cancelations
```


To produce the data behind a line chart for % Reason by year. If there was more data avalible month would be used (project data)
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



**Results & Business Recommendation:**
[Subscription Cancelation Analysis Report](BDE_Visuals.pdf)


**Business Recommendations:**
1. Since most users have selected Expensive and Not Useful as the reasons to cancel, we should roll out better onboarding and provide more help early on in their subscription to ensure users are understanding the product and finding it useful. If they find the product more useful and valuable, they also may become less cost-sensitive to the value.
2. For cost conscious users, we could also launch a rescue tactic at the beginning of the cancellation workflow that offers them a large discount to stay and not cancel their subscription.
Since the most common cancellation reason for the secondary reason is "Went to Competitor", we should research the market and ensure we're keeping up to date with industry trends.

**Next Steps:**
Explore the engagement of canceled subscriptions and see how they interacted with the product. 
What features did they or did they not use?
How often did they use the product? 
What if we compare them to non-cancelled subscriptions. Does anything stand out that could inform future retention efforts?
Work with the product manager to either understand how we can improve the cancelation workflow by adding in rescue tactics and adding friction without increasing user frustration. Otherwise work with the product manager to explore a reduced cancellation workflow.
