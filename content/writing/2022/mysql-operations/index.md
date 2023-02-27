---
title: "MySQL 数据库操作"
subtitle: ""
date: 2022-09-29T13:43:16+08:00
lastmod: 2022-09-29T13:43:16+08:00
draft: false
toc: true
comments: true
tags:
- Database
- MySQL
---

## 基本查询
```sql
SELECT * FROM students;
```

## 条件查询
```sql
SELECT * FROM students where score >= 80 AND gender='M';
```

score 和 gender 是查询条件
```sql
SELECT * FROM students WHERE NOT class_id = 2;
```

按 NOT 条件查询 students，查找条件不为 id=2 的数据
```sql
SELECT * FROM students WHERE score < 80 OR score > 90 AND gender = 'M';
```

如果不加括号，条件运算按照 NOT、AND、OR 的优先级进行，即 NOT 优先级最高，其次是 AND，最后是 OR。加上括号可以改变优先级。

## 投影查询
```sql
SELECT id, score points, name FROM students;
```

SELECT 语句将列名 score 重命名为 points，而 id 和 name 列名保持不变

## 排序
```sql
SELECT id, name, gender, score FROM students ORDER BY score;
```

默认按 score 从低到高
```sql
SELECT id, name, gender, score FROM students ORDER BY score DESC;
```

按照成绩从高到底排序，我们可以加上 DESC 表示 “倒序”
```sql
SELECT id, name, gender, score FROM students ORDER BY score DESC, gender;
```

如果 score 列有相同的数据，要进一步排序，可以继续添加列名。例如，使用 ORDER BY score DESC, gender 表示先按 score 列倒序，如果有相同分数的，再按 gender 列排序

## 分页查询
```sql
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 0;
```

我们把结果集分页，每页 3 条记录。要获取第 1 页的记录，可以使用 LIMIT 3 OFFSET 0
LIMIT3 表示每页最多 3 条记录，0 是结果集从 0 号记录开始。如果想查看第二页的记录，将 offset 设为 3.

## 聚合查询
> 对于统计总数、平均数这类计算，SQL 提供了专门的聚合函数，使用聚合函数进行查询，就是聚合查询，它可以快速获得结果

```sql
SELECT COUNT(*) num FROM students;
```
COUNT (*) 表示查询所有列的行数，要注意聚合的计算结果虽然是一个数字，但查询的结果仍然是一个二维表，只是这个二维表只有一行一列，并且列名是 COUNT (*)。

通常，使用聚合查询时，我们应该给列名设置一个别名（num），便于处理结果

> 除了 COUNT () 函数外，SQL 还提供了如下聚合函数

| 函数   | 说明                  |
|------|---------------------|
 | SUM  | 计算某一列的合计值，该列必须为数值类型 |
| AVG	 | 计算某一列的平均值，该列必须为数值类型 |
| MAX	 | 计算某一列的最大值           |
| MIN	 | 计算某一列的最小值           |

注意，MAX () 和 MIN () 函数并不限于数值类型。如果是字符类型，MAX () 和 MIN () 会返回排序最后和排序最前的字符
```sql
SELECT AVG(score) average FROM students WHERE gender = 'M';
```
使用聚合查询计算男生平均成绩

## 分组查询
```sql
SELECT COUNT(*) num FROM students GROUP BY class_id;
```
按 class_id 分组
```sql
SELECT class_id, COUNT(*) num FROM students GROUP BY class_id;
```
但是这 3 行结果分别是哪三个班级的，不好看出来，所以我们可以把 class_id 列也放入结果集中

## 多表查询
```sql
SELECT * FROM students, classes;
```
利用投影查询的 “设置列的别名” 来给两个表各自的 id 和 name 列起别名

```sql
SELECT
students.id sid,
students.name,
students.gender,
students.score,
classes.id cid,
classes.name cname
FROM students, classes;
```

给表名取别名
```sql
SELECT
s.id sid,
s.name,
s.gender,
s.score,
c.id cid,
c.name cname
FROM students s, classes c;
```

## 连接查询
> 连接查询是另一种类型的多表查询。连接查询对多个表进行 JOIN 运算，简单地说，就是先确定一个主表作为结果集，然后，把其他表的行有选择性地 “连接” 在主表结果集上。

## 内连接 INNER JOIN
1. 先确定主表，仍然使用 FROM <表 1> 的语法；
2. 再确定需要连接的表，使用 INNER JOIN <表 2> 的语法；
3. 然后确定连接条件，使用 ON <条件...>，这里的条件是 s.class_id = c.id，表示 students 表的 class_id 列与 classes 表的 id 列相同的行需要连接；
4. 可选：加上 WHERE 子句、ORDER BY 等子句。

```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
INNER JOIN classes c
ON s.class_id = c.id;
```

选出所有学生，同时返回班级名称

## 外连接 OUTER JOIN
```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
RIGHT OUTER JOIN classes c
ON s.class_id = c.id;
```

## 查询范式
```sql
SELECT ... FROM tableA ??? JOIN tableB ON tableA.column1 = tableB.column2;
```
