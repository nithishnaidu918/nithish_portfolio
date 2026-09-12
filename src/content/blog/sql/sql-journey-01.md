---
title: "SQL Learning Journey #1 - SELECT and Filtering Basics"

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
2. Find a City by ID

Question:

Query all columns for the city whose ID is 1661.

SELECT *

FROM CITY

WHERE ID = 1661;
3. Find Japanese Cities

Question:

Query all attributes of every Japanese city.

SELECT *

FROM CITY

WHERE COUNTRYCODE = 'JPN';
4. Select Only City Names

Question:

Query the names of all Japanese cities.

SELECT NAME

FROM CITY

WHERE COUNTRYCODE = 'JPN';
5. Select Multiple Columns

Question:

Query the CITY and STATE columns from the STATION table.

SELECT CITY, STATE

FROM STATION;
6. Find Cities With Even IDs

Question:

Query the distinct city names from STATION where the station ID is even.

SELECT DISTINCT CITY

FROM STATION

WHERE MOD(ID, 2) = 0;