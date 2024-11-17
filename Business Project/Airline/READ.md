 # Northern Lights Air
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=4&pageSize=5)

This is an analysis report for the __Northern Light Air__ . And this report we are going to answer the belowing:
1. What impact did the campaign have on loyalty program memberships (Gross/Net)?
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

* __Analysis__

Rather than using data segmented by pre- and post-promotion periods to assess the results of the loyalty program, I prefer to use a year-over-year (YOY) comparison. As shown in the first table, the promotion contributed 67.08% of the revenue compared to the same period last year. Furthermore, the program demonstrated significant advertising effectiveness, as evidenced by the 12.58% revenue generated after the promotion, attributed to continued membership enrollment.

2. Was the campaign adoption more successful for certain demographics of loyalty members?
At this part, if we just divied every data by within and without the promotion, it would be really hard to measure the result of the promotion. So, I suggested we can approach this via the same way which is compared the data in the same period but different year. Therefore, I create a table which combined the `enrollment year` and `enrollment month` together and put it into date data type so that I can filter the date easily.

* __By Geography (`province`)__
```bigquery
WITH cte AS (SELECT province,enrollment_type,SUM(CLV) AS total_transaction
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_loyalty_program_date`
WHERE enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' OR enrollment_date BETWEEN '2018-02-01'AND '2018-04-01')
GROUP BY province,enrollment_type
ORDER BY province,total_transaction),
cte3 AS (SELECT cte1.province, 
cte1.total_transaction AS standard,
cte2.total_transaction AS promotion
FROM cte AS cte1
JOIN cte AS cte2 USING (province)
WHERE cte1.enrollment_type = "Standard"
AND cte2.enrollment_type = "2018 Promotion")
SELECT province,ROUND(standard,2) AS standard,ROUND(cte3.promotion,2) AS promotion,
ROUND(promotion-standard, 2) AS variance,CONCAT(ROUND((promotion-standard)/standard*100,2),"%") AS percantage_variance
FROM cte3
ORDER BY CONCAT(ROUND((promotion-standard)/standard*100,2),"%")
```
![image](https://github.com/user-attachments/assets/d3fe1dc2-8fa7-4c67-9184-981a66ffe0bd)


* __Analysis__

From a geographical perspective, the majority of provinces have experienced impressive growth, with Newfoundland demonstrating a remarkable doubling in growth. Even the provinces with the lowest positive growth have seen an increase of approximately 19%. However, it is important to note that Prince Edward Island, Manitoba, Alberta, and Nova Scotia have not achieved the same level of success.

* __By Gender__
```bigquery
WITH cte AS (SELECT gender,enrollment_type,SUM(CLV) AS total_transaction
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_loyalty_program_date`
WHERE enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' OR enrollment_date BETWEEN '2018-02-01'AND '2018-04-01')
GROUP BY gender,enrollment_type
ORDER BY gender,total_transaction),
cte3 AS (SELECT cte1.gender, 
cte1.total_transaction AS standard,
cte2.total_transaction AS promotion
FROM cte AS cte1
JOIN cte AS cte2 USING (gender)
WHERE cte1.enrollment_type = "Standard"
AND cte2.enrollment_type = "2018 Promotion")
SELECT gender,ROUND(standard,2) AS standard,ROUND(cte3.promotion,2) AS promotion,
ROUND(promotion-standard, 2) AS variance,CONCAT(ROUND((promotion-standard)/standard*100,2),"%") AS percantage_variance
FROM cte3
ORDER BY CONCAT(ROUND((promotion-standard)/standard*100,2),"%") DESC
```
![image](https://github.com/user-attachments/assets/b3446fcf-e602-497b-af82-c3f5a85b3038)

* __Analysis__

  The analysis of the promotion results, segmented by gender, reveals significant growth, and the demographics of male is more successful than female.

 * __By Education__
```bigquery
WITH cte AS (SELECT education,enrollment_type,SUM(CLV) AS total_transaction
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_loyalty_program_date`
WHERE enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' OR enrollment_date BETWEEN '2018-02-01'AND '2018-04-01')
GROUP BY education,enrollment_type
ORDER BY education,total_transaction),
cte3 AS (SELECT cte1.education, 
cte1.total_transaction AS standard,
cte2.total_transaction AS promotion
FROM cte AS cte1
JOIN cte AS cte2 USING (education)
WHERE cte1.enrollment_type = "Standard"
AND cte2.enrollment_type = "2018 Promotion")
SELECT education,ROUND(standard,2) AS standard,ROUND(cte3.promotion,2) AS promotion,
ROUND(promotion-standard, 2) AS variance,CONCAT(ROUND((promotion-standard)/standard*100,2),"%") AS percantage_variance
FROM cte3
ORDER BY CONCAT(ROUND((promotion-standard)/standard*100,2),"%") DESC
```
![image](https://github.com/user-attachments/assets/f0814477-4cec-4765-aabb-67ca33873443)

* __Analysis__
  This promotional campaign has been particularly successful among customers with a bachelor's degree, exhibiting an 87% increase compared to the same period last year. The next most successful demographic includes individuals with a high school or college degree. However, the campaign has shown limited or no impact on customers with master's or doctoral degrees, with some instances even reflecting a decline in engagement.

* __By Marital Status__
```bigquery
WITH cte AS (SELECT marital_status,enrollment_type,SUM(CLV) AS total_transaction
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_loyalty_program_date`
WHERE enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' OR enrollment_date BETWEEN '2018-02-01'AND '2018-04-01')
GROUP BY marital_status,enrollment_type
ORDER BY marital_status,total_transaction),
cte3 AS (SELECT cte1.marital_status, 
cte1.total_transaction AS standard,
cte2.total_transaction AS promotion
FROM cte AS cte1
JOIN cte AS cte2 USING (marital_status)
WHERE cte1.enrollment_type = "Standard"
AND cte2.enrollment_type = "2018 Promotion")
SELECT marital_status,ROUND(standard,2) AS standard,ROUND(cte3.promotion,2) AS promotion,
ROUND(promotion-standard, 2) AS variance,CONCAT(ROUND((promotion-standard)/standard*100,2),"%") AS percantage_variance
FROM cte3
ORDER BY CONCAT(ROUND((promotion-standard)/standard*100,2),"%") DESC
```
![image](https://github.com/user-attachments/assets/570cf845-4206-4a50-982c-f33007d73846)

* __Analysis__
The promotional campaign was effective across all marital status categories, with the divorced demographic showing the strongest growth in both percentage and absolute terms. Future campaigns may benefit from targeting this group more specifically, while continuing to nurture the engagement of both single and married customers.

* __By Salary__
For the salary, I segmented the figure into group and find the median salary.

>[!NOTE]
>There is some negative figure in the salary column. So, we also need to change this data to positive as well.

```bigquery
WITH cte AS (
    SELECT FLOOR(salary/10000)*10000 AS nearest_10_l ,IF(salary<0,0-salary,salary) AS salary,CEILING(salary/10000)*10000 AS nearest_10_h,CLV
    FROM (SELECT * FROM `Airline_Loyalty_Program.airline_loyalty_program_date` WHERE salary IS NOT NULL)
    ),
cte1 AS (SELECT CONCAT(cte.nearest_10_l,"~",cte.nearest_10_h) AS range_salary,salary,CLV
FROM cte)
SELECT range_salary, ROUND(SUM(CLV),2) AS total_revenue
FROM cte1
GROUP BY range_salary
```
Then I need to figure  

```bigquery
WITH cte AS (SELECT CAST(CONCAT(enrollment_year, '-', enrollment_month, '-01') AS DATE) AS enrollment_date,FLOOR(salary/10000)*10000 AS nearest_10_l ,
IF(salary<0,0-salary,salary) AS salary,CEILING(salary/10000)*10000 AS nearest_10_h,CLV,enrollment_type
    FROM (SELECT * FROM `Airline_Loyalty_Program.airline_loyalty_program_date` WHERE salary IS NOT NULL AND enrollment_date BETWEEN '2017-02-01' AND '2017-04-01' OR enrollment_date BETWEEN '2018-02-01'AND '2018-04-01')
    ),
cte1 AS (SELECT enrollment_date,enrollment_type,CONCAT(cte.nearest_10_l,"~",cte.nearest_10_h) AS range_salary,salary,CLV
FROM cte),
cte2 AS (SELECT range_salary, enrollment_type,ROUND(SUM(CLV),2) AS total_revenue
FROM cte1
GROUP BY range_salary,enrollment_type)
SELECT range_salary, 
cte2.total_revenue AS standard,
cte3.total_revenue AS promotion
FROM cte2 AS cte2
JOIN cte2 AS cte3 USING (range_salary)
WHERE cte2.enrollment_type = "Standard"
AND cte2.enrollment_type = "2018 Promotion"
```
