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
