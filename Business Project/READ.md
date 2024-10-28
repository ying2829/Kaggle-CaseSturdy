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
![image](https://github.com/user-attachments/assets/fd2bbad9-5633-4fa1-8e67-5afe55847f2f)

**Monthly Comparisons**

* **June**: exhibited the highest variance percentage at 4.16%, indicating that the changes during this month were significant in relation to the overall metric.
* **July**: experienced the highest variance but a lower percentage, suggesting that while there were larger fluctuations, these may have been in a more stable context compared to June.
* **August**: saw a slight decline in both variance and variance percentage, indicating a potential stabilization in the measured metric after the fluctuations observed in the earlier months.

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
![image](https://github.com/user-attachments/assets/e56d96d5-5a76-4f8e-9adc-474ff6da33ad)

**Monthly Comparisons**

* **October**: presented moderate fluctuations with a variance percentage of 1.50%.
* **November**: showed a clear peak in both variance and percentage, highlighting a month of significant change that may warrant further investigation.
* **December**: displayed a stark decline in both metrics, indicating a return to more stable conditions as the year concluded.

* Top 3 profitable months in 2025
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
HAVING year="2025"
ORDER BY variance DESC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/a948c28d-0f73-4ac2-a0e1-ce145e33024b)

**Monthly Comparisons**

* **January to February**: The data shows an increase in negative variance, indicating a worsening situation or increased downward pressure.
**February to March**: The trend continues, with March exhibiting the steepest decline in both variance and variance percentage, reinforcing the notion of heightened volatility during this period.

Top 3 unprofitable months in 2025
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
HAVING year="2025"
ORDER BY variance ASC
LIMIT 3
```
![image](https://github.com/user-attachments/assets/26fe4dc1-1241-4738-a18a-d8951c9e7c5e)

**Monthly Comparisons**
* **April**: The variance indicates a negative trend, suggesting that performance metrics fell short of expectations.
* **May**: The variance worsened in May compared to April, showing a decline in performance. The increase in the negative variance percentage suggests that issues affecting performance became more pronounced.
* **June**: June reflects the most significant negative variance among the three months analyzed. This ongoing decline raises concerns about sustained performance and may warrant further investigation into the underlying causes.

**Conclusion**
* Consistent Decline: There is a clear trend of increasing negative variances over the first half of 2025, with each month showing a worsening situation.
* Rising Variance Percentages: The variance percentages demonstrate that not only is the performance declining, but the rate of decline is also accelerating.

### Inspiration

>[!NOTE]
>From the raw data, although we have whole year data but it is crossed between 2024 and 2025.
>The raw data is updated in June which means the major reservation didn't happen yet.

1. Market Trends and Seasonal Travel Behavior: The summer season presents an opportune time for travel. An analysis of variance and variance percentages from June to December 2024 reveals significant trends in performance metrics. Notably, July exhibits a high variance, while December indicates a return to stability. This suggests that fluctuations during the summer months are more pronounced than those experienced in the winter. Observational data indicates that individuals are more likely to take vacations and travel during the summer, whereas winter months typically see increased home stay for celebrations like Thanksgiving and Christmas. This seasonal behavior strongly influences market dynamics and performance.
2. Impact of Pre-Booking Reservations: Two key pieces of evidence support the significance of pre-booking reservations. Firstly, the performance trend in 2024 shows a consistent decline, which may correlate with reservation behaviors, particularly since the raw data was updated in June. According to market ecology principles, earlier reservations typically yield lower property prices. Additionally, a comparison between data from June 2024 and June 2025 clearly demonstrates that price fluctuations during the same period are inversely related. This provides compelling evidence to explain the differences observed between these two years.

![image](https://github.com/user-attachments/assets/9c1dc434-adde-4199-852f-5e5769316665)

