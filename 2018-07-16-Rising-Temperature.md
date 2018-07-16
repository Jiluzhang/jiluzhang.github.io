---
title: LeetCode - Rising Temperature
description: SQL查询比前一天温度更高的日期
categories:
 - LeetCode
tags:
 - join
 - date 
 - case
---

Given a `Weather` table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.

```
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```

For example, return the following Ids for the above `Weather` table:

```
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

`注意：Id的排列未必是按顺序的！！！`



####解法一（Runtime: 338ms）

```sql
select w1.Id from Weather w1 inner join Weather w2
on TO_DAYS(w1.RecordDate) = TO_DAYS(w2.RecordDate) + 1
and w1.Temperature > w2.Temperature;
```

```sql
inner join (内连接，或等值连接)：获取两个表中字段匹配关系的记录
left join (左连接)：获取左表所有记录，即使右表没有对应匹配的记录
right join (右连接)：获取右表所有记录，即使左表没有对应匹配的记录

select * from tcount_tbl;
>>  +---------------+--------------+
    | runoob_author | runoob_count |
    +---------------+--------------+
    | 菜鸟教程      |           10 |
    | RUNOOB.COM    |           20 |
    | Google        |           22 |
    +---------------+--------------+
    3 rows in set (0.00 sec)
    
select * from runoob_tbl;
>> +-----------+---------------+---------------+-----------------+
   | runoob_id | runoob_title  | runoob_author | submission_date |
   +-----------+---------------+---------------+-----------------+
   |         1 | 学习 PHP      | 菜鸟教程      | 2017-04-12      |
   |         2 | 学习 MySQL    | 菜鸟教程      | 2017-04-12      |
   |         3 | 学习 Java     | RUNOOB.COM    | 2015-05-01      |
   |         4 | 学习 Python   | RUNOOB.COM    | 2016-03-06      |
   |         5 | 学习 C        | FK            | 2017-04-05      |
   +-----------+---------------+---------------+-----------------+
   5 rows in set (0.00 sec)

select a.runoob_id, a.runoob_author, b.runoob_count from runoob_tbl a inner join tcount_tbl b on a.runoob_author = b.runoob_author;
>> +-----------+---------------+--------------+
   | runoob_id | runoob_author | runoob_count |
   +-----------+---------------+--------------+
   |         1 | 菜鸟教程      |           10 |
   |         2 | 菜鸟教程      |           10 |
   |         3 | RUNOOB.COM    |           20 |
   |         4 | RUNOOB.COM    |           20 |
   +-----------+---------------+--------------+
   4 rows in set (0.00 sec)

select a.runoob_id, a.runoob_author, b.runoob_count from runoob_tbl a left join tcount_tbl b on a.runoob_author = b.runoob_author;
>> +-----------+---------------+--------------+
   | runoob_id | runoob_author | runoob_count |
   +-----------+---------------+--------------+
   |         1 | 菜鸟教程      |           10 |
   |         2 | 菜鸟教程      |           10 |
   |         3 | RUNOOB.COM    |           20 |
   |         4 | RUNOOB.COM    |           20 |
   |         5 | FK            |         NULL |
   +-----------+---------------+--------------+
   5 rows in set (0.00 sec)

select a.runoob_id, a.runoob_author, b.runoob_count from runoob_tbl a right join tcount_tbl b on a.runoob_author = b.runoob_author;
>> +-----------+---------------+--------------+
   | runoob_id | runoob_author | runoob_count |
   +-----------+---------------+--------------+
   |         1 | 菜鸟教程      |           10 |
   |         2 | 菜鸟教程      |           10 |
   |         3 | RUNOOB.COM    |           20 |
   |         4 | RUNOOB.COM    |           20 |
   |      NULL | NULL          |           22 |
   +-----------+---------------+--------------+
   5 rows in set (0.00 sec)


# to_days()用法
to_days()会返回给定的一个日期对应的一个天数(从年份0开始的天数)。
MySQL的"日期和时间类型"中的规则将日期中的二位数年份值转化为四位。例如，'1997-10-07'和'97-10-07'被视为相同的日期。
select to_days('2015-10-31')
>> +-----------------------+
   | to_days('2015-10-31') |
   +-----------------------+
   |                736267 |
   +-----------------------+
   1 row in set (0.00 sec)

select to_days('2015-11-01');
>> +-----------------------+
   | to_days('2015-11-01') |
   +-----------------------+
   |                736268 |
   +-----------------------+
   1 row in set (0.00 sec)
```



#### 解法二（Runtime: 339ms）

```sql
select w1.Id from Weather w1, Weather w2
where w1.Temperature > w2.Temperature and datediff(w1.RecordDate, w2.RecordDate) = 1;

# "datediff(w1.RecordDate, w2.RecordDate) = 1" can be replaced by "to_days(w1.RecordDate) = to_days(w2.RecordDate) + 1" (Runtime: 341ms)
# "subdate(w1.RecordDate, 1) = w2.RecordDate"  (Runtime: 857ms)
```



#### 解法三（Runtime: 216ms）

```sql
select Id from (
select case when Temperature > @pre_t and datediff(RecordDate, @pre_d) = 1 then Id else null end as Id,
@pre_t := Temperature, @pre_d := RecordDate
from Weather, (select @pre_t := null, @pre_d := null) as init order by RecordDate asc
) Id where Id is not null;
```

```sql
# 使用两个变量pre_t和pre_d分别表示上一个温度和上一个日期，然后当前温度要大于上一温度，且日期差为1，满足上述两个条件就选出来为Id，否则为null，然后更新pre_t和pre_d为当前的值，最后选出的Id不为空即可The most number 
# @pre_t 变量声明，:= 赋值
# (select @pre_t := null, @pre_d := null) as init  新增两列的表，并且赋予默认值；单独派生出一个表时要加别名，不然会出现错误
# case 可理解为“如果”
```





















