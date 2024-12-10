# Airline Satifactioon
[Data Source](https://www.mavenanalytics.io/data-playground?order=date_added%2Cdesc&page=2&pageSize=20)

This is the report to analyze the customer satisfaction scores from 120,000+ airline passengers. And, this report is aimed to answer the following:

1. Which percentage of airline passengers are satisfied? Does it vary by customer type? What about type of travel?
2. What is the customer profile for a repeating airline passenger?
3. Does flight distance affect customer preferences or flight patterns?
4. Which factors contribute to customer satisfaction the most? What about dissatisfaction?

## Satifaction demography
In total, we have 129880 customers involved in the research.
__Overall__
```Bigqery
SELECT Satisfaction,COUNT(Satisfaction) AS count_satifaction, CONCAT(ROUND(COUNT(Satisfaction)/129880*100,2),"%") AS percentage_of_satification
FROM `Airline_satifcation.airline_passenger_satifaction`
GROUP BY Satisfaction
```
