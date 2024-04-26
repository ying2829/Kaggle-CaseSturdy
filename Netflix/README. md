## Welcome to the Kaggle-Case Sturdy - Netflix

I found there is interesting data from Kaggle, combined with a couple of questions. 
So, let's imagine I am new in the BI team of Netflix. And, I got the data to analyze a couple of questions below:
1. What is the average IMDB score for each genre?
```BigQuery
SELECT Genre,ROUND(AVG(IMDB_Score)) AS avg_IMDBScore
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
GROUP BY Genre
```
<img width="236" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/1653d12f-ca26-4abc-a2a6-e865fa6bf47c">

2. Which genre has the highest average runtime?

In this case, I generate the temporary table about the average runtime for each genre. And, then I found the highest average runtime is 149.
```BigQuery
WITH cte AS (SELECT Genre,ROUND(Avg(Runtime)) AS avg_runtime
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
GROUP BY Genre)
SELECT MAX(cte.avg_runtime)
FROM cte
```

<img width="113" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/3bdbdba1-e8f3-46a2-a9ec-9eaa5ab8ee17">

Then following up this table, I change it slightly to see the name of the genre.

```BigQuery
WITH cte AS (SELECT Genre,ROUND(Avg(Runtime)) AS avg_runtime
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
GROUP BY Genre)
SELECT Genre
FROM cte
WHERE avg_runtime=149
```
<img width="167" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/0cf8ab07-cc59-4d48-a29b-e033ee5fa2ab">

Therefore, we can say both Anthology/ Dark comedy anf Heist film/ Thriller are highest average runtime movie. And, the average runtime is 149 mins.

3. Are longer movies rated higher on IMDB?
```BigqQuery
SELECT CORR(Runtime,IMDB_Score)
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
```
After the correlation coefficients calculation, we can find the value is -0.04. Therefore, we can say this statement is false because there is non-relation correlation.

4. What is the distribution of movies across different languages?

I have different angles to answer this question, please see the following:

a) The number of different languages(multiple language or single language)
```BigQuery
WITH cte AS (SELECT 
CASE WHEN REGEXP_CONTAINS(Language, r"/") THEN "multiple language"
ELSE "single language"
END AS catergory
FROM `netflix-421114.CaseStudy_Netflix.Netflix`)
SELECT catergory,COUNT(*)
FROM cte
GROUP BY catergory
```
<img width="283" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/63baa8f4-f19a-4708-9d10-88dbabf529f9">

b) Language distribution by genre
```SQL
WITH cte AS (SELECT Genre,
CASE WHEN REGEXP_CONTAINS(Language, r"/") THEN "multiple language"
ELSE "single language"
END AS catergory
FROM `netflix-421114.CaseStudy_Netflix.Netflix`)
SELECT Genre,catergory,COUNT(*)
FROM cte
GROUP BY Genre,catergory
```
<img width="431" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/f4107150-c9eb-427b-9fdc-a2b2a5713418">

5. What is the trend in the number of releases over the years?
```SQL
WITH cte AS (SELECT 
Title,Genre,REGEXP_EXTRACT(Premiere, r'\w+ \d+, (\d+)') AS year_premiere,
Runtime,IMDB_Score,Language
FROM `netflix-421114.CaseStudy_Netflix.Netflix`)
SELECT year_premiere,COUNT(*)
FROM cte
GROUP BY cte.year_premiere
HAVING cte.year_premiere IS NOT NULL
ORDER BY cte.year_premiere
```
<img width="284" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/04e0a214-1935-4070-9403-412f190b3ec5">

6. Which year had the highest average IMDB score?
```SQL
WITH cte AS (SELECT 
Title,Genre,REGEXP_EXTRACT(Premiere, r'\w+ \d+, (\d+)') AS year_premiere,
Runtime,IMDB_Score,Language
FROM `netflix-421114.CaseStudy_Netflix.Netflix`)
SELECT year_premiere,AVG(IMDB_Score) AS avg_score
FROM cte
GROUP BY cte.year_premiere
HAVING cte.year_premiere IS NOT NULL
ORDER BY AVG(IMDB_Score) DESC
```
Therefore, the highest average was in 2015.

7. Are movies in certain languages rated higher on average?

First, I use this to fetch the average of IMDB score.
```BigQuery
SELECT AVG(IMDB_Score)
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
```
Then I use the following query to filter the average IMDB score of each language that lower than the average.
```BigQuery
SELECT Language,AVG(IMDB_Score) AS avg_score
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
GROUP BY Language
HAVING AVG(IMDB_Score)>6.27
ORDER BY Language,AVG(IMDB_Score) DESC
```
<img width="289" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/cf8609d3-2810-45fa-a352-8bedaff7a502">

Therefore, from the data, we can say that as long as the movie has English as a language can have at least 7.
8. What is the most common genre on Netflix?
```SQL
SELECT Genre, COUNT (*) AS amount
FROM `netflix-421114.CaseStudy_Netflix.Netflix`
GROUP BY Genre
ORDER BY COUNT (*) DESC
```
<img width="284" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/8fa16bb4-8fac-4808-9bff-6486920f0213">

8. What is the average runtime of movies released in different months?
```SQL
WITH cte AS (SELECT 
Title,Genre,REGEXP_EXTRACT(Premiere, r'(\w+) \d+, \d+') AS month_premiere,
Runtime,IMDB_Score,Language
FROM `netflix-421114.CaseStudy_Netflix.Netflix`),
cte1 AS (SELECT Title,Genre,
 CASE WHEN cte.month_premiere = 'January' THEN 1
    WHEN cte.month_premiere = 'February' THEN 2
    WHEN cte.month_premiere = 'March' THEN 3
    WHEN cte.month_premiere = 'April' THEN 4
    WHEN cte.month_premiere = 'May' THEN 5
    WHEN cte.month_premiere = 'June' THEN 6
    WHEN cte.month_premiere = 'July' THEN 7
    WHEN cte.month_premiere = 'August' THEN 8
    WHEN cte.month_premiere = 'September' THEN 9
    WHEN cte.month_premiere = 'October' THEN 10
    WHEN cte.month_premiere = 'November' THEN 11
    WHEN cte.month_premiere = 'December' THEN 12
    ELSE NULL
    END AS month,Runtime,IMDB_Score,Language
    FROM cte)
    SELECT month,ROUND(AVG(Runtime))AS avg_runtime
    FROM cte1
    GROUP BY month
    HAVING month IS NOT NULL
    ORDER BY month
```
<img width="231" alt="image" src="https://github.com/ying2829/Kaggle-CaseSturdy/assets/162821565/b2f25a3b-84b9-4518-bf65-91d62b5a67f1">

