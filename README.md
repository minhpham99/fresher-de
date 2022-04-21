EXPLANATION FOR MEDIUM SQL CHALLENGES

1. The PADS \
First, create a Cursor for counting total of people in each occupation, for later use in the while loop. The while loop fetch each row in the result execute by the cursor, get the count of each occupation and print to console.

```
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
```
2. Occupation \
The solution use PIVOT to transform distinct values in column Occupation into corresponding column names. The query first assign row numbers to people in differnt occupation, grouped together using PARTITION. Then each Occupation is transform into column, and their values are corresponding names. The outer query select values from all four columns. 
```
SELECT Doctor, Professor, Singer, Actor FROM  (
    SELECT ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) [RowNumber], * 
    FROM OCCUPATIONS
) t PIVOT (
    MAX(Name) for Occupation in (Doctor, Professor, Singer, Actor)
) as piv;
```
3. Binary Tree Nodes \
The query check two cases: if P has NULL value or if N matches any value in P. If P is NULL, means N is the root node since it has no parents. If N matches a value in P, it means that N is a parent of another N(aka N has children), therefore N is an inner node. Otherwise N is a leaf node since it does not have any children.
```
SELECT N,
CASE
    WHEN P is NULL then 'Root'
    WHEN N IN (SELECT P FROM BST) then 'Inner'
    ELSE 'Leaf'
END as Type
FROM BST ORDER BY N;
```
4. New Companies \
The query performs multiple join operation of all tables and count distinct code for each type of employee. 
```
SELECT c.company_code, c.founder, COUNT(DISTINCT(l.lead_manager_code)), COUNT(DISTINCT(s.senior_manager_code)), 
COUNT(DISTINCT(m.manager_code)), COUNT(DISTINCT(e.employee_code)) 
FROM Company c, Lead_Manager l, Senior_Manager s, Manager m, Employee e 
WHERE c.company_code = l.company_code 
AND l.lead_manager_code = s.lead_manager_code
AND s.senior_manager_code = m.senior_manager_code
AND m.manager_code = e.manager_code
GROUP BY c.company_code, c.founder ORDER BY c.company_code;
```
5. Weather Stations \
Weather Station 18: \
The query takes the minimum and maximum latitude and longitude and calculate the Manhattan Distance, round to four decimal places
```
SELECT CAST(ROUND(ABS(MIN(LAT_N) - MAX(LAT_N)) + ABS(MIN(LONG_W) - MAX(LONG_W)), 4) AS FLOAT) FROM STATION 
```
Weather Station 19: \
The query takes the minimum and maximum latitude and longitude and calculate the Euclidean Distance, round to four decimal places
```
SELECT CAST(ROUND(SQRT(SQUARE(MAX(LAT_N) - MIN(LAT_N)) + SQUARE(MAX(LONG_W) - MIN(LONG_W))), 4) AS DECIMAL(10, 4)) FROM STATION;
```
Weather Station 20: \
The query first sort the latitude in ascending order and assigning corresponded row number. The median value is either the middle value if the number of record in the table is odd, or the average of the two middle value if the number of records in the table is even. The result is rounded to 4 decimal points.
```
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
```
6. The Report \
The inner query map each students with their corresponding mark using the score range in the mark table and the student's score. The outer query check if the student's grade is higher than 8, and assign their name or NULL value for the name column. The table is sorted by the student's grade in descending order, and by the student's name if two or more have the same grade
```
SELECT IIF(grade < 8, 'NULL', name)
,grade, mark FROM (
SELECT s.Name as name, g.Grade as grade, s.Marks as mark FROM Students s INNER JOIN Grades g 
ON s.Marks BETWEEN g.Min_Mark AND g.Max_Mark
) t
ORDER BY t.grade DESC, name;
```
7. Top Competitors \
The inner query map the hacker and challenge table to filter hackers that receive full score on their submissions. This result is join with the outer query to obtain the hacker's name and only keep those that have more than one submissions. The final result is sorted by number of submissions in descending order, and by hacker id if two or more have the same number of submission.
```
SELECT s.hacker_id, h.name FROM Hackers h INNER JOIN Submissions s ON h.hacker_id = s.hacker_id INNER JOIN (
    SELECT c.challenge_id as challenge_id, c.difficulty_level, d.score as full_score 
    FROM Challenges c INNER JOIN Difficulty d ON c.difficulty_level = d.difficulty_level) t
    ON s.challenge_id = t.challenge_id WHERE s.score = t.full_score 
    GROUP BY s.hacker_id, h.name
    HAVING COUNT(s.hacker_id) > 1 ORDER BY COUNT(s.hacker_id) DESC, s.hacker_id;
```
8. Ollivander's Inventory \
The inner query select the minimum amount of coins needed for a combination of power and age, filter out those wands that are "evil". The result is joined with the Wands and Wands Property table to obtain wand id base on coins needed, age and power. The final result is sorted by power and age in descending order. 
```
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
```
9. Challenges \
The query create a common table expression(CTE) that calculate number of challenge for each hacker. The CTE is used to count the number of students who create a certain amount of challenge. If two or more student create the same amount of challenge and it is not the maximum, they will be excluded from the result. The final result is sorted by the total number of challenge in descending order and by id name if two or more students create maximum number of challenge.
```
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
```
10. Contest Leaderboard \
The inner query select the hacker id, the challenge they participate and the maximum score they got for the challenge. There can be two submissions from the same hacker for the same challenge. The final score is calculate by sum up all maximum scores that the hacker achieve and those that have score of zero will be excluded. The final result is sroted by the score in descending order and by hacker id if two hacker got the same score.
```
SELECT h.hacker_id, h.name, SUM(max_score) FROM Hackers h INNER JOIN (
    SELECT hacker_id, challenge_id, MAX(Score) as max_score FROM Submissions GROUP BY hacker_id, challenge_id
) t ON h.hacker_id = t.hacker_id 
GROUP BY h.hacker_id, h.name 
HAVING SUM(max_score) > 0 ORDER BY SUM(max_score) DESC, h.hacker_id;
```
11. SQL Project Planning \
The query first select start date which are not overlapped with any end date, and similarly to end date, because tasks in the same project are execute continuously therefore end date of a task will be start date of the other. This is done in order to get the start and finish date of a project. The final result is ordered by the amount of time taken to finish the project in descending order, and by start date if two projects have the same amount of time.
```
WITH s1 AS(SELECT Start_Date FROM Projects
    WHERE Start_Date NOT IN (SELECT End_Date FROM Projects)),
s2 AS (SELECT End_Date FROM Projects
     WHERE End_Date NOT IN (SELECT Start_Date FROM Projects))
SELECT Start_Date, MIN(End_Date) FROM s1,s2
WHERE Start_Date < End_Date
GROUP BY Start_Date
ORDER BY DATEDIFF(day, MIN(End_Date), Start_Date) DESC, Start_Date
```
12. Placement \
The inner query join tables Friends and Packages and filter out records that has a lower salary than their friends. The result is joined with the outer query to obtain the student name and the final result is ordered by their friend's salary.
```
SELECT Students.Name FROM Students INNER JOIN(
SELECT Friends.ID as ID, P1.Salary as Salary, Friends.Friend_ID, P2.Salary as Friends_Salary 
FROM Friends INNER JOIN Packages P1 on Friends.ID = P1.ID 
INNER JOIN Packages P2 on Friends.Friend_ID = P2.ID WHERE P2.Salary > P1.Salary) t 
ON Students.ID = t.ID ORDER by t.Friends_Salary
```
13. Symmetric Pairs \
The first part select pairs where x and y are equal and contain duplicates in the table. The second part cross-join the table functions together, denote with f1 and f2. The query compares pair of x and y in f1 and f2, such that f1.x = f2.y and f1.y = f2.x, in order to find a symmetric pair. The final result is order by the value of x. 
```
SELECT * FROM(
SELECT x, y FROM Functions WHERE x = y GROUP BY x, y HAVING COUNT(*) >= 2
UNION
SELECT f1.x, f1.y FROM Functions f1, Functions f2 
WHERE f1.x < f1.y AND f1.x = f2.y AND f2.x = f1.y) t
ORDER BY x;
```
14. Print Prime Numbers \
The starting digit is 3 and 2 is appended initially to the final result string. The algorithm aims to check if the number is a prime number by dividing it to all numbers starting from 2 to itself - 1(skip 1 because all numbers are divisible by 1). If at any point the modulus of the division is zero, this means the digit has another dividend beside 1 and itself, therefore it is not a prime number. A boolean value(denote by 0 for false and 1 for true) is used to determine whether the digit is a prime number. If true, it is appended to the result string. After the loop has reach its limit, the result string is printed to console.
```
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
```

HARD
1. Interviews \
The query select contest id, hacker id and hacker name, calculate total submissions, total accepted submissions for each contest, total views and total unique view for each contest. Each college only held one contest and in a contest there can be many challenges. The contest table does not have data on the college and challenge id, therefore it must be join with the college table to get the college id, in order to get the challenges that the contest hold. A challenge can have no views or no submission, therefore the challenge table performs left join on the submission and views table, NULL values are replaced with zero. The outermost query then calculate the total submissions of all challenges in a contest and output the final result order by contest id.
```
SELECT cnt.contest_id, cnt.hacker_id, cnt.name, 
ISNULL(SUM(ss.total_submissions), 0) AS total_submissions, 
ISNULL(SUM(ss.total_accepted_submissions), 0) AS total_accepted_submissions, ISNULL(SUM(vs.total_views), 0) AS total_views, 
ISNULL(SUM(vs.total_unique_views), 0) AS total_unique_views
FROM Contests cnt JOIN Colleges clg ON cnt.contest_id = clg.contest_id 
JOIN Challenges chl ON clg.college_id = chl.college_id
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
HAVING ISNULL(SUM(ss.total_submissions), 0) + ISNULL(SUM(ss.total_accepted_submissions), 0)
+ ISNULL(SUM(vs.total_views), 0) + ISNULL(SUM(vs.total_unique_views), 0) > 0
ORDER BY cnt.contest_id;
```
2. 15 Days SQL \
The first part create a CTE that contains each submission day and the id and name of the hacker that make the maximum submission on that day. By using row number over partition of submission date and sort the data by number of submission in descending order, we can filter out rows that have rank = 1(i.e the hacker with the largest number of submissions). This CTE is then join with the one that queries number of hacker that submit everyday since the start date til the current date of submission. 
This query will count the number of hackers that made at least one submission every day from the start date(2016/03/01) until the current date. 
The final result is sorted by hacker id in ascending order(done when creating CTE max_submission).
```
DECLARE @start_date as VARCHAR(12)= '2016-03-01';
WITH max_submission AS (
SELECT t.submission_date, t.hacker_id, h.name FROM
( SELECT submission_date, hacker_id, ROW_NUMBER() OVER
(PARTITION BY submission_date ORDER BY COUNT(submission_id) DESC, hacker_id) as row_number
FROM Submissions
GROUP BY submission_date, hacker_id) t JOIN Hackers h ON t.hacker_id = h.hacker_id
WHERE t.row_number = 1)
SELECT max_submission.submission_date, 
(SELECT COUNT(DISTINCT s1.hacker_id) as total_hackers FROM Submissions s1
    WHERE s1.submission_date = max_submission.submission_date AND
    (SELECT COUNT(DISTINCT(submission_date)) FROM Submissions s2
                  WHERE s2.hacker_id = s1.hacker_id
                  AND s2.submission_date < max_submission.submission_date
                  ) = DATEDIFF(day, @start_date, max_submission.submission_date)
) AS submit_every_day,
max_submission.hacker_id, max_submission.name
FROM max_submission
```

