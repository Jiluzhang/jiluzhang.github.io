---
title: LeetCode - Second Highest Salary
description: 查询第二高的工资
categories:
 - LeetCode
tags:
 - where
 - max
 - distinct
---




Write a SQL query to get the second highest salary from the `Employee` table.

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

For example, given the above Employee table, the query should return `200` as the second highest salary. If there no second highest salary, then the query should return `null`.

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```



##### 解法一（Runtime: 186ms）

```sql
select Salary as SecondHighestSalary from Employee group by Salary
union all (select null as Salary)  
# If there no second highest salary, then the query will return null;
order by SecondHighestSalary desc limit 1 offset 1;

# GROUP BY语法可以根据给定数据列的每个成员对查询结果进行分组统计，最终得到一个分组汇总表

# UNION用于把来自多个SELECT语句的结果组合到一个结果集合中；在多个SELECT语句中，对应的列应该具有相同的字段属性，且第一个SELECT语句中被使用的字段名称也被用于结果的字段名称
# 当使用UNION时，MySQL会把结果集中重复的记录删掉，而使用UNION ALL，MySQL会把所有的记录返回，且效率高于UNION

# ORDER BY将所有记录按照默认的升序进行排列(即：从1到9， 从a到z)
# LIMIT用于返回指定的记录数，其接受一个或两个数字参数，必须是一个整数常量，第一个参数指定第一个返回记录行的偏移量（初始记录行的偏移量是0），第二个参数指定返回记录行的最大数目
# OFFSET表示偏移量
# Example: SELECT * FROM student limit 9,4           返回表student的第10、11、12、13行
#          SELECT * FROM student limit 4 offset 9    从表student的第10行开始，返回4行
```



##### 解法二（Runtime: 297ms）

```sql
select max(Salary) as SecondHighestSalary from Employee
where Salary not in
(select max(Salary) from Employee);
```



##### 解法三（Runtime: 152ms）

```sql
select max(Salary) as SecondHighestSalary from Employee
where Salary <
(select max(Salary) from Employee);
```



##### 解法四（Runtime: 483ms）【该解法可扩展到寻找Nth高的情况】

```sql
select distinct(Salary) as SecondHighestSalary from Employee E1
where 1 =
(select count(distinct(E2.Salary)) from Employee E2
where E2.Salary > E1.Salary)
union all (select null as Salary)
order by SecondHighestSalary desc limit 1;

# 对Employee中的数据进行逐行检验where语句是否满足
# 对于第一行E1.Salary为100，则最后括号中的结果是2，不等于1
# 对于第二行，结果是1，满足条件，选到
# 对于第三行，结果是0，不等于1
```











