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
