# Airline Satifactioon
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=2&pageSize=20)

This is the report to analyze the customer satisfaction scores from 120,000+ airline passengers. And, this report is aimed to answer the following:

1. Which percentage of airline passengers are satisfied? Does it vary by customer type? What about type of travel?
2. What is the customer profile for a repeating airline passenger?
3. Does flight distance affect customer preferences or flight patterns?
4. Which factors contribute to customer satisfaction the most? What about dissatisfaction?

## Satifaction demography
In total, we have 129880 customers involved in the research.

* __Overall__
```Bigqery
SELECT Satisfaction,COUNT(Satisfaction) AS count_satifaction, CONCAT(ROUND(COUNT(Satisfaction)/129880*100,2),"%") AS percentage_of_satification
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Satisfaction
```
![image](https://github.com/user-attachments/assets/2da552d0-a239-4ba6-96a0-83e2f1504282)

The majority of respondents (56.55%) reported being neutral or dissatisfied, while 43.45% expressed satisfaction.

* __Customer_type__
```Bigquery
SELECT Customer_Type,Satisfaction,COUNT(Satisfaction) AS count_satifaction, CONCAT(ROUND(COUNT(Satisfaction)/129880*100,2),"%") AS percentage_of_satification
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Customer_Type,Satisfaction
```
![image](https://github.com/user-attachments/assets/8970444a-0b65-4d2e-bc9c-05e4004c0a92)

The major `Customer_Type` of the respondents is returing, which occupied 81.69%, while 18.31%  `Customer_Type` is first time. However, it seems that there is a trend that the "first time" `Customer_Type` tends to have higher satifaction then the one with the returing in `Customer_Type`.

* __Type_of_Travel__
```Bigquery
SELECT Type_of_Travel,Satisfaction,COUNT(Satisfaction) AS count_satifaction, CONCAT(ROUND(COUNT(Satisfaction)/129880*100,2),"%") AS percentage_of_satification
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Type_of_Travel,Satisfaction
```
![image](https://github.com/user-attachments/assets/b76b2064-a214-4c5f-8aa4-45f82aed0de6)

The major `type_of_travel` of the respendents is for business purpose which occupied 69.06% compared to 30.94% for the customers for the personal purpose. However, the customer for business purpose tends to have higher percentage of satifaction than the one for the personal purpose. 

## Respendents demography

* __Gender__
```Bigquery
SELECT Gender,COUNT(Gender) AS count_gender, CONCAT(ROUND(COUNT(Gender)/129880*100,2),"%") AS percentage_of_gender
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Gender
```
![image](https://github.com/user-attachments/assets/bdb414ff-cc6b-4121-b46c-0afd6f27c147)
* __Class__
```Bigquery
SELECT Class,COUNT(Class) AS count_class,CONCAT(ROUND(COUNT(Class)/129880*100,2),"%") AS percentage_class
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Class
```
![image](https://github.com/user-attachments/assets/2db8dac4-b41e-4ef9-9ab8-a48cdd75f524)
* __Age Group__
```Bigquery
WITH cte AS (SELECT FLOOR(Age / 10) * 10 AS nearest_10_l,Age,CEILING(Age / 10) * 10 AS nearest_10_h
FROM `Airline_satifcation.airline_passenger_satifaction`),
cte1 AS (SELECT IF(nearest_10_l = nearest_10_h, nearest_10_l - 10, nearest_10_l) AS adjusted_nearest_10_l,Age,nearest_10_h
FROM cte)
SELECT CONCAT(adjusted_nearest_10_l," ~ ",nearest_10_h) AS age_range,COUNT(Age) AS count_age,
CONCAT(ROUND(COUNT(Age)/129880*100,2),"%") AS percentage_of_age
FROM cte1
GROUP BY age_range
ORDER BY age_range
```
![image](https://github.com/user-attachments/assets/6c67038f-9fa4-4bf2-aa9a-4c41811f208a)

The analysis of the respondents indicates that the majority of participants are economy and business-class customers. Their age predominantly falls within the working-age range of 20 to 50 years. Additionally, the gender distribution is relatively balanced, with no significant disparity observed between male and female participants.

## Distance & customer perference

* __By Class__
```Bigquery
WITH cte AS (SELECT *,Flight_Distance,
CASE WHEN Flight_Distance<500 THEN "below_500"
WHEN Flight_Distance BETWEEN 500 AND 1000 THEN "500~1000"
WHEN Flight_Distance BETWEEN 1001 AND 2000 THEN "1001~2000"
WHEN Flight_Distance BETWEEN 2001 AND 3000 THEN "2001~3000"
WHEN Flight_Distance BETWEEN 3001 AND 4000 THEN "3001~4000"
WHEN Flight_Distance BETWEEN 4001 AND 5000 THEN "4001~5000"
END AS distance_range
FROM `Airline_satifcation.airline_passenger_satifaction`)
SELECT Class,distance_range, COUNT(distance_range) AS total_distance
FROM cte
GROUP BY Class,distance_range
ORDER BY distance_range,Class
```
![image](https://github.com/user-attachments/assets/e1730d80-473e-47c6-a44a-4dd64f73b348)
* __By Gender__
```Bigquery
WITH cte AS (SELECT *,Flight_Distance,
CASE WHEN Flight_Distance<500 THEN "below_500"
WHEN Flight_Distance BETWEEN 500 AND 1000 THEN "500~1000"
WHEN Flight_Distance BETWEEN 1001 AND 2000 THEN "1001~2000"
WHEN Flight_Distance BETWEEN 2001 AND 3000 THEN "2001~3000"
WHEN Flight_Distance BETWEEN 3001 AND 4000 THEN "3001~4000"
WHEN Flight_Distance BETWEEN 4001 AND 5000 THEN "4001~5000"
END AS distance_range
FROM `Airline_satifcation.airline_passenger_satifaction`)
SELECT Gender,distance_range, COUNT(distance_range) AS total_distance
FROM cte
GROUP BY Gender,distance_range
ORDER BY distance_range,Gender
```
![image](https://github.com/user-attachments/assets/043ee2d9-62aa-40b0-899f-935de8515f3e)

## Key Factor affecting the satisfaction

```Bigquery
WITH cte AS (SELECT ID, Gender,Age,Customer_Type,Type_of_Travel,Class,Flight_Distance,Departure_Delay,Arrival_Delay,
Ease_of_Online_Booking+Check_in_Service+Online_Boarding+Gate_Location+On_board_Service+Seat_Comfort+Leg_Room_Service+Cleanliness+Food_and_Drink+In_flight_Service+In_flight_Wifi_Service+In_flight_Entertainment+Baggage_Handling AS total_grade,Satisfaction
FROM `Airline_satifcation.airline_passenger_satisfaction`)
SELECT
```

