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

* TOP 3 profitable month
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
ORDER BY variance DESC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/1c31f2b8-50d6-4cd6-8ac7-244675e1317e)

The data from June to August 2024 reveals a remarkable increaseing in the `variance_percentage`, and it all the month are in summer, suggesting a significant ongoing change or instability. For these summer season, 

* TOP 5 unprofitable month
```Bigquery
WITH cte AS (SELECT REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'(\d+)-\d+-\d+') AS year,
REGEXP_EXTRACT(CAST(calendar.date AS STRING),  r'\d+-(\d+)-\d+') AS month,
listing_id,calendar.date, `airbnb_seattle.listings`.price AS initial_price,calendar.price AS float_price, 
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE) AS calendar
JOIN `airbnb_seattle.listings` ON calendar.listing_id=`airbnb_seattle.listings`.id)
SELECT year,month, MAX(ROUND(cte.initial_price - cte.float_price, 2)) AS variance,
MAX (CONCAT(ROUND((cte.float_price-cte.initial_price)/cte.initial_price*100, 2),"%")) AS variance_percentage
FROM cte
GROUP BY year,month
ORDER BY variance DESC
LIMIT 5
```
![image](https://github.com/user-attachments/assets/1dfd1b3a-e71e-4cc6-be37-e4d93f34d000)

f

In which month is there likely to be the biggest difference between the float price and the initial price?
