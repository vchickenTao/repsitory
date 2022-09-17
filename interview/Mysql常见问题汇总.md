### DateDiff 函数	

**描述：**
返回两个日期之间的时间间隔。
**语法：**
DateDiff(interval, date1, date2 [,firstdayofweek[, firstweekofyear]])

### 四大排名函数
#### 一、ROW_NUMBER()
Row_number() 在排名是序号 连续 不重复，即使遇到表中的两个一样的数值亦是如此

select *,row_number() OVER(order by number ) as row_num
from num 

数据如下：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/71b837b2-2dd2-4d75-92ed-28f62c53f146.png)


结果如图：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/50bedb88-ef0e-4a6b-8c20-0bb28882d1b0.png)


注意：在使用row_number() 实现分页时需要特别注意一点，over子句中的order by 要与SQL排序记录中的order by保持一致，否则得到的序号可能不是连续的
select *,row_number() OVER(order by number ) as row_num
from num
ORDER BY id

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/12dfd2b8-b6b1-43e4-88fb-ea8ccdaa2040.png)


#### 二、rank()
Rank() 函数会把要求排序的值相同的归为一组且每组序号一样，排序不会连续执行

select *,rank() OVER(order by number ) as row_num
from num 

结果如下：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/eff31d86-e981-42d6-81c0-7188b96c7999.png)


#### 三、dense_rank()
Dense_rank() 排序是连续的，也会把相同的值分为一组且每组排序号一样

select *,dense_rank() OVER(order by number ) as row_num
from num 

结果如下：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/a1569dbb-15da-48c2-808c-28dc2bf7a487.png)


#### 四、ntile()
Ntile(group_num) 将所有记录分成group_num个组，每组序号一样

select *,ntile(2) OVER(order by number ) as row_num
from num 

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-30/52e7c2c0-a85e-441d-85e6-c03c5e434854.png)

### MySQL-窗口函数
#### 一、定义
窗口可以理解为`记录集合`，窗口函数就是在满足某种条件的记录集合上执行的特殊函数。
即：应用在窗口内的函数。
**静态窗口：**每条记录都要在此窗口内执行函数，窗口大小都是固定的。
**动态窗口：**不同的记录对应着不同的窗口，这种动态变化的窗口叫滑动窗口。

#### 二、语法格式
```mysql
函数名(字段名) over(子句) 
```
over()括号内若不写，则意味着窗口函数基于满足where条件的所有行进行计算。
若括号内不为空，则支持以下语法来设置窗口。
```mysql
函数名(字段名) over(partition by <要分列的组> order by <要排序的列> rows between <数据范围>) 
```
数据范围：
```mysql
rows between 2 preceding and current row # 取本行和前面两行

rows between unbounded preceding and current row # 取本行和之前所有的行 

rows between current row and unbounded following # 取本行和之后所有的行 

rows between 3 preceding and 1 following # 从前面三行和下面一行，总共五行 

# 当order by后面没有rows between时，窗口规范默认是取本行和之前所有的行

# 当order by和rows between都没有时，窗口规范默认是分组下所有行（rows between unbounded preceding and unbounded following） 
```

> Mysql分组的partition by 与group by的区别？https://zhuanlan.zhihu.com/p/423702371

> OVER()窗口函数: https://blog.csdn.net/weixin_46544385/article/details/120609601

#### 三、分类
**1、聚合类**
**聚合窗口函数与普通聚合函数的区别：**
- 普通场景下的聚合函数是将多条记录聚合为一条(**多到一**)；
- 窗口函数是每条记录都会执行，有几条记录执行完还是几条(**多到多**)。

先创建user_trade表：
```sql
create table user_trade (
 user_name  varchar(20) COMMENT '用户名',
 piece  int COMMENT '购买数量',
 price  double COMMENT '价格',
 pay_amount double COMMENT '支付金额',
 goods_category varchar(20) COMMENT '商品品类',
 pay_time date COMMENT '支付日期'
);
```
> 导入数据：https://gitee.com/hu-weiqing/datasource/blob/master/user_trade.xlsx

- **累计求和：sum()over()**

```sql
-- 需求1: 查询出2019年每月的支付总额和当年累积支付总额 
select a.mon,a.pay_amount,sum(a.pay_amount) over(order by a.mon) as sum_amount
from(

select month(a.pay_time) as mon,sum(a.pay_amount) as pay_amount
from user_trade a
where year(a.pay_time) = '2019'
group by month(a.pay_time)
) a ;
```
![需求1: 查询出2019年每月的支付总额和当年累积支付总额 ](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/27c83fa1-3409-44db-9e34-4b06515e611e.png)

```sql
-- 需求2:查询出2018-2019年每月的支付总额和当年累积支付总额
select a.*,sum(a.pay_amount) over(partition by a.year order by a.mon) as sum_amount
from(

select year(a.pay_time) as year,month(a.pay_time) as mon,sum(a.pay_amount) as pay_amount
from user_trade a
where year(a.pay_time) in('2018','2019')
group by year(a.pay_time),month(a.pay_time)
) a ;
```
![需求2:查询出2018-2019年每月的支付总额和当年累积支付总额](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/c465b036-6ad5-4d7c-a10b-a464ed69de19.png)

- **移动平均：avg() over()**

```sql
-- 需求3: 查询出2019年每个月的近三月移动平均支付金额
select a.mon,a.pay_amount,
avg(a.pay_amount) over(order by a.mon rows between 2 preceding and current row) as avg_amount
from(

select month(a.pay_time) as mon,sum(a.pay_amount) as pay_amount
from user_trade a
where year(a.pay_time) = '2019'
group by month(a.pay_time)
) a ;
```
![查询出2019年每个月的近三月移动平均支付金额](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/ee3aead0-db0c-4bbe-8216-c94a23c0ec0d.png)

- **最大/最小值：max()/min() over()**

```sql
-- 需求4: 查询出每四个月的最大月总支付金额
select 
a.mon,
a.pay_amount,
max(a.pay_amount) over(order by a.mon rows between 3 preceding and current row) as max_amount
from(

select SUBSTRING(a.pay_time,1,7) as mon,sum(a.pay_amount) as pay_amount
from user_trade a
group by SUBSTRING(a.pay_time,1,7)
)a ;
```
![查询出每四个月的最大月总支付金额](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/7860d62c-4b11-4353-87c8-b47e8defc033.png)

**2、排序类**
- **row_number()、rank() 和dense_rank()**

```sql
-- 需求5: 2020年1月，购买商品品类数的用户排名
SELECT 
a.user_name,
COUNT(DISTINCT a.goods_category) AS cat_num,
ROW_NUMBER() over(ORDER BY COUNT(DISTINCT a.goods_category)) AS rank1,
rank() over(ORDER BY COUNT(DISTINCT a.goods_category)) AS rank2,
DENSE_RANK() over(ORDER BY COUNT(DISTINCT a.goods_category)) AS rank3
FROM user_trade a
WHERE SUBSTRING(a.pay_time,1,7) = '2020-01'
GROUP BY a.user_name;
```
![2020年1月，购买商品品类数的用户排名](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/3872d89f-cac1-4c0a-ae6c-b06059e46808.png)

**row_number()、rank() 和dense_rank() 三种排序函数的区别：**

1. **row_number**：每一行记录生成一个序号，依次排序且不会重复。 1234...（正常排序）
2. **rank**：跳跃排序，生成的序号有可能不连续。1134..（理解为考试成绩排名，并列第一，成绩第二自然成了第三名，跳跃排序）
3. **dense_rank**：在生成序号时是连续的。1123...（密集排序）

- **ntile(n)over()**
`ntile(n)`用于将分组数据按照顺序切分成n片，返回当前切片值，n表示切片的数量；**不支持rows between**。

```sql
-- 需求6: 查询出将2020年2月的支付用户，按照支付金额分成5组后的结果
select 
a.user_name,
sum(a.pay_amount) as pay_amount,
ntile(5) over(order by sum(a.pay_amount) desc) as level
from user_trade a
where SUBSTRING(a.pay_time,1,7) = '2020-02'
group by a.user_name;

-- 需求7: 查询出2020年支付金额排名前30%的所有用户
select a.user_name,a.pay_amount
from (
select 
a.user_name,
sum(a.pay_amount) as pay_amount,
ntile(10) over(order by sum(a.pay_amount) desc) as level
from user_trade a
where year(a.pay_time) = '2020'
group by a.user_name
) a 
where a.level in(1,2,3);
```
![查询出将2020年2月的支付用户，按照支付金额分成5组后的结果](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/5b870020-058e-46c5-840d-27fc23721d6e.png)

![查询出2020年支付金额排名前30%的所有用户](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/e5aa9235-d1ba-4815-811d-16f1c9ab9805.png)

**3、偏移分析函数**

- **lag() over()向上偏移**
lag(exp_str,offset,defval)
exp_str：字段名
offset：偏移量
defval：默认值。当向上偏移了offset行已经超出了表的范围时，lag()函数将defval这个参数值作为函数的返回值，若没有指定默认值，则返回NULL。
```sql
-- 需求8: 查询出King和West的时间偏移(前N行)
select a.user_name,a.pay_time,
lag(a.pay_time,1,a.pay_time) over(partition by a.user_name order by a.pay_time) as lag1,
-- 没有传入偏移量，那么默认就是1，找不到的话，此处也没有给默认值，为null
lag(a.pay_time) over(partition by a.user_name order by a.pay_time) as lag2,
lag(a.pay_time,2,a.pay_time) over(partition by a.user_name order by a.pay_time) as lag3,
lag(a.pay_time,2) over(partition by a.user_name order by a.pay_time) as lag4
from user_trade a 
where a.user_name in('King','West');
```
![查询出King和West的时间偏移(前N行)](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/2d7d10ae-3109-45ba-80b6-db2073c0c870.png)

- **lead() over()向下偏移**
用法同lag()over()函数。

```sql
-- 需求9: 查询出支付时间间隔超过100天的用户数
select count(distinct a.user_name)
from (
select a.user_name,a.pay_time,
lag(a.pay_time) over(partition by a.user_name order by a.pay_time) as lg
from user_trade a 
) a 
where DATEDIFF(a.pay_time,a.lg) >100;

# 需求9运行结果为180


-- 需求10: 查询出每年支付时间间隔最长的用户
select c.years,c.user_name,c.pay_days 
from(

select b.years,b.user_name,datediff(b.pay_time,b.lg) as pay_days,
rank() over(partition by b.years order by datediff(b.pay_time,b.lg) desc) as rk 
from (

select year(a.pay_time) as years,a.user_name,a.pay_time,
lag(a.pay_time) over(partition by a.user_name,year(a.pay_time) order by a.pay_time) as lg
from user_trade a 
) b 
where b.lg is not null
) c 
where c.rk = 1;
```
![查询出支付时间间隔超过100天的用户数](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/786af4f9-b672-46d4-9bde-bbea2e3ef7c6.png)

![查询出每年支付时间间隔最长的用户](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-29/a5f36437-44ea-49ff-8b94-919cd16796ff.png)

> 本文原作者：`HC户户` 文章地址：https://zhuanlan.zhihu.com/p/409561328 


### IF函数
IF函数根据判断条件是否成立进行选择执行，成立时执行一条语句，不成立时执行另一条语句

**语法结构**
```sql
IF(condition, value_if_true, value_if_false)

# 参数说明:
condition: 判断条件
value_if_true: 如果condition的结果为TRUE，返回该值
value_if_false: 如果condition的结果为FALSE，返回该值
```

### 对count(*)、count(列）、 count(1) 理解
- `count(*)` 包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL

- `count(1)` 包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
- `count(列名)` 只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

### mysql 安全等于（<=>）

`安全等于 <=>` 

- IS NULL:仅仅可以判断null值
- <=>:既可以 判断null，又可以判断普通的数值，可读性较低

**案例：**

```sql
SELECT name FROM customer WHERE referee_id IS NULL OR referee_id != 2
等价于
SELECT name FROM customer WHERE NOT referee_id <=> 2;
```



