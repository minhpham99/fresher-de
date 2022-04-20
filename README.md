EXPLANATION FOR MEDIUM SQL CHALLENGES

1. The PADS
Create a Cursor for counting total of people in each occupation, for later use in the while loop. The while loop fetch each row in the result execute by the cursor, get the count of each occupation and print to console

Code:
DECLARE @occupationCount AS INT
DECLARE @occupationName AS VARCHAR(10)
DECLARE @getOccupationCount CURSOR
SET @getOccupationCount = CURSOR FOR SELECT LOWER(Occupation), COUNT(Occupation) AS total 
FROM OCCUPATIONS GROUP BY Occupation ORDER BY COUNT(Occupation), Occupation;
SELECT CONCAT(Name, "(", SUBSTRING(Occupation, 1, 1), ")") FROM OCCUPATIONS ORDER BY Name;
OPEN @getOccupationCount
FETCH NEXT
FROM @getOccupationCount INTO @occupationName, @occupationCount
WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'There are a total of ' + CAST(@occupationCount AS VARCHAR(2)) + ' ' + @occupationName + 's.'
    FETCH NEXT
    FROM @getOccupationCount INTO @occupationName, @occupationCount
END
CLOSE @getOccupationCount
DEALLOCATE @getOccupationCount

2. Occupation
SELECT Doctor, Professor, Singer, Actor FROM  (
    SELECT ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) [RowNumber], * 
    FROM OCCUPATIONS
) t PIVOT (
    MAX(Name) for Occupation in (Doctor, Professor, Singer, Actor)
) as piv;

3. Binary Tree Nodes
SELECT N,
CASE
    WHEN P is NULL then 'Root'
    WHEN N IN (SELECT P FROM BST) then 'Inner'
    ELSE 'Leaf'
END as Type
FROM BST ORDER BY N;

4. New Companies 
SELECT c.company_code, c.founder, COUNT(DISTINCT(l.lead_manager_code)), COUNT(DISTINCT(s.senior_manager_code)), 
COUNT(DISTINCT(m.manager_code)), COUNT(DISTINCT(e.employee_code)) 
FROM Company c, Lead_Manager l, Senior_Manager s, Manager m, Employee e 
WHERE c.company_code = l.company_code 
AND l.lead_manager_code = s.lead_manager_code
AND s.senior_manager_code = m.senior_manager_code
AND m.manager_code = e.manager_code
GROUP BY c.company_code, c.founder ORDER BY c.company_code;

5. Weather Stations
18:
SELECT CAST(ROUND(ABS(MIN(LAT_N) - MAX(LAT_N)) + ABS(MIN(LONG_W) - MAX(LONG_W)), 4) AS FLOAT) FROM STATION 
19:
SELECT CAST(ROUND(SQRT(SQUARE(MAX(LAT_N) - MIN(LAT_N)) + SQUARE(MAX(LONG_W) - MIN(LONG_W))), 4) AS DECIMAL(10, 4)) FROM STATION;
20:
DECLARE @median as INT;
DECLARE @odds_even as INT;
SELECT @odds_even = COUNT(*) % 2 FROM STATION
SELECT @median = COUNT(*) / 2 FROM STATION;
IF @odds_even != 0
    WITH ordered_lat AS (SELECT ROW_NUMBER() OVER(
        ORDER BY LAT_N
    ) ROW_NUM, CITY, STATE, LAT_N, LONG_W FROM STATION)
    SELECT CAST(ROUND(LAT_N,4) AS DECIMAL(10,4)) FROM ordered_lat WHERE ROW_NUM = @median + 1;
ELSE
    WITH ordered_lat AS (SELECT ROW_NUMBER() OVER(
        ORDER BY LAT_N
    ) ROW_NUM, CITY, STATE, LAT_N, LONG_W FROM STATION)
    SELECT CAST(ROUND(AVG(LAT_N),4)AS DECIMAL(10,4)) FROM ordered_lat WHERE ROW_NUM >= @median AND ROW_NUM <= @median + 1;

6. The Report
SELECT IIF(grade < 8, 'NULL', name)
,grade, mark FROM (
SELECT s.Name as name, g.Grade as grade, s.Marks as mark FROM Students s INNER JOIN Grades g 
ON s.Marks BETWEEN g.Min_Mark AND g.Max_Mark
) t
ORDER BY t.grade DESC, name;

7. 
SELECT s.hacker_id, h.name FROM Hackers h INNER JOIN Submissions s ON h.hacker_id = s.hacker_id INNER JOIN (
    SELECT c.challenge_id as challenge_id, c.difficulty_level, d.score as full_score 
    FROM Challenges c INNER JOIN Difficulty d ON c.difficulty_level = d.difficulty_level) t
    ON s.challenge_id = t.challenge_id WHERE s.score = t.full_score 
    GROUP BY s.hacker_id, h.name
    HAVING COUNT(s.hacker_id) > 1 ORDER BY COUNT(s.hacker_id) DESC, s.hacker_id;

8. Ollivander's Inventory
SELECT w1.id, t.age, t.coins_needed, t.power
FROM (
    SELECT w_p.code, w_p.age, w.power, MIN(w.coins_needed) as coins_needed 
    FROM Wands w JOIN Wands_Property w_p 
    ON w.code = w_p.code
    WHERE w_p.is_evil = 0
    GROUP BY  w_p.code, w.power, w_p.age
) t JOIN Wands w1 ON t.coins_needed = w1.coins_needed AND t.code = w1.code AND w1.power = t.power 
JOIN Wands_Property w_p1 on t.age = w_p1.age 
ORDER BY t.power DESC, t.age DESC;

9. Challenges
With hackers_total_challenge AS (
SELECT h.hacker_id as id, h.name as name, COUNT(c.challenge_id) as total_challenge FROM Hackers h, Challenges c 
WHERE h.hacker_id = c.hacker_id GROUP BY h.hacker_id, h.name)
SELECT id, name, total_challenge FROM hackers_total_challenge GROUP BY id, name, total_challenge
HAVING total_challenge NOT IN (
    SELECT total_challenge FROM hackers_total_challenge
    GROUP BY total_challenge
    HAVING COUNT(total_challenge) > 1
    AND total_challenge NOT IN (SELECT MAX(total_challenge) FROM hackers_total_challenge)
)
ORDER BY total_challenge DESC, id;

10. Contest Leaderboard
SELECT h.hacker_id, h.name, SUM(max_score) FROM Hackers h INNER JOIN (
    SELECT hacker_id, challenge_id, MAX(Score) as max_score FROM Submissions GROUP BY hacker_id, challenge_id
) t ON h.hacker_id = t.hacker_id 
GROUP BY h.hacker_id, h.name 
HAVING SUM(max_score) > 0 ORDER BY SUM(max_score) DESC, h.hacker_id;

11. SQL Project Planning
WITH s1 AS(SELECT Start_Date FROM Projects
    WHERE Start_Date NOT IN (SELECT End_Date FROM Projects)),
s2 AS (SELECT End_Date FROM Projects
     WHERE End_Date NOT IN (SELECT Start_Date FROM Projects))
SELECT Start_Date, MIN(End_Date) FROM s1,s2
WHERE Start_Date < End_Date
GROUP BY Start_Date
ORDER BY DATEDIFF(day, MIN(End_Date), Start_Date) DESC, Start_Date

12. Placement
SELECT Students.Name FROM Students INNER JOIN(
SELECT Friends.ID as ID, P1.Salary as Salary, Friends.Friend_ID, P2.Salary as Friends_Salary 
FROM Friends INNER JOIN Packages P1 on Friends.ID = P1.ID 
INNER JOIN Packages P2 on Friends.Friend_ID = P2.ID WHERE P2.Salary > P1.Salary) t 
ON Students.ID = t.ID ORDER by t.Friends_Salary

13. Symmetric Pairs
SELECT * FROM(
SELECT x, y FROM Functions WHERE x = y GROUP BY x, y HAVING COUNT(*) >= 2
UNION
SELECT f1.x, f1.y FROM Functions f1, Functions f2 
WHERE f1.x < f1.y AND f1.x = f2.y AND f2.x = f1.y) t
ORDER BY x;

14. Print Prime Numbers
DECLARE @limit as INT
DECLARE @n as INT
DECLARE @result as NVARCHAR(MAX)
DECLARE @isPrime as INT
DECLARE @start as INT
SET @limit = 1000
SET @result = '2&'
SET @n = 3
WHILE @n <= @limit 
    BEGIN
    SET @start = 2
    SET @isPrime = 1
        WHILE @n > @start
            BEGIN
                IF @n % @start = 0
                    BEGIN
                        SET @isPrime = 0
                        BREAK;
                    END
                SET @start = @start + 1
            END
        IF @isPrime = 1
        BEGIN
            SET @result = @result + CAST((@n) as VARCHAR(4))+ '&'
        END
        SET @n = @n + 1
    END
PRINT LEFT(@result, LEN(@result) - 1)


HARD
1. Interviews
SELECT cnt.contest_id, cnt.hacker_id, cnt.name, 
ISNULL(SUM(ss.total_submissions), 0) AS total_submissions, 
ISNULL(SUM(ss.total_accepted_submissions), 0) AS total_accepted_submissions, ISNULL(SUM(vs.total_views), 0) AS total_views, 
ISNULL(SUM(vs.total_unique_views), 0) AS total_unique_views
FROM Contests cnt JOIN Colleges clg ON cnt.contest_id = clg.contest_id JOIN Challenges chl ON clg.college_id = chl.college_id
LEFT JOIN (
    SELECT challenge_id,SUM(total_submissions) AS total_submissions, SUM(total_accepted_submissions) AS total_accepted_submissions 
    FROM Submission_Stats GROUP BY challenge_id
) AS ss
ON chl.challenge_id = ss.challenge_id
LEFT JOIN(
    SELECT challenge_id, SUM(total_views) AS total_views, SUM(total_unique_views) AS total_unique_views 
    FROM View_Stats GROUP BY challenge_id
) vs 
ON chl.challenge_id = vs.challenge_id
GROUP BY cnt.contest_id, cnt.hacker_id, cnt.name
HAVING ISNULL(SUM(ss.total_submissions), 0) + ISNULL(SUM(ss.total_accepted_submissions), 0) + ISNULL(SUM(vs.total_views), 0) + ISNULL(SUM(vs.total_unique_views), 0) > 0
ORDER BY cnt.contest_id;

2. 15 Days SQL



