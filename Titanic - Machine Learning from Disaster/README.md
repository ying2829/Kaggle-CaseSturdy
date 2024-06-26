# Kaggle-CaseSturdy -  Titanic - Machine Learning from Disaster

Compared to predict the accuracy of the table, I would like to do the other training by using this two tables which is find the similarity. Let's say we supposed that the test table should be identical to the train table. However, someone did it by mistakes, so we need to measure the similarity to predict if the passengers in the test spreadsheet survived or not. So, are you ready? Here we go:


Create a view which we can find the similarity via the same ticket number between train and test table. Also, I put the filter about the SibSp and Parch column is 0.
Becasue I realized that there are lots of family ticket which will miss up the data in the train table.
```mysql
SELECT Ticket,`case-study-titanic.Kaggle_casestudy.train`Survived,`case-study-titanic.Kaggle_casestudy.train`. PassengerId AS train_passenger,`Kaggle_casestudy.test`.PassengerId AS test_passenger,`case-study-titanic.Kaggle_casestudy.train`.Name AS train_name,`Kaggle_casestudy.test`. Name AS test_name, `case-study-titanic.Kaggle_casestudy.train`.Age AS train_age, `Kaggle_casestudy.test`.Age AS test_age,`case-study-titanic.Kaggle_casestudy.train`.sex AS train_sex,`Kaggle_casestudy.test`. sex AS test_sex,`case-study-titanic.Kaggle_casestudy.train`.Embarked AS train_embarked,`Kaggle_casestudy.test`. Embarked AS test_embarked
FROM `case-study-titanic.Kaggle_casestudy.train` 
INNER JOIN `Kaggle_casestudy.test` USING (ticket)
WHERE `case-study-titanic.Kaggle_casestudy.train`.SibSp=0 AND `case-study-titanic.Kaggle_casestudy.train`.Parch=0
ORDER BY Ticket
```
And then I created a table to calculate the simiarity about the data I compared to the train table.

```mysql
CREATE OR REPLACE TABLE `Kaggle_casestudy.predition` AS
WITH raw_data AS (
  SELECT Survived,train_name,test_name, train_age,test_age,train_sex,test_sex,train_embarked,test_embarked,test_passenger
FROM `Kaggle_casestudy.single_list_byticket`),
split_name AS (
  SELECT Survived,train_name,test_name,test_passenger,
REGEXP_EXTRACT(train_name, r'^\s*(.*?),\s*.*') AS train_fname,
REGEXP_EXTRACT(train_name,  r',\s*(.*?)\s*$') AS train_lname,
REGEXP_EXTRACT(test_name, r'^\s*(.*?),\s*.*') AS test_fname,
REGEXP_EXTRACT(test_name, r',\s*(.*?)\s*$') AS test_lname,
train_age,test_age,train_sex,test_sex,train_embarked,test_embarked
FROM raw_data),
final_data AS (
  SELECT 
Survived,train_name,test_name,test_passenger,
IF(split_name.train_fname=split_name.test_fname,"1","0") AS first_name,
IF(split_name.train_lname=split_name.test_lname,"1","0") AS last_name,
train_age,test_age,train_sex,test_sex,train_embarked,test_embarked
FROM split_name),
tf_data AS ( 
SELECT Survived, first_name,last_name,test_passenger,
IF(train_age=test_age, "1","0") AS age,
IF (train_sex=test_sex,"1","0") AS sex,
IF (train_embarked=test_embarked, "1","0") AS embarked
FROM final_data)
SELECT *
FROM tf_data
```
However, for the prediction table, I have no idea why they keep metioning my data type for first_name,last_name,sex,age and embarked columns are wrong.
So, I cast the data type, and then I sum those columns to find the similarity. Therefore, I can delete more similar data and find the more accurate ones.
```mysql
WITH data_type AS (SELECT 
Survived, test_passenger, 
CAST (first_name AS INT64 ) AS first_name,
CAST(last_name AS INT64) AS last_name,
CAST(sex AS INT64) AS sex,
CAST(age AS INT64) AS age,
CAST(embarked AS INT64) AS embarked
FROM `Kaggle_casestudy.predition`)
SELECT 
Survived, test_passenger,
CONCAT(ROUND((first_name+last_name+sex+age+embarked)/5*100),"%") AS similarity
FROM data_type
ORDER BY test_passenger
```
Therefore, we will get the table that we can filter lots of data by the similarity. And, this also form our own predition.
<img width="659" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/14279e01-708b-45f6-9b44-5cdbbc3ed8c5">


In this part, I prefer to use excel to compel the data.

<img width="205" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/ee77ff0e-3d21-4b58-8fa6-f25811f1d23d">
