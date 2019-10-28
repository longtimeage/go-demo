# SQL面试题

## 1. 组合两个表
> 无论person是否有值，都显示
```go
SELECT t1.FirstName, t1.LastName, t2.City, t2.State FROM person t1 LEFT JOIN Address t2 ON t1.PersonId = t2.PersonId 
```

## 2. 第二高的薪水
> Salary 小于 最大值中的最大值就是第二高的
```sql
SELECT Max(Salary) as SecondHighestSalary FROM Employee Where Salary < (SELECT Max(Salary) FROM Employee)
```

## 3. 第N高的薪水
> 根据薪水从高到低排序，取offset
```sql
 Select (SELECT DISTINCT(Salary) From Employee Order By Salary DESC limit 1 OFFSET N) as NthHighestSalary
```

## 4. 分数排名
> 通过自连接的方式,查找出大于等于它的成绩的数量即为其排名，整理排序即得结果
```sql
SELECT 
    a.Score AS Score,
    COUNT(DISTINCT b.Score) AS Rank
FROM Scores AS a, Scores AS b
WHERE b.Score >= a.Score
GROUP BY a.id  
ORDER BY a.Score DESC
```

## 5. 连续数字
> 查找所有至少连续出现三次的数字
```sql
SELECT distinct t1.Num as ConsecutiveNums FROM Logs t1, Logs t2, Logs t3 
WHERE t1.Num = t2.Num and t2.Num = t3.Num 
and t1.Id + 1 = t2.Id and t2.Id + 1 = t3.Id
```

## 6. 查找哪些员工薪资超过自己经理的
> 自连表进行查询，t1为员工，t2为经理
```sql
SELECT t1.Name FROM Employee t1, Employee t2 WHERE t1.ManagerId = t2.Id AND t1.Salary > t2.Salary
```
## 7. 查找重复的电子邮箱
```sql
SELECT Email FROM Person GROUP BY Email Having count(*) > 1
```
## 8. 查找重没有购买过的客户
> 使用not in
```sql
SELECT t1.Name FROM Customers t1 WHERE t1.Id not in (SELECT CustomerId FROM Orders)
```

## 9. 部分中最高的工资
> 通过 `IN` max(Salary)
```sql
SELECT
t2.name AS 'Department',
t1.name AS 'Employee',
t1.Salary
FROM Employee t1 JOIN Department t2 ON t1.DepartmentId = t2.Id
WHERE (t1.DepartmentId , t1.Salary) IN
(SELECT DepartmentId, MAX(Salary)  FROM Employee GROUP BY DepartmentId);
```
## 10. 删除重复邮箱,只保留ID最小的
> 自连接
```sql
DELETE P1 
FROM Person P1, Person P2
WHERE P1.Email = P2.Email
AND P1.Id > P2.Id 
```
> 查出重复邮箱中ID最大的，然后 使用 in 删除这些最大的Id
```sql
delete from person where id in 
(select id from 
( select distinct t1.id as id from person t1,person t2
where t1.email=t2.email and t1.id>t2.id) as temp);

```

## 11. 上升的温度
> 查找昨天的温度比今天高的Id
```sql
SELECT t1.id FROM weather t1 
JOIN weather t2 ON DATEDIFF(t1.date, t2.date) = 1 AND t1.Temperature > t2.Temperature
```

## 12. 取消率
> ROUND(x, d)  x: 要处理的数 d: 保留几位小数
>
> IF(表达式, x, y): 表达式为true时取 x, false时取 y, 类似于Java的三元表达式
```sql
SELECT t1.Request_at, ROUND(SUM(IF(t1.Status = 'cancelled',1,0))/ COUNT(t1.Status), 2) FROM Trips t1
JOIN Users t2 ON (t1.Client_Id = t2.Users_Id AND t2.Banned = 'No')
JOIN Users t3 ON (t1.Driver_Id = t3.Users_Id AND t3.Banned = 'No')
Where t1.Request_at BETWEEN '2013-10-01' AND '2013-10-03' GROUP BY t1.Request_at
```
## 13. 游戏分析
1. 玩家第一天登录游戏日期
```sql
SELECT player_id, MIN(event_date)  AS first_login FROM Activity GROUP BY player_id
```
2. 玩家第一天登录游戏设备名称
```sql
SELECT player_id, device_id FROM Activity WHERE (player_id, event_date ) 
IN (SELECT player_id, MIN(event_date) FROM Activity GROUP BY player_id)
```
3. 统计各个日期前玩家的登录次数
```sql
SELECT t2.player_id, t2.event_date, sum(t1.games_played) AS games_played_so_far FROM Activity t1 JOIN Activity t2
ON (t1.event_date <= t2.event_date AND t1.player_id = t2.player_id) GROUP BY t2.player_id,t2.event_date
```