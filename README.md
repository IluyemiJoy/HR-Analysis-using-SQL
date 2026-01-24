# HR-Analysis-using-SQL

## Project Overview
This analytics project performs a multi-dimensional examination of employee data to understand the composition, distribution, and dynamics of the workforce. The analysis spans across demographic factors, employment history, geographic presence, and organizational structure.

## Objectives
To conduct a comprehensive analysis of workforce demographics, employment patterns, and organizational dynamics within the company to identify trends in employee distribution, turnover, and growth across various dimensions including gender, race, age, location, department, and job roles.

## Project Structure

## 1. Database Setup
* Database Creation: The project starts by creating a database named `human_resources`.
* Table Creation: The dataset was provided from Kaggle and importied successfully into the database and was named hr. The table structure includes columns for emp_id, first name, last  name, birthdate, gender, race, department, jobtitle, location, hiredfate, termination date, location city and location state.

  ```sql
  CREATE DATABASE human_resources;
  ```

## 2. Data Exploration and Cleaning Process
- Change the error in the column names
```sql
ALTER TABLE hr
CHANGE COLUMN ï»¿id emp_id VARCHAR(30) NULL;
```

- Check for duplicacy of empID
```sql
SELECT emp_id, count(*) as count
from hr 
group by emp_id
having count(*) > 1;
```

- To correct errors in birthdate
```sql
UPDATE hr 
SET birthdate = CASE	
	WHEN birthdate LIKE '%/%' THEN DATE_FORMAT(STR_TO_DATE(birthdate, '%m/%d/%Y'), '%Y-%m-%d')
    WHEN birthdate LIKE '%-%' THEN DATE_FORMAT(STR_TO_DATE(birthdate, '%m-%d-%y'), '%Y-%m-%d')
    ELSE NULL
END;
```

- To check for empty cells in birthdate
```sql
SELECT birthdate
from hr 
where birthdate IS NULL;
```
- To change the data type of birthdate
```sql
ALTER TABLE hr
MODIFY COLUMN birthdate DATE;
```

- To correct errors in hiredate
```sql
UPDATE hr 
SET hire_date = CASE	
	WHEN hire_date LIKE '%/%' THEN DATE_FORMAT(STR_TO_DATE(hire_date, '%m/%d/%Y'), '%Y-%m-%d')
    WHEN hire_date LIKE '%-%' THEN DATE_FORMAT(STR_TO_DATE(hire_date, '%m-%d-%y'), '%Y-%m-%d')
    ELSE NULL
END;
```

- To check for null cells in hiredate
```sql
SELECT hire_date
from hr 
where hire_date IS NULL;
```

- To change the data type of hiredate
```sql
ALTER TABLE hr
MODIFY COLUMN hire_date DATE;
```

- To view termdate and check for inconsistencies
```sql
select termdate from hr;
```

- To correct errors in termdate
```sql
UPDATE hr
SET termdate = CASE
	WHEN termdate LIKE '%-%' THEN date_format(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC'), '%Y-%m-%d')
    ELSE NULL
END;
```

- To change the data type of termdate
```sql
ALTER TABLE hr
MODIFY COLUMN termdate DATE;
```

- To regroup employees into active, terminated and future termination
```sql
ALTER TABLE hr
ADD COLUMN EmpStatus Varchar(50) NULL;
```

```sql
UPDATE hr
SET EmpStatus = CASE
	WHEN termdate IS NULL THEN 'Active'
    WHEN termdate > CURDATE() THEN 'Future Termination'
    ELSE 'Terminated'
END;
```

- To add age column
```sql
ALTER TABLE hr 
ADD COLUMN age int;
```

- To calculate the ages
```sql
UPDATE hr
SET age = TIMESTAMPDIFF(YEAR, birthdate, CURDATE());
```

- To remove inconsitencies in the age (cases where ages are less than 18)
```sql
SELECT AGE FROM hr 
WHERE age >= 18;
```

- To add year_month column
```sql
ALTER TABLE hr
ADD COLUMN yearmonth VARCHAR(10) GENERATED ALWAYS AS (date_format(hire_date, '%Y-%m'));
```

- To add year column
```sql
Alter table hr
ADD COLUMN hire_year smallint GENERATED ALWAYS AS (YEAR(hire_date)) STORED;
```

- To ADD AGE GROUP COLUMN
```sql
Alter table hr
ADD COLUMN age_group varchar(40);
```

- TO group the age into groups of 0-29, 30-39, 40-49, 50-59
```sql
update HR
SET age_group = CASE
	WHEN age between 20 and 29 THEN '20-29'
    WHEN age between 30 and 39 THEN '30-39'
    WHEN age between 40 and 49 THEN '40-49'
    ELSE '50-59'
END;
```

## 3. Data Analysis and Findings
The following SQL queries were delveloped to answer sdpecific business questions:

1. What is the gender breakdown of employees in the company
```sql
SELECT gender, count(*) AS 'Total Employees'
FROM hr
group by gender;
```

2. WHat is the race/ethnicity breakwon of employees in the company
```sql
SELECT race, count(*) AS 'Total Employees'
FROM hr
group by race
order by count(*) DESC;
```

3. What is the age distribution of employees in the company
```sql
Select age_group, count(*) AS 'Total Employees'
from hr
group by age_group
order by count(*) DESC;
```

4. How many employees work at the headquarters versus remote locations?
```sql
select location, count(*) AS 'Total Employees by Location'
from hr
group by location;
```

5. What is the average length of employment for employees who have been terminated
```sql
SELECT AVG(TIMESTAMPDIFF(MONTH, hire_date, termdate)) AS avg_length_of_employment_in_month
FROM hr
WHERE EmpStatus = 'Terminated';
```

6. How does gender distribution vary across departments and job titles
```sql
SELECT department, jobtitle, gender, count(*) AS totalemployees
FROM hr
GROUP BY department, jobtitle, gender
ORDER BY department, jobtitle;
```

7. WHat is the distribution of job titles across the company
```sql
SELECT jobtitle, count(*) as totalemp
FROM hr
GROUP BY jobtitle
ORDER BY count(*) DESC;
```

8. To get the top 10 jobtitle by distribution
```sql
SELECT jobtitle, count(*) as totalemp
FROM hr
GROUP BY jobtitle
ORDER BY count(*) DESC
LIMIT 10;
```

8. Which department has the highest turnover rate
```sql
WITH term_emp AS (
	SELECT department, count(*) as terminations
    FROM hr
    WHERE EmpStatus = 'Terminated'
    GROUP BY department
),
total_emp AS(
	SELECT department, count(*) AS total
	FROM hr
    GROUP BY department
)
SELECT  
	term_emp.department,
    term_emp.terminations,
    term_emp.terminations/ total_emp.total AS turnover_rate
FROM term_emp
JOIN total_emp ON term_emp.department = total_emp.department
ORDER BY turnover_rate DESC
LIMIT 10;
```

9.  What is the distribution of employees across locations by city and state?
```sql
SELECT location_state, location_city, count(*) AS totalemployees
FROM hr
GROUP BY location_state, location_city
ORDER BY location_state, location_city;
```

10. How has the company employee count changed based on hire and term date?
```sql
WITH total_hired_emp AS (
	SELECT hire_year, 
		COUNT(*) AS total_emp_hired
	FROM hr
    GROUP BY hire_year
    ),
    total_term_emp AS (
    SELECT hire_year,
		count(*) AS total_emp_terminated
	FROM hr
	WHERE EmpStatus = 'Terminated'
    GROUP BY hire_year
    )
    SELECT total_hired_emp.hire_year,
		total_hired_emp.total_emp_hired,
        total_term_emp.total_emp_terminated,
        total_hired_emp.total_emp_hired - total_term_emp.total_emp_terminated AS net_change,
        (total_term_emp.total_emp_terminated/total_hired_emp.total_emp_hired) * 100 as termination_rate
	FROM total_hired_emp
    JOIN total_term_emp ON total_hired_emp.hire_year = total_term_emp.hire_year
   ORDER BY total_hired_emp.hire_year;
```



