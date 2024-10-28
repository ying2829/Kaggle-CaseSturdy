# Seattle Airbnb Report

## Busiest time

### TOP 3 Monthly for 2024
``` Bigquey
SELECT 
    REGEXP_EXTRACT(CAST(date AS STRING),  r'(\d+)-\d+-\d+') AS year,REGEXP_EXTRACT(CAST(date AS STRING),  r'\d+-(\d+)-\d+') AS month,
    COUNT(*) AS total_transactions
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE)
GROUP BY year,month
HAVING year="2024"
ORDER BY total_transactions DESC 
LIMIT 3
```
![image](https://github.com/user-attachments/assets/8f520ed8-dd8e-4d16-a711-3dc89ae823c2)

### TOP 3 Monthly for 2025
```Bigquery
SELECT 
    REGEXP_EXTRACT(CAST(date AS STRING),  r'(\d+)-\d+-\d+') AS year,REGEXP_EXTRACT(CAST(date AS STRING),  r'\d+-(\d+)-\d+') AS month,
    COUNT(*) AS total_transactions
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE)
GROUP BY year,month
HAVING year="2025"
ORDER BY total_transactions DESC 
LIMIT 3
```
![image](https://github.com/user-attachments/assets/ac9f2a0d-a803-40a1-ba94-79581a5e4d01)

## Price Spike

### Price Spike for 2024
```Bigquery
WITH cte AS (SELECT  REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'(\d+)-\d+-\d+') AS year,
REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'\d+-(\d+)-\d+') AS month,
listing_id,calendar.date, `airbnb_seattle.listings`.price AS initial_price,calendar.price AS floart_price, 
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE) AS calendar
JOIN `airbnb_seattle.listings` ON calendar.listing_id=`airbnb_seattle.listings`.id)
SELECT cte.year,cte.month,cte.initial_price
FROM cte
WHERE cte.year="2024"
ORDER by cte.initial_price DESC
LIMIT 1
```
![image](https://github.com/user-attachments/assets/5eef9c61-01a1-48b2-b204-ff79757dcbd2)

### Price Spike for 2025
```Bigquery
WITH cte AS (SELECT  REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'(\d+)-\d+-\d+') AS year,
REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'\d+-(\d+)-\d+') AS month,
listing_id,calendar.date, `airbnb_seattle.listings`.price AS initial_price,calendar.price AS floart_price, 
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE) AS calendar
JOIN `airbnb_seattle.listings` ON calendar.listing_id=`airbnb_seattle.listings`.id)
SELECT cte.year,cte.month,cte.initial_price
FROM cte
WHERE cte.year="2025"
ORDER by cte.initial_price DESC
LIMIT 1
```
![image](https://github.com/user-attachments/assets/2fb1042a-a67a-4a8d-ac5a-71a8daf70b49)

### Analysis
Which month is/isn't the most profitable? (The difference between initial price and float price)
To analyze this question, first, we should eliminate the null price in the listing table because it will mess up the report.

* TOP 3 profitable month in 2024
```Bigquery
WITH cte AS (SELECT REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'(\d+)-\d+-\d+') AS year,
REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'\d+-(\d+)-\d+') AS month,
listing_id,calendar.date, `airbnb_seattle.listings`.price AS initial_price,calendar.price AS float_price, 
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE) AS calendar
JOIN `airbnb_seattle.listings` ON calendar.listing_id=`airbnb_seattle.listings`.id)
SELECT year, month, 
AVG(ROUND(cte.float_price - cte.initial_price, 2)) AS variance,
AVG(ROUND((cte.float_price - cte.initial_price) / cte.initial_price * 100, 2)) AS variance_percentage
FROM cte
GROUP BY year, month
HAVING year="2024"
ORDER BY variance DESC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/1c31f2b8-50d6-4cd6-8ac7-244675e1317e)

**Monthly Comparisons**

* **June**: exhibited the highest variance percentage at 4.16%, indicating that the changes during this month were significant in relation to the overall metric.
* **July**: experienced the highest variance but a lower percentage, suggesting that while there were larger fluctuations, these may have been in a more stable context compared to June.
* **August**: saw a slight decline in both variance and variance percentage, indicating a potential stabilization in the measured metric after the fluctuations observed in the earlier months.

**Conclusion**: The analysis of variance and variance percentage from June to August 2024 reveals interesting dynamics in performance. While July showed the largest absolute variance, June had the most significant relative impact. The slight decline in August suggests a potential stabilization following the earlier volatility.

* TOP 3 unprofitable month in 2024
```Bigquery
WITH cte AS (SELECT REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'(\d+)-\d+-\d+') AS year,
REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'\d+-(\d+)-\d+') AS month,
listing_id,calendar.date, `airbnb_seattle.listings`.price AS initial_price,calendar.price AS float_price, 
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE) AS calendar
JOIN `airbnb_seattle.listings` ON calendar.listing_id=`airbnb_seattle.listings`.id)
SELECT year, month, 
AVG(ROUND(cte.float_price - cte.initial_price, 2)) AS variance,
AVG(ROUND((cte.float_price - cte.initial_price) / cte.initial_price * 100, 2)) AS variance_percentage
FROM cte
GROUP BY year, month
HAVING year="2024"
ORDER BY variance ASC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/cf838cc2-dea4-4932-b75e-4e3a8151a8b8)

**Monthly Comparisons**

* **October**: presented moderate fluctuations with a variance percentage of 1.50%.
* **November**: showed a clear peak in both variance and percentage, highlighting a month of significant change that may warrant further investigation.
* **December**: displayed a stark decline in both metrics, indicating a return to more stable conditions as the year concluded.

**Conclusion**
The analysis of variance and variance percentage from October to December 2024 reveals critical insights into performance dynamics in the final quarter of the year. November stands out as a month of notable fluctuation, while December reflects a return to stability.

* Compared to the data in 2025

In which month is there likely to be the biggest difference between the float price and the initial price?
