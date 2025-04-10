# Subscription-Cancelation-Analysis-for-SaaS-Company

**Executive Summary:**
This project focuses on analyzing subscription cancellations for a SaaS company. By leveraging SQL and Power BI, I uncover key trends from user-reported cancellation reasons. Facing significant churn and revenue loss, company leadership seeks deeper insights into why customers are leaving to inform future retention strategies. This analysis identifies primary and secondary reasons for cancellations and provides actionable recommendations to improve customer retention.

**Business Problem:**
The company has experienced a substantial increase in churn this year, negatively impacting revenue. As part of a company-wide retention initiative, leadership needs to understand why users are canceling their subscriptions. Currently, there is no detailed reporting on cancellation reasons, so the analytics team has analyzed user-reported data from the cancellation workflow within the product. The goal is to identify key trends and provide data-driven insights to enhance retention strategies.

![image](https://github.com/user-attachments/assets/3c2aae43-3ad4-40d0-8f80-eda5e9666de6)


**Methodology:**
1. EDA: Patterns between cancelation reasons
2. Product Funnel Analysis
3. Data Visualization (Power BI)


**Skills:**
* Data Visualization
    * Power BI
* Data Wrangling
* Data Cleaning
* Data Science Notebook (HEX)
* Snowflake Data warehouse
* SQL – CTEs, CASE, Window function, Union, View creation
* SQL Code used:

Relation between number of subscriptions per cancelation reason
```sql
WITH reasons as (SELECT
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


To produce the data behind a line chart for % Reason by year. If there was more data avalible month would be used
```sql
WITH yearly as (SELECT
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



**Results:**

<img width="1100" alt="image" src="https://github.com/user-attachments/assets/84711d42-4348-41b7-a321-398d7e8a109c" />




**Business Recommendations:**
1. Improve Onboarding & Product Education: Since the most common cancellation reasons are **"Expensive"** and **"Not Useful"**, we should enhance user onboarding and provide better educational resources early in the subscription lifecycle. Ensuring users fully understand and utilize the product's value could reduce price sensitivity and increase engagement.
2. Implement Targeted Retention Offers: For cost-conscious users, we should introduce a retention offer at the beginning of the cancellation process, such as a limited-time discount or extended trial period, to incentivize them to stay.
3. Competitor Analysis & Market Adaptation: As a significant number of users cite **"Went to a Competitor"** as a secondary cancellation reason, we should conduct market research to stay competitive, ensuring our product and pricing remain aligned with industry standards.


**Next Steps:**
* Explore the engagement of canceled subscriptions and see how they interacted with the product. 
   * What features did they or did they not use? How often did they use the product? 
   * What if we compare them to non-cancelled subscriptions. Does anything stand out that could inform future retention efforts?
* Work with the product manager to either understand how we can improve the cancelation workflow by adding in rescue tactics and adding friction without increasing user frustration. Otherwise work with the product manager to explore a reduced cancellation workflow.
* Develop and implement an enhanced onboarding process with guided tutorials and personalized support.
* Test and evaluate discount-based retention offers to measure effectiveness in reducing churn.
* Conduct a competitive analysis to assess market trends, pricing, and feature gaps.
