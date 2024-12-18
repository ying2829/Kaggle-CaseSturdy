 # Northern Lights Air
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=4&pageSize=5)

This is an analysis report for the 2018 promotion plan in the __Northern Light Air__ . And this report we are going to answer the belowing:
1. What impact did the campaign have on loyalty program memberships (Gross/Net)?
2. Was the campaign adpotion more successful for certain demographic of loyalty program?
3. What impact did the campaign have on booked flights during summer?

Also, this is the report which more focus on the query writing and analysis. I will also create a PowerPoint report for reference. Please follow me more via LinkedIn.

## Revenue Impact

__By Promotion time__

First of all, I use `rollup` function to know the total revenue of the CLV which is 133710161.32000037. And if we only observe the case in the surface, we can simply divided the reveue by `enrollment_type` which is standard and the 2018 promotion.
```bigquery
SELECT enrollment_type,SUM (CLV) AS total_revenue
FROM `Airline_Loyalty_Program.airline_loyalty_history`
GROUP BY enrollment_type
```
![image](https://github.com/user-attachments/assets/77a5140b-5a2d-421c-a00d-545dfece1350)

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

Based on the data, the promotion contributed 5.84% of the total revenue. Additionally, we can categorize the standard promotion type into two phases—before and after the promotion—to analyze its impact from an advertising perspective. Notably, even after the promotion period, approximately 11% of customers opted to sign up for membership.

However, it is important to note that this analysis cannot reliably identify trends due to the significant time gap between the standard plan and the promotion plan. To address this limitation, I strongly recommend adopting a comparative analysis method that evaluates the same time period with and without the promotion in place:

__By YOY__

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

## Other Category Impact

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

For the salary, I segmented the figure into every 10k as a group. And, since I put the salary into the range, it would be easier to observe the data from a macroscopic perspective. Then the same as all the analysis, I put this promotion into the comparsion of the same period last year.

>[!NOTE]
>There is some negative figure in the salary column. So, we also need to change this data to positive as well. 

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
SELECT range_salary, cte2.total_revenue AS standard,cte3.total_revenue AS promotion, ROUND(cte3.total_revenue-cte2.total_revenue,2) AS variance,
CONCAT(ROUND((cte3.total_revenue-cte2.total_revenue)/cte2.total_revenue*100,2),"%") AS percentage_variance
FROM cte2 AS cte2
JOIN cte2 AS cte3 USING (range_salary)
WHERE cte2.enrollment_type = "Standard"
AND cte3.enrollment_type = "2018 Promotion"
```
![image](https://github.com/user-attachments/assets/939775ce-ad9f-4857-bf7d-0dbdd6f2f8fb)
![Picture1](https://github.com/user-attachments/assets/b27831a7-ce72-4f5f-9ebf-d1623841798d)

* __Analysis__
From the chart, several key insights can be observed. Firstly, the salary range of 260k to 270k exhibits the highest percentage variance, followed by the range of 280k to 290k as the second highest. However, there is no clear trend indicating that higher earnings correlate with greater project success, despite the high variance observed in the lower salary ranges.

## Promotion within or witout summertime

* Total Booking Flights
```bigquery
WITH cte AS (SELECT flight.year,SUM (total_flights) AS total_flights
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_flight_activity` WHERE month IN (6,7,8)) AS flight
JOIN `Airline_Loyalty_Program.airline_loyalty_program_date` AS loyalty USING (loyalty_number)
GROUP BY year)
SELECT 
cte_2017.total_flights AS total_flight_2017,
cte_2018.total_flights AS total_flights_2018,
ROUND (cte_2018.total_flights-cte_2017.total_flights,2) AS variance,
CONCAT(ROUND((cte_2018.total_flights - cte_2017.total_flights) / cte_2017.total_flights * 100, 2), "%") AS percentage_variance
FROM cte AS cte_2017
JOIN cte AS cte_2018 ON cte_2017.year = 2017 AND cte_2018.year = 2018
```
![image](https://github.com/user-attachments/assets/d98e52a1-aa99-4d23-979a-0b984c362353)

Compared to the previous summer, there is a variance of approximately 50%. To accurately assess the profitability generated by this promotion, it is essential to distinguish between the promotion and non-promotion periods in 2018. By doing so, the proportion of revenue attributable to the promotion can be calculated more effectively, allowing for a clearer understanding of its impact on total booking flights.
```bigquery
SELECT flight.year,enrollment_type,SUM (total_flights) AS total_flights
FROM 
(SELECT * FROM`Airline_Loyalty_Program.airline_flight_activity` WHERE month IN (6,7,8)) AS flight
JOIN `Airline_Loyalty_Program.airline_loyalty_program_date` AS loyalty USING (loyalty_number)
GROUP BY year,enrollment_type
```
![image](https://github.com/user-attachments/assets/d884c247-f3b4-40a1-b52a-5c0f8edc9fc1)

The comparison between the total variance and the total booking flights generated during the promotion period indicates that the promotion accounted for approximately 75% of the observed variance.

* Revenue Impact
```bigquery
WITH cte AS (SELECT year,ROUND (SUM(CLV),2) AS total_revenue
FROM (SELECT * FROM `Airline_Loyalty_Program.airline_flight_activity` WHERE month IN (6,7,8))
JOIN  `Airline_Loyalty_Program.airline_loyalty_program_date` USING (loyalty_number)
GROUP BY year)
SELECT 
cte1.total_revenue AS total_revenue_2017,
cte2.total_revenue AS total_revenue_2018,
ROUND (cte2.total_revenue-cte1.total_revenue,2) AS variance,
CONCAT(ROUND((cte2.total_revenue-cte1.total_revenue)/cte1.total_revenue*100,2),"%") AS percentage_variance
FROM cte AS cte1
JOIN cte AS cte2 ON cte1.year = 2017 AND cte2.year = 2018
```
![image](https://github.com/user-attachments/assets/7dc0d9af-4d57-477f-b39f-b9c6a4d57102)
```bigquery
SELECT year,enrollment_type,ROUND (SUM(CLV),2) AS total_revenue
FROM (SELECT * FROM `Airline_Loyalty_Program.airline_flight_activity` WHERE year IN (2017,2018) AND month IN (6,7,8))
JOIN  `Airline_Loyalty_Program.airline_loyalty_program_date` USING (loyalty_number)
GROUP BY year,enrollment_type
```
![image](https://github.com/user-attachments/assets/fa49a43c-b3bf-45bb-ae7c-e41bfba3ff04)

From a profitability perspective, the promotion contributed to a 6.19% revenue growth compared to the previous summer. Additionally, since the 2017 revenue aligns closely with the 2018 revenue excluding the promotion, it can be inferred that the promotion directly accounted for the entirety of the observed revenue increase.

