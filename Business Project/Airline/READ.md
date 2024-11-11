 # Northern Lights Air
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=4&pageSize=5)

This is an analysis report for the __Northern Light Air__ . And this report we are going to answer the belowing:
1. What impact did tje campaign have on loyalty program memberships (Gross/Net)?
2. Was the campaign adpotion more successful for certain demographic of loyalty program?
3. What impact did the campaign have on booked flights during summer?

Also, this is the report which more focus on the query writing and analysis. I will also create a dashboard in Power BI and attach it to here for reference.

```bigquery
WITH cte AS (SELECT loyalty_number,enrollment_year,enrollment_month,cancellation_year,cancellation_month,
CAST(CONCAT(enrollment_year,'-',enrollment_month,'-','01')AS DATE) AS enrollment_date,
CAST(CONCAT(cancellation_year,'-',cancellation_month,'-','01') AS DATE) AS cancellation_date,
CLV
FROM `Airline_Loyalty_Program.airline_loyalty_history`),
cte1 AS (SELECT *,
CASE WHEN cte.enrollment_date<'2018-02-01'THEN 'before_promotion'
WHEN cte.enrollment_date BETWEEN '2018-02-01' AND '2018-04-01' THEN 'under_promotion'
WHEN cte.enrollment_date >'2018-04-01' THEN 'after_promotion'
END AS category
FROM cte),
cte2 AS (SELECT category, SUM(CLV) AS total_revenue
FROM cte1
GROUP BY category)
SELECT category,total_revenue,CONCAT(ROUND(cte2.total_revenue/133710161.32000037*1000,2),"%") AS percentage_of_total
FROM cte2
```
