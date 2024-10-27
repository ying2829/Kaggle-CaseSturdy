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

In which month is there likely to be the biggest difference between the float price and the initial price?
