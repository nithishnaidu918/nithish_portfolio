---
title: "SQL Learning Journey #2"

description: "Practicing JOINs, GROUP BY, aggregate functions, CASE expressions, HAVING, and correlated subqueries with SQL."

date: "2026-09-12"

category: "SQL"

tags: ["SQL", "HackerRank", "JOIN", "GROUP BY", "CASE", "Subquery"]
---

# SQL Learning Journey #2

This session moves into intermediate SQL concepts including `JOIN`, `GROUP BY`, aggregate functions, `CASE`, `HAVING`, and subqueries.



```sql

## 1. Cities in Africa

**Question:**

Given the `CITY` and `COUNTRY` tables, find the names of all cities where the continent is `Africa`.

SELECT ci.NAME

FROM CITY AS ci

JOIN COUNTRY AS co
    ON ci.CountryCode = co.Code

WHERE co.CONTINENT = 'Africa';

Concepts: JOIN, ON, WHERE






## 2. Average City Population by Continent

Question:

Find each continent and its average city population, rounded down to the nearest integer.

SELECT
    co.Continent,
    FLOOR(AVG(ci.Population)) AS Average_Population

FROM CITY AS ci

JOIN COUNTRY AS co
    ON ci.CountryCode = co.Code

GROUP BY co.Continent;

Concepts: JOIN, AVG(), FLOOR(), GROUP BY





## 3. Student Grades Report

Question:

Generate a report containing Name, Grade, and Mark.

Students with grades below 8 should have their names displayed as NULL.

Sort by:

Grade descending
For grades 8-10, name alphabetically
For grades 1-7, marks ascending
SELECT
    CASE
        WHEN g.Grade < 8 THEN NULL
        ELSE s.Name
    END AS Name,
    g.Grade,
    s.Marks

FROM Students AS s

JOIN Grades AS g
    ON s.Marks BETWEEN g.Min_Mark AND g.Max_Mark

ORDER BY
    g.Grade DESC,
    CASE
        WHEN g.Grade >= 8 THEN s.Name
    END ASC,
    CASE
        WHEN g.Grade < 8 THEN s.Marks
    END ASC;

Concepts: JOIN, CASE, BETWEEN, ORDER BY, conditional sorting





## 4. Hackers With Multiple Full Scores

Question:

Find hackers who achieved full scores in more than one challenge.

Sort by:

Number of full-score challenges descending
hacker_id ascending when the count is the same
SELECT
    s.hacker_id,
    h.name

FROM Submissions AS s

JOIN Challenges AS c
    ON s.challenge_id = c.challenge_id

JOIN Difficulty AS d
    ON c.difficulty_level = d.difficulty_level

JOIN Hackers AS h
    ON s.hacker_id = h.hacker_id

WHERE s.score = d.score

GROUP BY
    s.hacker_id,
    h.name

HAVING COUNT(DISTINCT s.challenge_id) > 1

ORDER BY
    COUNT(DISTINCT s.challenge_id) DESC,
    s.hacker_id ASC;

Concepts: Multiple JOINs, GROUP BY, HAVING, COUNT(DISTINCT), filtering






## 5. Non-Evil Wands With Minimum Cost

Question:

Find the id, age, coins_needed, and power of non-evil wands.

For each combination of power and age, select the wand requiring the minimum number of coins.

Sort by:

power descending
age descending
SELECT
    w.id,
    wp.age,
    w.coins_needed,
    w.power

FROM Wands AS w

JOIN Wands_Property AS wp
    ON w.code = wp.code

WHERE wp.is_evil = 0

  AND w.coins_needed = (
      SELECT MIN(w2.coins_needed)

      FROM Wands AS w2

      JOIN Wands_Property AS wp2
          ON w2.code = wp2.code

      WHERE wp2.is_evil = 0
        AND w2.power = w.power
        AND wp2.age = wp.age
  )

ORDER BY
    w.power DESC,
    wp.age DESC;

Concepts: JOIN, subquery, MIN(), correlated subquery, filtering, ORDER BY




## 6. Hacker Challenge Creation Leaderboard

**Question:**

Print the `hacker_id`, `name`, and total number of challenges created by each hacker.

Sort by:

1. Total challenges in descending order
2. `hacker_id` in ascending order when counts are equal

If multiple hackers have the same challenge count and that count is less than the maximum challenge count, exclude them.


SELECT
    h.hacker_id,
    h.name,
    COUNT(c.challenge_id) AS challenge_count

FROM Hackers AS h

JOIN Challenges AS c
    ON h.hacker_id = c.hacker_id

GROUP BY
    h.hacker_id,
    h.name

HAVING challenge_count = (
    SELECT MAX(total)
    FROM (
        SELECT COUNT(*) AS total
        FROM Challenges
        GROUP BY hacker_id
    ) AS t
)

OR challenge_count NOT IN (
    SELECT total
    FROM (
        SELECT COUNT(*) AS total
        FROM Challenges
        GROUP BY hacker_id
    ) AS t2

    GROUP BY total

    HAVING COUNT(*) > 1
       AND total < (
           SELECT MAX(total)
           FROM (
               SELECT COUNT(*) AS total
               FROM Challenges
               GROUP BY hacker_id
           ) AS t3
       )
)

ORDER BY
    challenge_count DESC,
    h.hacker_id ASC;






##7. Hacker Total Score

Question:

Calculate the total score of each hacker.

For each hacker, find their maximum score for every challenge and then add those maximum scores together.

Exclude hackers whose total score is 0.

Sort by:

Total score descending
hacker_id ascending when scores are equal
SELECT
    h.hacker_id,
    h.name,
    SUM(s.max_score) AS total_score

FROM Hackers AS h

JOIN (
    SELECT
        hacker_id,
        challenge_id,
        MAX(score) AS max_score

    FROM Submissions

    GROUP BY
        hacker_id,
        challenge_id
) AS s
    ON h.hacker_id = s.hacker_id

GROUP BY
    h.hacker_id,
    h.name

HAVING SUM(s.max_score) > 0

ORDER BY
    total_score DESC,
    h.hacker_id ASC;




How it works

First, find the maximum score achieved by each hacker for each challenge:

SELECT
    hacker_id,
    challenge_id,
    MAX(score) AS max_score

FROM Submissions

GROUP BY
    hacker_id,
    challenge_id;

Then add those maximum scores for each hacker:

SUM(s.max_score)

For example:

Hacker 4071

Challenge 19797 → 95
Challenge 49593 → max(43, 96) = 96

Total = 95 + 96 = 191

Another example:

Hacker 74842

Challenge 19797 → max(98, 5) = 98
Challenge 63132 → 76

Total = 98 + 76 = 174

For hacker 84072:

Challenge 49593 → 100
Challenge 63132 → 0

Total = 100

The same calculation can be performed for the remaining hackers.

Concepts: MAX(), SUM(), subquery, GROUP BY, HAVING, aggregation







##8. Symmetric Pairs

Question:

Two pairs (X1, Y1) and (X2, Y2) are symmetric if:

X1 = Y2
X2 = Y1

Find all symmetric pairs and display them in ascending order by X.

Only list pairs where:

X <= Y
SELECT
    f1.X,
    f1.Y

FROM Functions AS f1

JOIN Functions AS f2
    ON f1.X = f2.Y
   AND f1.Y = f2.X

GROUP BY
    f1.X,
    f1.Y

HAVING f1.X < f1.Y
    OR COUNT(*) > 1

ORDER BY
    f1.X;




How it works

The table is joined with itself:

Functions AS f1
JOIN Functions AS f2

The JOIN checks whether:

f1.X = f2.Y
AND
f1.Y = f2.X

For example:

(20, 21)
(21, 20)

These are symmetric because:

20 = 20
21 = 21

For pairs where:

X = Y

such as:

(20, 20)

there must be more than one matching row, which is why:

COUNT(*) > 1

is used.

Concepts: Self JOIN, GROUP BY, HAVING, COUNT()






## 9. The Report - Contest Statistics

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





## 10. 15 Days of Learning SQL

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





## 11. Average Population of California Cities

Question:

Find the average population of cities where the district is California.

SELECT AVG(POPULATION)
FROM CITY
WHERE DISTRICT = 'California';





## 12. Average Population - Rounded Down

Question:

Find the average population of all cities and round it down to the nearest integer.

SELECT FLOOR(AVG(POPULATION))
FROM CITY;






## 13. Japanese City Population

Question:

Find the total population of all Japanese cities.

Japan has country code JPN.

SELECT SUM(POPULATION)
FROM CITY
WHERE COUNTRYCODE = 'JPN';





## 14. Population Difference

Question:

Find the difference between the maximum and minimum city populations.

SELECT MAX(POPULATION) - MIN(POPULATION)
FROM CITY;




## 15. The Blunder

Question:

Samantha calculated the average salary after removing all zeros from salaries.

Find the difference between the actual average salary and her incorrect average, then round the result up.

SELECT CEIL(
    AVG(SALARY) -
    AVG(CAST(REPLACE(SALARY, '0', '') AS UNSIGNED))
)
FROM EMPLOYEES;




## 16. Top Earners

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





## 17. Weather Observation Station 2

Question:

Find:

The sum of all LAT_N values.
The sum of all LONG_W values.

Round both values to 2 decimal places.

SELECT
    ROUND(SUM(LAT_N), 2),
    ROUND(SUM(LONG_W), 2)
FROM STATION;




## 18. Weather Observation Station 20

Question:

Find the sum of LAT_N values where:

LAT_N > 38.7880
LAT_N < 137.2345

Truncate the result to 4 decimal places.

SELECT TRUNCATE(SUM(LAT_N), 4)
FROM STATION
WHERE LAT_N > 38.7880
  AND LAT_N < 137.2345;