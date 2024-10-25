# Seattle Airbnb Report

## Busiest time

# Monthly
``` Bigquey
SELECT 
    REGEXP_EXTRACT(CAST(date AS STRING),  r'\d+-(\d+)-\d+') AS month, 
    COUNT(*) AS total_visit
FROM (SELECT * FROM`airbnb_seattle.calendar` WHERE available=FALSE)
GROUP BY REGEXP_EXTRACT(CAST(date AS STRING), r'\d+-(\d+)-\d+')
ORDER BY total_visit DESC LIMIT 1
```
![image](https://github.com/user-attachments/assets/1bb7afbf-46ac-4b20-b2f3-82ac491d786a)
