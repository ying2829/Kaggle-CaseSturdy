 # Northern Lights Air
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=4&pageSize=5)

This is an analysis report for the __Northern Light Air__ . And this report we are going to answer the belowing:
1. What impact did tje campaign have on loyalty program memberships (Gross/Net)?
2. Was the campaign adpotion more successful for certain demographic of loyalty program?
3. What impact did the campaign have on booked flights during summer?

Also, this is the report which more focus on the query writing and analysis. I will also create a dashboard in Power BI and attach it to here for reference.

First of all, I use `rollup` function to know the total revenue of the CLV which is 133710161.32000037. And if we only observe the case in the surface, we can simply divided the reveue by `enrollment_type` which is standard and the 2018 promotion.
```bigquery
SELECT enrollment_type,SUM (CLV) AS total_revenue
FROM `Airline_Loyalty_Program.airline_loyalty_history`
GROUP BY enrollment_type
```
![image](https://github.com/user-attachments/assets/77a5140b-5a2d-421c-a00d-545dfece1350)

Therefore, the promtion achieve 5.84% revenue of the total reveue. However, if we divied the data by time, then it would be more clear about the resultof the promotion.
```bigquery
WITH cte AS (SELECT loyalty_number,enrollment_year,enrollment_month,cancellation_year,cancellation_month,
CAST(CONCAT(enrollment_year,'-',enrollment_month,'-','01')AS DATE) AS enrollment_date,
CAST(CONCAT(cancellation_year,'-',cancellation_month,'-','01') AS DATE) AS cancellation_date,
CLV
FROM `Airline_Loyalty_Program.airline_loyalty_history`),
cte1 AS (SELECT *,
CASE WHEN cte.enrollment_date<'2018-02-01'THEN 'ahead_promotion'
WHEN cte.enrollment_date BETWEEN '2018-02-01' AND '2018-04-01' THEN 'promotion'
WHEN cte.enrollment_date >'2018-04-01' THEN 'sub_promotion'
END AS category
FROM cte),
cte2 AS (SELECT category, SUM(CLV) AS total_revenue
FROM cte1
GROUP BY category)
SELECT category,total_revenue,
CONCAT(ROUND(cte2.total_revenue/133710161.32000037*100,2),"%") AS percentage_of_total
FROM cte2
ORDER BY category
```
![image](https://github.com/user-attachments/assets/b1c1543e-1553-447a-b79f-268a6435ce71)


## YOY ##

* __The same period during the promotion__
 ```bigquery
WITH cte AS (SELECT loyalty_number,enrollment_year,enrollment_month,cancellation_year,cancellation_month,
CAST(CONCAT(enrollment_year, '-', enrollment_month, '-01') AS DATE) AS enrollment_date,
CAST(CONCAT(cancellation_year, '-', cancellation_month, '-01') AS DATE) AS cancellation_date,
CLV
FROM `Airline_Loyalty_Program.airline_loyalty_history`),
cte1 AS (SELECT *,
CASE WHEN enrollment_date BETWEEN '2018-02-01' AND '2018-04-01' THEN 'after_promotion'
WHEN enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' THEN 'yoy_after_promotion'
ELSE NULL
END AS category
FROM cte),
cte2 AS (SELECT category,ROUND(SUM(CLV),2)AS total_revenue,LAG(ROUND(SUM(CLV),2))OVER(ORDER BY category DESC) AS per_revenue
FROM cte1
GROUP BY category
HAVING category IS NOT NULL
ORDER BY category DESC)
SELECT ROUND (cte2.total_revenue-per_revenue,2) AS varaince, CONCAT(ROUND((cte2.total_revenue-per_revenue)/per_revenue*100,2),"%") AS percentage_variance
FROM cte2
```
![image](https://github.com/user-attachments/assets/489bb745-db5d-4c42-8088-0cf635022a50)

* __The same period after the promotion__

```bigquery
WITH cte AS (SELECT loyalty_number,enrollment_year,enrollment_month,cancellation_year,cancellation_month,
CAST(CONCAT(enrollment_year, '-', enrollment_month, '-01') AS DATE) AS enrollment_date,
CAST(CONCAT(cancellation_year, '-', cancellation_month, '-01') AS DATE) AS cancellation_date,
CLV
FROM `Airline_Loyalty_Program.airline_loyalty_history`),
cte1 AS (SELECT *,
CASE WHEN enrollment_date BETWEEN '2018-05-01' AND '2018-12-01' THEN 'after_promotion'
WHEN enrollment_date BETWEEN '2017-05-01' AND '2017-12-01' THEN 'yoy_after_promotion'
ELSE NULL
END AS category
FROM cte),
cte2 AS (SELECT category,ROUND(SUM(CLV),2)AS total_revenue,LAG(ROUND(SUM(CLV),2))OVER(ORDER BY category DESC) AS per_revenue
FROM cte1
GROUP BY category
HAVING category IS NOT NULL
ORDER BY category DESC)
SELECT ROUND (cte2.total_revenue-per_revenue,2) AS varaince, CONCAT(ROUND((cte2.total_revenue-per_revenue)/per_revenue*100,2),"%") AS percentage_variance
FROM cte2 
```

![image](https://github.com/user-attachments/assets/31550198-aaf2-4bca-82fd-1f89e15a6656)

* __Analyse__

Rather than using data segmented by pre- and post-promotion periods to assess the results of the loyalty program, I prefer to use a year-over-year (YOY) comparison. As shown in the first table, the promotion contributed 67.08% of the revenue compared to the same period last year. Furthermore, the program demonstrated significant advertising effectiveness, as evidenced by the 12.58% revenue generated after the promotion, attributed to continued membership enrollment.

2. Was the campaign adoption more successful for certain demographics of loyalty members?
