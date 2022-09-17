### [175.组合两个表](https://leetcode.cn/problems/combine-two-tables/)

```sql
表: Person
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
personId 是该表的主键列。
该表包含一些人的 ID 和他们的姓和名的信息。
 

表: Address

+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
addressId 是该表的主键列。
该表的每一行都包含一个 ID = PersonId 的人的城市和州的信息。
 

编写一个SQL查询来报告 Person 表中每个人的姓、名、城市和州。如果 personId 的地址不在 Address 表中，则报告为空  null 。

以 任意顺序 返回结果表。
```

**题解：**

```sql
# Write your MySQL query statement below
SELECT
	a.firstName,
	a.lastName,
	b.city,
	b.state 
FROM
	Person a
left join
	Address b 
on
	a.PersonId = b.PersonId;
```

### [176.第二高的薪水](https://leetcode.cn/problems/second-highest-salary/)

```sql
Employee 表：
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| salary      | int  |
+-------------+------+
id 是这个表的主键。
表的每一行包含员工的工资信息。

编写一个 SQL 查询，获取并返回 Employee 表中第二高的薪水 。如果不存在第二高的薪水，查询应该返回 null 。
```

**题解：**

```sql
SELECT
	IFNULL( ( SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1 ), NULL ) AS SecondHighestSalary
```

### [177. 第N高的薪水](https://leetcode.cn/problems/nth-highest-salary/)

```sql
表: Employee

+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| salary      | int  |
+-------------+------+
Id是该表的主键列。
该表的每一行都包含有关员工工资的信息。
 

编写一个SQL查询来报告 Employee 表中第 n 高的工资。如果没有第 n 个最高工资，查询应该报告为 null 。
```

**题解：**

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
SET N := N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT ( SELECT DISTINCT Salary FROM employee ORDER BY Salary DESC LIMIT N , 1) AS SecondHighestSalary
  );
END
```

### [178. 分数排名](https://leetcode.cn/problems/rank-scores/)

```sql
表: Scores

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| score       | decimal |
+-------------+---------+
Id是该表的主键。
该表的每一行都包含了一场比赛的分数。Score是一个有两位小数点的浮点值。
 
编写 SQL 查询对分数进行排序。排名按以下规则计算:

分数应按从高到低排列。
如果两个分数相等，那么两个分数的排名应该相同。
在排名相同的分数后，排名数应该是下一个连续的整数。换句话说，排名之间不应该有空缺的数字。
按 score 降序返回结果表。
```

**题解：**

```sql
# 解法1：耗时806ms
select a.Score as Score,
(select count(distinct b.Score) from Scores b where b.Score >= a.Score) as `Rank`
from Scores a
order by a.Score DESC

# 解法2：使用排名窗口函数dense_rank()优化 耗时 313ms
select score, dense_rank() over (order by score desc) as `rank`
from scores;
```

### [180. 连续出现的数字](https://leetcode.cn/problems/consecutive-numbers/)

表：Logs

```sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| num         | varchar |
+-------------+---------+
id 是这个表的主键。
```

编写一个 SQL 查询，查找所有至少连续出现三次的数字。

返回的结果表中的数据可以按 任意顺序 排列。

**题解：**
```sql
# Write your MySQL query statement below
SELECT
	Distinct l1.num ConsecutiveNums 	
FROM
logs l1,
logs l2,
logs l3
WHERE
l1.id = l2.id - 1
and 
l2.id = l3.id - 1
and 
l1.num = l2.num
and 
l2.num = l3.num
```

### [181. 超过经理收入的员工](https://leetcode.cn/problems/employees-earning-more-than-their-managers/)

```sql
表：Employee 

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| salary      | int     |
| managerId   | int     |
+-------------+---------+
Id是该表的主键。
该表的每一行都表示雇员的ID、姓名、工资和经理的ID。
 

编写一个SQL查询来查找收入比经理高的员工。

以 任意顺序 返回结果表。
```

**题解：**

```sql
# 解法1：
SELECT
     a.NAME AS Employee
FROM Employee AS a JOIN Employee AS b
     ON a.ManagerId = b.Id
     AND a.Salary > b.Salary

# 解法2：
SELECT 
a1.`name` Employee
FROM 
`employee` a1,
`employee` a2
WHERE
a1.`managerId` = a2.`id`  
AND 
a1.`salary` > a2.`salary`
```

### [182. 查找重复的电子邮箱](https://leetcode.cn/problems/duplicate-emails/)

```sql
编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

示例：

+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
根据以上输入，你的查询应返回以下结果：

+---------+
| Email   |
+---------+
| a@b.com |
+---------+
说明：所有电子邮箱都是小写字母。
```

**题解：**

```sql
# 400ms
select Email
from Person
group by Email
having count(Email) > 1;
```

### [183. 从不订购的客户](https://leetcode.cn/problems/customers-who-never-order/)

某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

```SQL
Customers 表：

+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders 表：

+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
例如给定上述表格，你的查询应返回：

+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

**题解：**
```sql
# 解法1：
select 
c.Name Customers 
from 
Customers c
where 
c.Id not in 
(select o.CustomerId from Orders  o)

# 解法2：
select a.Name as Customers
from Customers as a
left join Orders as b
on a.Id=b.CustomerId
where b.CustomerId is null;
```

### [184. 部门工资最高的员工](https://leetcode.cn/problems/department-highest-salary/)

```sql
表： Employee

+--------------+---------+
| 列名          | 类型    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
id是此表的主键列。
departmentId是Department表中ID的外键。
此表的每一行都表示员工的ID、姓名和工资。它还包含他们所在部门的ID。
 

表： Department

+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
id是此表的主键列。
此表的每一行都表示一个部门的ID及其名称。
 

编写SQL查询以查找每个部门中薪资最高的员工。
按 任意顺序 返回结果表。
查询结果格式如下例所示。
```

**题解：**
```sql
# Write your MySQL query statement below
SELECT 
  d.name Department,
  e.`name` Employee,
  e.`salary` Salary 
FROM
  `department` d,
  `employee` e 
WHERE d.`id` = e.`departmentId` 
  AND (e.`departmentId`, e.`salary`) IN 
  (SELECT 
    DepartmentId,
    MAX(Salary) 
  FROM
    Employee 
  GROUP BY DepartmentId)
```

### [185. 部门工资前三高的所有员工](https://leetcode.cn/problems/department-top-three-salaries/)

```sql
表: Employee

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
Id是该表的主键列。
departmentId是Department表中ID的外键。
该表的每一行都表示员工的ID、姓名和工资。它还包含了他们部门的ID。
 

表: Department

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
Id是该表的主键列。
该表的每一行表示部门ID和部门名。
 

公司的主管们感兴趣的是公司每个部门中谁赚的钱最多。一个部门的 高收入者 是指一个员工的工资在该部门的 不同 工资中 排名前三 。

编写一个SQL查询，找出每个部门中 收入高的员工 。

以 任意顺序 返回结果表。
```
**题解：**
```sql
# 解法1：
公司里前 3 高的薪水意味着有不超过 3 个工资比这些值大。
SELECT
    d.Name AS 'Department', e1.Name AS 'Employee', e1.Salary
FROM
    Employee e1
        JOIN
    Department d ON e1.DepartmentId = d.Id
WHERE
    3 > (SELECT
            COUNT(DISTINCT e2.Salary)
        FROM
            Employee e2
        WHERE
            e2.Salary > e1.Salary
                AND e1.DepartmentId = e2.DepartmentId
        )

#解法2：
select
    base.Department,
    base.Employee,
    base.Salary
from (
    select
        Department.Name as 'Department',
        Employee.Name as 'Employee',
        Employee.Salary,
        # 同分不同名-连续：row_number()
        # 同分不同名-不连续：你猜？
        # 同分同名-连续：dense_rank()
        # 同分同名-不连续：rank()
        dense_rank() over(partition by Employee.DepartmentId order by Employee.Salary desc) as 'Ranking'
    from Employee
    inner join Department
    on Employee.DepartmentId = Department.Id
) as base
where base.Ranking <= 3
```

### [196. 删除重复的电子邮箱](https://leetcode.cn/problems/delete-duplicate-emails/)

```sql
表: Person

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| email       | varchar |
+-------------+---------+
id是该表的主键列。
该表的每一行包含一封电子邮件。电子邮件将不包含大写字母。
 

编写一个 SQL 删除语句来 删除 所有重复的电子邮件，只保留一个id最小的唯一电子邮件。

以 任意顺序 返回结果表。 （注意： 仅需要写删除语句，将自动对剩余结果进行查询）
```

**题解：**

```sql
# 1037ms
DELETE p1 FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
```

### [197. 上升的温度](https://leetcode.cn/problems/rising-temperature/)

```sql
表： Weather

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| recordDate    | date    |
| temperature   | int     |
+---------------+---------+
id 是这个表的主键
该表包含特定日期的温度信息
 

编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 id 。

返回结果 不要求顺序 。
```
**题解：**
```sql
# 解法1：
SELECT
    weather.id AS 'Id'
FROM
    weather
        JOIN
    weather w ON DATEDIFF(weather.recordDate, w.recordDate) = 1
        AND weather.Temperature > w.Temperature
;

# 解法2（lead窗口函数）：
SELECT Id
FROM 
(
	SELECT 
		temperature,
		recordDate,
		lead(id,1) over (ORDER BY recordDate) AS 'Id',
		lead(recordDate,1) over (ORDER BY recordDate) AS 'nextDate',
		lead(temperature,1) over (ORDER BY recordDate) AS 'nextTemp'
	FROM weather 
)t 
WHERE nextTemp > temperature AND DATEDIFF(nextDate, recordDate) = 1

# 错误解法（7/14出错）：
select
w2.id Id
from
Weather w1,
Weather w2
where
w2.id = w1.id + 1 
and
w2.temperature > w1.temperature
```

### [262. 行程和用户](https://leetcode.cn/problems/trips-and-users/)

```sql
表：Trips
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| id          | int      |
| client_id   | int      |
| driver_id   | int      |
| city_id     | int      |
| status      | enum     |
| request_at  | date     |     
+-------------+----------+
id 是这张表的主键。
这张表中存所有出租车的行程信息。每段行程有唯一 id ，其中 client_id 和 driver_id 是 Users 表中 users_id 的外键。
status 是一个表示行程状态的枚举类型，枚举成员为(‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’) 。
 

表：Users

+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| users_id    | int      |
| banned      | enum     |
| role        | enum     |
+-------------+----------+
users_id 是这张表的主键。
这张表中存所有用户，每个用户都有一个唯一的 users_id ，role 是一个表示用户身份的枚举类型，枚举成员为 (‘client’, ‘driver’, ‘partner’) 。
banned 是一个表示用户是否被禁止的枚举类型，枚举成员为 (‘Yes’, ‘No’) 。
 

取消率 的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)。

写一段 SQL 语句查出 "2013-10-01" 至 "2013-10-03" 期间非禁止用户（乘客和司机都必须未被禁止）的取消率。非禁止用户即 banned 为 No 的用户，禁止用户即 banned 为 Yes 的用户。

返回结果表中的数据可以按任意顺序组织。其中取消率 Cancellation Rate 需要四舍五入保留 两位小数 。
```
**解法：**
```sql
# 解法1：
select request_at 'Day',
round(count(if(status!='completed',status,null))/count(*),2) 'Cancellation Rate'
from Trips
where request_at between '2013-10-01' and '2013-10-03'
and client_id not in (select users_id from Users where banned='Yes') 
and driver_id not in (select users_id from Users where banned='Yes')
group by request_at;

#解法2：
select request_at 'Day',
round(avg(status!='completed'),2) 'Cancellation Rate'
from Trips
where request_at between '2013-10-01' and '2013-10-03'
and client_id not in (select users_id from Users where banned='Yes') 
and driver_id not in (select users_id from Users where banned='Yes')
group by request_at;
```

### [584. 寻找用户推荐人](https://leetcode.cn/problems/find-customer-referee/)

```sql
给定表 customer ，里面保存了所有客户信息和他们的推荐人。

+------+------+-----------+
| id   | name | referee_id|
+------+------+-----------+
|    1 | Will |      NULL |
|    2 | Jane |      NULL |
|    3 | Alex |         2 |
|    4 | Bill |      NULL |
|    5 | Zack |         1 |
|    6 | Mark |         2 |
+------+------+-----------+
写一个查询语句，返回一个客户列表，列表中客户的推荐人的编号都 不是 2。

对于上面的示例数据，结果为：

+------+
| name |
+------+
| Will |
| Jane |
| Bill |
| Zack |
+------+
```

**题解：**

```sql
# 解法1：
select
`name`
from 
customer
where
referee_id is null
or
referee_id != 2

# 解法2：
select name from customer where ifnull(referee_id,0) !=2

# 解法3：用安全等于取反，与解法1、2相同，但是可读性较差，不建议使用
SELECT name
FROM customer
WHERE NOT referee_id <=> 2;
```

**特别说明：**

> 可能很多人第一眼看到这个题目，就会很容易的写出一下语句：
>
> ```sql
> SELECT name FROM customer WHERE referee_Id <> 2;
> ```
>
> 如果不在数据库上运行，可能看不出来问题，但是实际运行后发现：
>
> 没有推荐人（referee_id 字段值为 `NULL`) 的全部都消失了，这是为什么呢？
>
> MySQL 使用三值逻辑 —— TRUE, FALSE 和 UNKNOWN。任何与 NULL 值进行的比较都会与第三种值 UNKNOWN 做比较。这个“任何值”包括 NULL 本身！这就是为什么 MySQL 提供 IS NULL 和 IS NOT NULL 两种操作来对 NULL 特殊判断。
>
> 因此，在 WHERE 语句中我们需要做一个额外的条件判断 `referee_id IS NULL'
>



### [586. 订单最多的客户](https://leetcode.cn/problems/customer-placing-the-largest-number-of-orders/)

```sql
表: Orders

+-----------------+----------+
| Column Name     | Type     |
+-----------------+----------+
| order_number    | int      |
| customer_number | int      |
+-----------------+----------+
Order_number是该表的主键。
此表包含关于订单ID和客户ID的信息。
 

编写一个SQL查询，为下了 最多订单 的客户查找 customer_number 。

测试用例生成后， 恰好有一个客户 比任何其他客户下了更多的订单。
```

**题解：**

```sql
# 解法1：532ms 
SELECT
    customer_number
FROM
    orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1

进阶： 如果有多位顾客订单数并列最多，你能找到他们所有的 customer_number 吗？
# 解法2：594ms 
select
customer_number 
from
(
select 
    customer_number, 
    dense_rank() over(order by count(order_number) desc) as `rank`
from orders
group by customer_number
) t
where
t.rank = 1
```

### [595. 大的国家](https://leetcode.cn/problems/big-countries/)

```sql
World 表：

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| name        | varchar |
| continent   | varchar |
| area        | int     |
| population  | int     |
| gdp         | int     |
+-------------+---------+
name 是这张表的主键。
这张表的每一行提供：国家名称、所属大陆、面积、人口和 GDP 值。
 

如果一个国家满足下述两个条件之一，则认为该国是 大国 ：

面积至少为 300 万平方公里（即，3000000 km2），或者
人口至少为 2500 万（即 25000000）
编写一个 SQL 查询以报告 大国 的国家名称、人口和面积。

按 任意顺序 返回结果表。
```

**题解：**

```sql
# 解法1：259ms
select
`name`,
population,
`area`  
from 
World
where
`area` >= 3000000
or
population >= 25000000

# 解法2：155ms
SELECT
    name, population, area
FROM
    world
WHERE
    area >= 3000000

UNION

SELECT
    name, population, area
FROM
    world
WHERE
    population >= 25000000
```

**特别说明：**

> 解法1如果where中的两个字段非索引，则可能导致索引失效，在数据量大的时候可以选择解法2的union来提高查询效率



### [596. 超过5名学生的课](https://leetcode.cn/problems/classes-more-than-5-students/)

```sql
表: Courses

+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| student     | varchar |
| class       | varchar |
+-------------+---------+
(student, class)是该表的主键列。
该表的每一行表示学生的名字和他们注册的班级。
 

编写一个SQL查询来报告 至少有5个学生 的所有班级。

以 任意顺序 返回结果表。
```

**题解：**

```sql
SELECT
    class
FROM
    (SELECT
        class, COUNT(DISTINCT student) AS num
    FROM
        courses
    GROUP BY class) AS temp_table
WHERE
    num >= 5
```

