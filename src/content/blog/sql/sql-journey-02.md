---
title: "SQL Learning Journey #2 - Filtering, Strings, Aggregation & JOINs"
description: "Practicing COUNT, DISTINCT, string functions, filtering, sorting, and basic JOIN operations with SQL."
date: "2026-09-11"
category: "SQL"
tags: ["SQL", "HackerRank", "Filtering", "String Functions", "JOIN"]
---

# SQL Learning Journey #2

This session covers filtering, string functions, aggregation, sorting, and basic JOIN operations.



```sql
## 1. Difference Between Total and Distinct CITY Entries

Question:

Find the difference between the total number of `CITY` entries and the number of distinct `CITY` entries in `STATION`.

SELECT COUNT(CITY) - COUNT(DISTINCT CITY)
FROM STATION;




## 2. Shortest and Longest CITY Names

Question:
Find the shortest and longest city names along with their lengths. If there is a tie, choose the alphabetically first city.

Shortest
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY), CITY
FETCH FIRST 1 ROW ONLY;
Longest
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY) DESC, CITY
FETCH FIRST 1 ROW ONLY;




## 3. CITY Names Starting With Vowels

Question:
Find distinct city names that start with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(LEFT(CITY, 1)) IN ('a', 'e', 'i', 'o', 'u');




## 4. CITY Names Ending With Vowels

Question:
Find distinct city names that end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      IN ('a', 'e', 'i', 'o', 'u');




## 5. CITY Names Not Starting With Vowels

Question:
Find distinct city names that do not start with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;




## 6. CITY Names Not Ending With Vowels

Question:
Find distinct city names that do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u');




## 7. CITY Names That Do Not Start OR End With Vowels

Question:
Find distinct city names that either do not start with a vowel or do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
   OR LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;





## 8. CITY Names That Do Not Start AND Do Not End With Vowels

Question:
Find distinct city names that do not start with a vowel and do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
  AND LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;




## 9. Students Scoring More Than 75

Question:
Find the names of students who scored more than 75. Sort by the last three characters of the name and then by ascending ID.

SELECT NAME
FROM STUDENTS
WHERE MARKS > 75
ORDER BY SUBSTRING(NAME, -3), ID;




## 10. Employee Names in Alphabetical Order

Question:
Print employee names in alphabetical order.

SELECT name
FROM Employee
ORDER BY name;




## 11. Employees With Salary Greater Than 2000

Question:
Find employees earning more than 2000 per month who have worked for less than 10 months. Sort by ascending employee_id.

SELECT name
FROM Employee
WHERE salary > 2000
  AND months < 10
ORDER BY employee_id;




## 12. Total Population of Cities in Asia

Question:
Find the total population of all cities whose country belongs to the continent Asia.

SELECT SUM(CITY.POPULATION)
FROM CITY
JOIN COUNTRY
  ON CITY.COUNTRYCODE = COUNTRY.CODE
WHERE COUNTRY.CONTINENT = 'Asia';








Concepts: JOIN, ON, SUM(), filtering

Concepts Practiced
COUNT()
COUNT(DISTINCT)
SUM()
DISTINCT
WHERE
AND
OR
IN
NOT IN
ORDER BY
LENGTH()
LOWER()
LEFT()
SUBSTR()
SUBSTRING()
JOIN
ON