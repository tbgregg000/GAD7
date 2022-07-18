# Take Home Test
### Part 1
To make this as easy to review as possible, I created a Jupyter Notebook in Kaggle that provides the entire analysis.
* Here is the link to the Notebook: **https://www.kaggle.com/thomgregg/gad-7-take-home/edit**

To execute the code, please follow the below steps in order:
1. Create a copy of the notebook using the Copy & Edit button on the top right of the page
2. To add the data file to the workbook, select the '|<' button on the top right corner of the notebook and add the csv file  
![image](https://user-images.githubusercontent.com/106389348/179556064-0c203d39-25e1-4036-98b5-381bb89da2b9.png)

4. Copy the path to the '.csv' file (you will have to expand phq-all-final first)  
![image](https://user-images.githubusercontent.com/106389348/179555561-773b6706-c0ae-44aa-8f12-23b8555f7f58.png)
4. Replace the string of code below with an updated file path. Just paste the '.csv' file path into the code below.
```
phq_uncleaned = pd.read_csv('../input/phq-all-final/phq_all_final.csv')
```
5. Click Run All and the workbook should update promptly


### Part 2

1. How many users completed an exercise in their first month per monthly cohort?
* We have two ways to perform this task, depending on which version of SQL we are using

Version 1 (without Date_Trunc available):

```
SELECT
CAST(CONCAT(YEAR(u.created_at),'-',MONTH(u.created_at),'-01') as DATE) as created_month_cohort
  , COUNT(e.user_id)/COUNT(u.user_id) as cohort_first_month_exercise_completion_rate
FROM users u 
LEFT JOIN exercises e on e.user_id = u.user_id 
	AND CAST(CONCAT(YEAR(u.created_at),'-',MONTH(u.created_at),'-01') as DATE) = CAST(CONCAT(YEAR(e.exercise_completion_date),'-',MONTH(e.exercise_completion_date),'-01') as DATE)
GROUP BY 1;
```

Version 2 (with Date_Trunc):

```
SELECT
DATE_TRUNC(u.created_at, MONTH) as created_month_cohort
  , COUNT(e.user_id)/COUNT(u.user_id) as cohort_first_month_exercise_completion_rate
FROM users u 
LEFT JOIN exercises e on e.user_id = u.user_id 
	AND DATE_TRUNC(u.created_at, MONTH) = DATE_TRUNC(e.exercise_completion_date, MONTH)
GROUP BY 1;
```


2. How many users completed a given amount of exercises?
```
WITH user_exercise_totals AS (
SELECT
u.user_id
, COUNT(e.exercise_id) as exercise_id_count
FROM users u 
LEFT JOIN exercises e on e.user_id = u.user_id 
GROUP BY 1
)
SELECT
COUNT(CASE WHEN exercise_id_count >= 0 then user_id ELSE NULL END) AS 'all_participants'
, COUNT(CASE WHEN exercise_id_count >= 1 then user_id ELSE NULL END) as 'exercises_1'
, COUNT(CASE WHEN exercise_id_count >= 10 then user_id ELSE NULL END) as 'exercises_10'
, COUNT(CASE WHEN exercise_id_count >= 100 then user_id ELSE NULL END) as 'exercises_100'
FROM user_exercise_totals;
```

3. Which organizations have the most severe patient population?
```
SELECT
organization_name
, organization_id
, sum(score) / count(patient_id) as AverageScore
FROM providers pr
LEFT JOIN phq9 ph ON ph.provider_id = pr.provider_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;
```
