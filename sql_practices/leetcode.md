# Leetcode SQL Problems

### [1126. Active Businesses](https://leetcode.com/problems/active-businesses/
)
- think about when to use `GROUP BY`


```sql
# Write your MySQL query statement below
/*
1. compute the avg occus in each event type 
2. count num of event_type of each business group by business_id that has greater than the avg occur in each event type
3. select the one that has > 1 event_type meet the condition
*/

WITH Avg_event AS (

    SELECT event_type, AVG(occurences) AS avg_event_occur
    FROM Events
    GROUP BY event_type

)

SELECT b.business_id 

FROM (
    SELECT e.business_id, COUNT(e.event_type) AS event_count
    FROM Events e, Avg_event a
    WHERE e.event_type = a.event_type AND e.occurences > a.avg_event_occur
    GROUP BY e.business_id
    
    )b

WHERE b.event_count > 1
GROUP BY b.business_id

```


### [550. Game Play Analysis IV](https://leetcode.com/problems/game-play-analysis-iv/)
- Figure what what is mean by "first log in"
- it has distinct player_id and event_date

``` sql
WITH Player_event AS (

    SELECT player_id, MIN(event_date) AS first_log_in
    FROM Activity
    GROUP BY player_id 

)

SELECT ROUND(COUNT(DISTINCT p.player_id) / (SELECT COUNT(DISTINCT player_id) from Activity),2) AS fraction
FROM Player_event p
JOIN Activity a
ON p.player_id = a.player_id 
WHERE DATEDIFF(a.event_date, p.first_log_in) = 1
```

### [1212. Team Scores in Football Tournament](https://leetcode.com/problems/team-scores-in-football-tournament/)
- `UNION ALL` 
- `IFNULL(col, default)` if null value use default value
- Must use alias if in nested query
- `ORDER BY` could use multiple col in order if ties


```sql
WITH Host_score AS(
    SELECT 
    host_team,
    CASE
        WHEN host_goals > guest_goals THEN 3
        WHEN host_goals = guest_goals THEN 1
        ELSE 0
    END AS host_score
    
    FROM Matches

),

Guest_score AS(
    SELECT 
    guest_team,
    CASE
        WHEN host_goals > guest_goals THEN 0
        WHEN host_goals = guest_goals THEN 1
        ELSE 3
    END AS guest_score
    
    FROM Matches

),


Team_union AS (
    SELECT host_team AS team_id, SUM(host_score) AS score
    FROM host_score
    GROUP BY host_team
    
    UNION ALL
    
    SELECT guest_team, SUM(guest_score) AS score
    FROM guest_score
    GROUP BY guest_team
    
)

SELECT team_id, team_name, IFNULL(score,0) AS num_points
FROM (
    
    SELECT t.team_id as team_id, t.team_name as team_name,
            SUM(tu.score) AS score
    
    FROM Team_union tu
    RIGHT JOIN Teams t
    ON t.team_id = tu.team_id
    GROUP BY t.team_id
) f
ORDER BY num_points DESC, team_id ASC
```

### [1204. Last Person to Fit in the Elevator](https://leetcode.com/problems/last-person-to-fit-in-the-elevator/)
- `SUM(col1) OVER(ORDER BY col2 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`, but if cumulative sum from start, could short fot `SUM(col1) OVER(ORDER BY col2 ASC|DESC)`
- `LIMIT 1` put at the end of the query

```sql
# Write your MySQL query statement below
WITH Total_weights AS (

SELECT 
    person_name, 
    SUM(weight) OVER (ORDER BY turn ASC) AS cum_weight
    FROM Queue
    ORDER BY turn ASC

)

SELECT person_name 
FROM Total_weights
WHERE cum_weight <=1000
ORDER BY cum_weight DESC
LIMIT 1
```

### [1132. Reported Posts II](https://leetcode.com/problems/reported-posts-ii/)
- `LEFT JOIN` here to keep the total post_id in Action table.
- `DISTINCT` to remove the duplicated rows since no primary key in Action table
- `AVG` function

```sql
WITH Spam_ratio AS (

    SELECT COUNT(DISTINCT r.post_id) / COUNT(DISTINCT a.post_id) AS ratio, a.action_date
    FROM Actions a
    LEFT JOIN Removals r
    ON a.post_id = r.post_id 
    WHERE a.extra="spam"
    GROUP BY a.action_date
    
)

SELECT ROUND(AVG(ratio)*100,2) AS average_daily_percent
FROM Spam_ratio
```
### [1454. Active Users](https://leetcode.com/problems/active-users/)
- `LAG(col, interval)` function and kinds of [window function](https://mode.com/sql-tutorial/sql-window-functions/#lag-and-lead)
- `DATEDIFF()` function, it has [different inputs requirement](https://mode.com/help/articles/use-your-data-with-the-mode-playbook/) in different sql system 
- `NATURAL JOIN` vs `INNER JOIN`. The former implicit find the common column and join without duplicate common columns, [example](https://www.geeksforgeeks.org/difference-between-natural-join-and-inner-join-in-sql/)
- Multiple CTEs use `,` to separate, only need one 'WITH' at first. [Example](https://mode.com/blog/use-common-table-expressions-to-keep-your-sql-clean/)

```sql
WITH Distinct_log AS (
    SELECT DISTINCT * 
    FROM Logins
),

    Lag_log_date AS (
    SELECT id, login_date, 
        LAG(login_date, 4) OVER(PARTITION BY id ORDER BY login_date) AS lag_4_date

    FROM Distinct_log
)

SELECT DISTINCT a.id, a.name

FROM Lag_log_date l
Natural Join Accounts a
WHERE DATEDIFF(login_date, lag_4_date)=4
ORDER BY a.id
```


### [1355. Activity Participants](https://leetcode.com/problems/activity-participants/)
- `SELECT MIN(num_friends) from Count_activity` to get the min/max values of the column
- sql query order

```sql
SELECT column_name(s)
FROM table_name
WHERE condition
GROUP BY column_name(s)
ORDER BY column_name(s);
```

```sql
WITH Count_activity AS (
    SELECT activity, COUNT(id) AS num_friends
    FROM Friends
    GROUP BY activity
)

SELECT activity 
FROM Count_activity c
JOIN Activities a
ON c.activity = a.name
WHERE c.num_friends > (SELECT MIN(num_friends) from Count_activity) AND c.num_friends < (SELECT MAX(num_friends) from Count_activity)
```

### [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)
__Key points__
- `DENSE_RANK() OVER (PARTITION BY ... ORDER BY ... [DESC|ASC])`
- Use `JOIN` here instead of `LEFT JOIN` to prevent if the "Department" table is NULL

``` sql
# Write your MySQL query statement below

# dense_rank 

WITH Dept_rank_salary AS (

    SELECT DepartmentId, Name, salary, DENSE_RANK() OVER(PARTITION BY DepartmentId ORDER BY salary DESC) AS dept_salary_rank
    FROM Employee
    
) 

SELECT t.Name as Department, d.Name AS Employee, d.salary as Salary

FROM Dept_rank_salary as d

JOIN Department t

ON t.id = d.departmentid 

WHERE d.dept_salary_rank = 1

```





### 177. Nth Highest Salary
__Key points__
- `DENSE_RANK() OVER(ORDER BY ... DESC)`, why not use `ROW_NUMBER()` or `RANK()`  here, [example](https://codingsight.com/similarities-and-differences-among-rank-dense_rank-and-row_number-functions/)
- `DISTINCT()` render only 1 result 
- [Great article](https://leetcode-cn.com/problems/nth-highest-salary/solution/mysql-zi-ding-yi-bian-liang-by-luanz/) on different methods and their query efficiency

```sql
      SELECT DISTINCT salary 
      FROM 
      (
        SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank_salary
        FROM Employee
      ) rank_s
      
      WHERE rank_salary=N 

```


### 1468. Calculate Salaries
__Key points__
- CASE WHEN... THEN ... ELSE ... END AS [Example](https://mode.com/sql-tutorial/sql-case/)
- `ROUND` FUNCTION, default round number to integer [example](https://www.w3schools.com/sql/func_sqlserver_round.asp)
- `JOIN ON` and `LEFT JOIN ON`
-  [Common table  expression (CTE)](https://blog.csdn.net/bcbobo21cn/article/details/71155359), `WITH... AS ()`


__Method 1__
``` sql
SELECT s.company_id, s.employee_id, s.employee_name, 
ROUND(s.salary*(1-m.tax_rate)) AS salary
FROM Salaries s

LEFT JOIN (
    SELECT 
    company_id,
    CASE
        WHEN MAX(salary) < 1000 THEN 0
        WHEN MAX(salary) BETWEEN 1000 and 10000 THEN 0.24
        ELSE 0.49
    END AS tax_rate
    FROM Salaries
    GROUP BY company_id
) m

ON m.company_id = s.company_id 
```

__METHOD 2, cte__
```sql

WITH Company_tax AS (

    SELECT company_id, 
        CASE 
            WHEN MAX(salary) < 1000 THEN 0
            WHEN MAX(salary) BETWEEN 1000 AND 10000 THEN 0.24
            WHEN MAX(salary) > 10000 THEN 0.49  
        END AS tax_rate
    
    FROM Salaries
    
    GROUP BY company_id
)




SELECT 
    s.company_id, 
    s.employee_id, 
    s.employee_name, 
    ROUND(s.salary*(1 - t.tax_rate)) AS salary

FROM Salaries s 

LEFT JOIN Company_tax t

ON t.company_id = s.company_id



```


### 176. Second Highest Salary
``` sql
SELECT MAX(Salary) AS SecondHighestSalary 
FROM Employee 
WHERE Salary < (SELECT MAX(Salary) AS max_salary FROM Employee)
```

### 185. Department Top Three Salaries
- `dense_rank() over`
- `partition by` 

```sql

SELECT d.Name as Department, a.Name as Employee, a.Salary
FROM
(
    SELECT e.*, DENSE_RANK() OVER (PARTITION BY e.DepartmentId ORDER BY e.Salary DESC) AS DeptSalRank
    FROM Employee e
) a

JOIN Department d
ON d.Id = a.DepartmentId
WHERE DeptSalRank <=3
```


### 262. Trips and Users
- Join two tables on `Users_Id` and `Client_Id` and `Driver_Id` with condition `Banned !=No`
- `AVG(status!='completed')` and `GROUP BY` `Request_at`

```sql
SELECT t.Request_at AS DAy , ROUND(AVG(t.Status!='completed'),2) AS 'Cancellation Rate'
FROM trips t JOIN Users u1 ON (u1.Users_Id = t.Client_Id AND u1.Banned ='NO')
             JOIN Users u2 on (u2.Users_Id = t.Driver_Id AND u2.Banned = 'NO')
WHERE t.Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.Request_at
```

### 181. Employees Earning More Than Their Managers
-  `JOIN ON`

```sql

SELECT a.Name as Employee
FROM Employee a JOIN Employee b
ON a.ManagerId = b.Id
WHERE a.Salary > b.Salary ;

```

### 1179. Reformat Department Table
- `CASE WHEN THEN ELSE END` syntax
- `MAX` as Agg function make one record in one row



```sql

# Write your MySQL query statement below
SELECT id, 
MAX(CASE WHEN month='Jan' then revenue ELSE NULL END) AS Jan_Revenue,
MAX(CASE WHEN month='Feb' then revenue ELSE NULL END) AS Feb_Revenue,
MAX(CASE WHEN month='Mar' then revenue ELSE NULL END) AS Mar_Revenue,
MAX(CASE WHEN month='Apr' then revenue ELSE NULL END) AS Apr_Revenue,
MAX(CASE WHEN month='May' then revenue ELSE NULL END) AS May_Revenue,
MAX(CASE WHEN month='Jun' then revenue ELSE NULL END) AS Jun_Revenue,
MAX(CASE WHEN month='Jul' then revenue ELSE NULL END) AS Jul_Revenue,
MAX(CASE WHEN month='Aug' then revenue ELSE NULL END) AS Aug_Revenue,
MAX(CASE WHEN month='Sep' then revenue ELSE NULL END) AS Sep_Revenue,
MAX(CASE WHEN month='Oct' then revenue ELSE NULL END) AS Oct_Revenue,
MAX(CASE WHEN month='Nov' then revenue ELSE NULL END) AS Nov_Revenue,
MAX(CASE WHEN month='Dec' then revenue ELSE NULL END) AS Dec_Revenue

FROM Department
GROUP BY id
ORDER BY id ;
```

### 626. Exchange Seats
- `MOD()` 
- usage of commas

```sql
SELECT 
(CASE
    WHEN MOD(id,2)!=0 and counts != id THEN id+1
    WHEN MOD(id,2)!=0 and counts = id THEN id
    ELSE id-1 END
) AS id,
    student
FROM seat,
    (
        SELECT COUNT(*) as counts
        FROM seat
    ) as seat_counts
    
ORDER BY id;


```


### 1270. All People Report to the Given Manager

```sql

# Write your MySQL query statement below
SELECT e1.employee_id
FROM Employees e1
JOIN Employees e2 ON e1.manager_id = e2.employee_id
JOIN Employees e3 ON e2.manager_id = e3.employee_id 
WHERE e3.manager_id=1 AND  e1.employee_id != 1


```

### 196. Delete Duplicate Emails

```sql

DELETE p1 
FROM Person p1, Person p2
WHERE p1.Email = p2.Email AND p1.Id> p2.Id;
```



### 615. Average Salary: Departments VS Company
- [Solution from LC Chinese official website](https://leetcode-cn.com/problems/average-salary-departments-vs-company/solution/ping-jun-gong-zi-bu-men-yu-gong-si-bi-jiao-by-leet/)
- date_format(pay_date,'%Y-%M')


```sql

/* First step: get monthly avg company salary */
/*
SELECT 
    AVG(amount) as company_month_avg_salary, 
    date_format(pay_date, '%Y-%m') AS pay_month
FROM salary
GROUP BY date_format(pay_date, '%Y-%M');

*/

/* Second step: get department monthly avg department salary */
/*
SELECT 
    department_id,
    AVG(amount) as dept_month_avg_salary, 
    date_format(pay_date, '%Y-%m') AS pay_month
    
    
FROM salary s 
JOIN employee e
ON  s.employee_id = e.employee_id 
GROUP BY date_format(pay_date, '%Y-%M'), department_id;

*/

-- Final step: 

SELECT 
    dept_month_salary.pay_month,
    dept_month_salary.department_id,
    (
        CASE 
            WHEN dept_month_avg_salary>company_month_avg_salary THEN 'higher'
            WHEN dept_month_avg_salary = company_month_avg_salary THEN 'same'
            ELSE 'lower' 
        END
    ) AS comparison
    
FROM
    (
        SELECT 
        AVG(amount) as company_month_avg_salary, 
        date_format(pay_date, '%Y-%m') AS pay_month
        FROM salary
        GROUP BY date_format(pay_date, '%Y-%M')
    ) AS company_month_salary
    
    JOIN
    (
        SELECT 
            department_id,
            AVG(amount) as dept_month_avg_salary, 
            date_format(pay_date, '%Y-%m') AS pay_month

            FROM salary s 
            JOIN employee e
            ON  s.employee_id = e.employee_id 
            GROUP BY date_format(pay_date, '%Y-%M'), department_id
    ) AS dept_month_salary
    
    ON dept_month_salary.pay_month = company_month_salary.pay_month;

```


### 180. Consecutive Numbers
- This [Solution](https://leetcode-cn.com/problems/consecutive-numbers/solution/sql-server-jie-fa-by-neilsons/) could be applied to larger N.
- [Solution from LC sql chinese website](https://leetcode-cn.com/problems/consecutive-numbers/solution/lian-xu-chu-xian-de-shu-zi-by-leetcode/)
-  `DISTINCT` to remove the duplicates


```sql
# Write your MySQL query statement below
SELECT DISTINCT b.num AS consecutiveNums
FROM
(
SELECT a.num, count(*) AS count_group
FROM(
    SELECT id, num, 
    ROW_NUMBER() OVER(ORDER BY id)  - ROW_NUMBER() OVER(PARTITION BY num ORDER BY id) AS row_diff
    
    FROM Logs
) a
GROUP BY a.num, a.row_diff
) b     
WHERE count_group>=3


```


```sql

# Write your MySQL query statement below
SELECT DISTINCT l1.Num AS ConsecutiveNums 
FROM
    Logs l1, Logs l2, Logs l3

WHERE 
    l1.Num = l2.Num AND l2.Num = l3.Num
    AND l1.id = l2.id -1 AND l2.id = l3.id -1;


```