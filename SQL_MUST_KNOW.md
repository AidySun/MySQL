# SQL Must Know


## Common tips

`SELECT ... FROME ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...`

### EXIST v.s. IN

* the common sence is that for nested loops, big loop inside would be faster. 
```
IN表是外边和内表进行hash连接，是先执行子查询。
EXISTS是对外表进行循环，然后在内表进行查询。
因此如果外表数据量大，则用IN，如果外表数据量小，也用EXISTS。
IN有一个缺陷是不能判断NULL，因此如果字段存在NULL值，则会出现返回，因为最好使用NOT EXISTS。
====
SELECT * FROM A WHERE cc IN (SELECT cc FROM B)
SELECT * FROM A WHERE EXIST (SELECT cc FROM B WHERE B.cc=A.cc)
当A小于B时，用EXIST。因为EXIST的实现，相当于外表循环，实现的逻辑类似于：
for i in A
    for j in B
        if j.cc == i.cc then ...

当B小于A时，用IN，因为实现的逻辑类似于：
for i in B
    for j in A
        if j.cc == i.cc then ...
所以哪个表小就用哪个表来驱动，A表小 就用EXIST，B表小 就用IN
```

### SQL Execution Progress
1. FROM
  * CROSS JOIN -> vt1 (virtual table 1)
  * ON -> filter on vt1 -> vt2
  * add external rows for JOIN (left, right, cross) -> vt3
  * repeat above if there are more tables to JOIN
1. WHERE
1. GROUP BY
1. cluster function
1. HAVING
1. expression calculation
1. SELECT
1. DISTINCT
1. ORDER BY
1. LIMIT

## ORDER BY

* default by ASC
* last in select syntax

## DISTINCT

* must appear at the beginning of columns
* affer all columns after it

### GROUP BY
* By default, it sorts result records
  * But, if it has `GROUP BY` before it, it sorts the **groups**, **not** the records in groups

### LIMIT
* stop finding as soon as count matched

### Wildcards
```
_ : single char
% : one or more char, cannot match NULL
```

### count
* time : `count(*)` = `count(1)` < `count(column)`
  * for `MySQL ISAM`, `count(*)` is `O(1)`


## Oracle

#### Execution Path
* Syntax Analysis -> Semantic Analysis -> Permissiong checking 
  -> shared pool -> hard analysis -> optimizitor 
      |-> soft analysis  -> executor  <- |

## MySQL
Cient-server mode, server is `mysqld`

#### Execution Path in mysqld
* connecting layer
* SQL layer
* storage engine layer
  * InnoDB: transaction, row lock, foreign key restraint
  * MyISAM: fast, low resource; no-transaction, no-foreign key
  * Memory
  * NDB - cluster
  * Archive 

#### PROCEDURE

```
DECLARE @id INT = 1;
SET @id = @id + 1;
```
