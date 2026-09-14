---
title: "SQL Learning Journey #5 - Aggregation, Dates & Advanced Queries"

description: "Practicing aggregate functions, conditional counting, date-based queries, multi-table joins, and advanced SQL calculations."

date: "2026-09-14"

category: "SQL"

tags: ["SQL", "HackerRank", "Aggregation", "Dates", "JOIN", "Subqueries"]
---

# SQL Learning Journey #5

This session focuses on aggregate functions, multi-table joins, date-based analysis, conditional counting, and advanced SQL calculations.



```sql
## 1. The Report - Contest Statistics

**Question:**

For each contest, calculate the total submissions, accepted submissions, views, and unique views. Exclude contests where all four totals are zero.

SELECT
    c.contest_id,
    c.hacker_id,
    c.name,
    COALESCE(SUM(ss.total_submissions), 0) AS total_submissions,
    COALESCE(SUM(ss.total_accepted_submissions), 0) AS total_accepted_submissions,
    COALESCE(SUM(vs.total_views), 0) AS total_views,
    COALESCE(SUM(vs.total_unique_views), 0) AS total_unique_views
FROM Contests AS c
JOIN Colleges AS co
    ON c.contest_id = co.contest_id
JOIN Challenges AS ch
    ON co.college_id = ch.college_id
LEFT JOIN Submission_Stats AS ss
    ON ch.challenge_id = ss.challenge_id
LEFT JOIN View_Stats AS vs
    ON ch.challenge_id = vs.challenge_id
GROUP BY
    c.contest_id,
    c.hacker_id,
    c.name
HAVING
    SUM(COALESCE(ss.total_submissions, 0)) > 0
    OR SUM(COALESCE(ss.total_accepted_submissions, 0)) > 0
    OR SUM(COALESCE(vs.total_views, 0)) > 0
    OR SUM(COALESCE(vs.total_unique_views, 0)) > 0
ORDER BY c.contest_id;








##2. 15 Days of Learning SQL

Question:

For each day of the contest:

Find the number of unique hackers who submitted every day from March 1 up to that day.
Find the hacker who made the maximum number of submissions that day.
If there is a tie, choose the lowest hacker_id.

SELECT
    d.submission_date,

    (
        SELECT COUNT(*)
        FROM (
            SELECT
                s2.hacker_id
            FROM Submissions AS s2
            WHERE s2.submission_date <= d.submission_date
              AND s2.submission_date >= '2016-03-01'
            GROUP BY s2.hacker_id
            HAVING COUNT(DISTINCT s2.submission_date)
                   = DATEDIFF(d.submission_date, '2016-03-01') + 1
        ) AS consistent_hackers
    ) AS unique_hackers,

    (
        SELECT s3.hacker_id
        FROM Submissions AS s3
        WHERE s3.submission_date = d.submission_date
        GROUP BY s3.hacker_id
        ORDER BY COUNT(*) DESC, s3.hacker_id ASC
        LIMIT 1
    ) AS hacker_id,

    (
        SELECT h.name
        FROM Hackers AS h
        JOIN Submissions AS s4
            ON h.hacker_id = s4.hacker_id
        WHERE s4.submission_date = d.submission_date
        GROUP BY h.hacker_id, h.name
        ORDER BY COUNT(*) DESC, h.hacker_id ASC
        LIMIT 1
    ) AS name

FROM (
    SELECT DISTINCT submission_date
    FROM Submissions
    WHERE submission_date BETWEEN '2016-03-01' AND '2016-03-15'
) AS d
ORDER BY d.submission_date;









##3. Average Population of California Cities

Question:

Find the average population of cities where the district is California.

SELECT AVG(POPULATION)
FROM CITY
WHERE DISTRICT = 'California';








##4. Average Population - Rounded Down

Question:

Find the average population of all cities and round it down to the nearest integer.

SELECT FLOOR(AVG(POPULATION))
FROM CITY;










##5. Japanese City Population

Question:

Find the total population of all Japanese cities.

Japan has country code JPN.

SELECT SUM(POPULATION)
FROM CITY
WHERE COUNTRYCODE = 'JPN';








##6. Population Difference

Question:

Find the difference between the maximum and minimum city populations.

SELECT MAX(POPULATION) - MIN(POPULATION)
FROM CITY;









##7. The Blunder

Question:

Samantha calculated the average salary after removing all zeros from salaries.

Find the difference between the actual average salary and her incorrect average, then round the result up.

SELECT CEIL(
    AVG(SALARY) -
    AVG(CAST(REPLACE(SALARY, '0', '') AS UNSIGNED))
)
FROM EMPLOYEES;










##8. Top Earners

Question:

Find the maximum employee earnings and the number of employees who have those maximum earnings.

Earnings are:

months × salary
SELECT
    MAX(months * salary),
    COUNT(*)
FROM Employee
WHERE months * salary = (
    SELECT MAX(months * salary)
    FROM Employee
);









##9. Weather Observation Station 2

Question:

Find:

The sum of all LAT_N values.
The sum of all LONG_W values.

Round both values to 2 decimal places.

SELECT
    ROUND(SUM(LAT_N), 2),
    ROUND(SUM(LONG_W), 2)
FROM STATION;






##10. Weather Observation Station 20

Question:

Find the sum of LAT_N values where:

LAT_N > 38.7880
LAT_N < 137.2345

Truncate the result to 4 decimal places.

SELECT TRUNCATE(SUM(LAT_N), 4)
FROM STATION
WHERE LAT_N > 38.7880
  AND LAT_N < 137.2345;