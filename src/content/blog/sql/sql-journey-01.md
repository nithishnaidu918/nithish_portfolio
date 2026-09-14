---
title: "SQL Learning Journey #1 "

description: "Practicing SELECT, WHERE, DISTINCT, string filtering, and MOD() through SQL problems."

date: "2026-09-11"

category: "SQL"

tags: ["SQL", "HackerRank", "SELECT", "WHERE"]
---

# SQL Learning Journey #1

This session covers basic data retrieval, filtering, selecting columns, removing duplicates, and working with even numbers in SQL.


```sql
## 1. Query All Columns

**Question:**

Query all columns for every row in the `CITY` table.

SELECT *

FROM CITY;




##2. Find a City by ID

Question:

Query all columns for the city whose ID is 1661.

SELECT *

FROM CITY

WHERE ID = 1661;




##3. Find Japanese Cities

Question:

Query all attributes of every Japanese city.

SELECT *

FROM CITY

WHERE COUNTRYCODE = 'JPN';





##4. Select Only City Names

Question:

Query the names of all Japanese cities.

SELECT NAME

FROM CITY

WHERE COUNTRYCODE = 'JPN';





##5. Select Multiple Columns

Question:

Query the CITY and STATE columns from the STATION table.

SELECT CITY, STATE

FROM STATION;




##6. Find Cities With Even IDs

Question:

Query the distinct city names from STATION where the station ID is even.

SELECT DISTINCT CITY

FROM STATION

WHERE MOD(ID, 2) = 0;





## 7. Difference Between Total and Distinct CITY Entries

Question:

Find the difference between the total number of `CITY` entries and the number of distinct `CITY` entries in `STATION`.

SELECT COUNT(CITY) - COUNT(DISTINCT CITY)
FROM STATION;




## 8. Shortest and Longest CITY Names

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




## 9. CITY Names Starting With Vowels

Question:
Find distinct city names that start with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(LEFT(CITY, 1)) IN ('a', 'e', 'i', 'o', 'u');




## 10. CITY Names Ending With Vowels

Question:
Find distinct city names that end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      IN ('a', 'e', 'i', 'o', 'u');




## 11. CITY Names Not Starting With Vowels

Question:
Find distinct city names that do not start with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;




## 12. CITY Names Not Ending With Vowels

Question:
Find distinct city names that do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u');




## 13. CITY Names That Do Not Start OR End With Vowels

Question:
Find distinct city names that either do not start with a vowel or do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
   OR LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;





## 14. CITY Names That Do Not Start AND Do Not End With Vowels

Question:
Find distinct city names that do not start with a vowel and do not end with a vowel.

SELECT DISTINCT CITY
FROM STATION
WHERE LOWER(SUBSTR(CITY, 1, 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
  AND LOWER(SUBSTR(CITY, LENGTH(CITY), 1))
      NOT IN ('a', 'e', 'i', 'o', 'u')
ORDER BY CITY;




## 15. Students Scoring More Than 75

Question:
Find the names of students who scored more than 75. Sort by the last three characters of the name and then by ascending ID.

SELECT NAME
FROM STUDENTS
WHERE MARKS > 75
ORDER BY SUBSTRING(NAME, -3), ID;




## 16. Employee Names in Alphabetical Order

Question:
Print employee names in alphabetical order.

SELECT name
FROM Employee
ORDER BY name;




## 17. Employees With Salary Greater Than 2000

Question:
Find employees earning more than 2000 per month who have worked for less than 10 months. Sort by ascending employee_id.

SELECT name
FROM Employee
WHERE salary > 2000
  AND months < 10
ORDER BY employee_id;




## 18. Total Population of Cities in Asia

Question:
Find the total population of all cities whose country belongs to the continent Asia.

SELECT SUM(CITY.POPULATION)
FROM CITY
JOIN COUNTRY
  ON CITY.COUNTRYCODE = COUNTRY.CODE
WHERE COUNTRY.CONTINENT = 'Asia';




