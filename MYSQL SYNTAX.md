# MySQL Syntax 

## mysql

```SQL
# connect
$ mysql -u root -pmypassword db_name

$ mysql user@host/db_name
> USE db_name;

# check
> SHOW databases;
> SHOW tables;

# exe
$ mysql -u root --sql --file /mydb.sql

# version
select version();

```


* timezone
```SQL
SELECT @@global.time_zone, @@session.time_zone;
select timediff(now(),convert_tz(now(),@@session.time_zone,'+00:00'));

set time_zone = '+00:00'; # set connection/client time zone, not server's
```

## check db status

```SQL
# current connection
> select user();
> select database():

> show variables like 'port';
> show variables like 'character%';

> status;
> show variables like '%max_connections%';
> show status like 'Threads%';

# user info
> select distinct concat('user: ''',user,'''@''',host,''';')
>    as query from mysql.user;

# user permission
> show grants for 'root'@'localhost';

# data file path
> show variables like 'datadir%';

```

## DB analyze tools

1. `explain`
2. `show engine innodb status;    # show db status, e.g. dead lock`

* example
  1. dead lock 
    * preconditions
      * isolation is *repeatable read*
        * `set session transation isolation level repeatable read;`
      * `set session autocommit=0;`

    ```
    create table t1 (
    	...
    	cell varchar(20) unique
   	)engine=innodb;

   	# transation 1
   	start transation;
   	insert into t1(cell) values (4444);
   										# transation 2
   										start transation;
   										insert into t1(cell) values(5555);
   	update t1 set cell=4000 where cell=4444;	# lock here
   										update t1 set cell=5000 where cell=5555; 	# roll back transation 2
    ```
    * root couse
      * `cell` type is `char`, but update uses `int` which creates the table lock.
      * add quote would fix the dead lock

* profiling
```
select @@profiling;
set profiling=1;
show profiles;
show profile;  # last execution

```

## Tips

* when the index would be ignored
  * colume types are not matched, `where id=111;` while `id char`
  * `join` tables with diff encoding, e.g. one table is `utf-8` and the other is `latin`


## mysqlsh

```SQL
# connect
$ mysqlsh user@host/db_name
> \connect user@host/db_name

# exec
> \source /mydb.sql

```

## Notes

* How to store array?
```cpp
// There is no array type in database.
// In source code, using following methods to convert between string and array
implode(); // convert array to string
explode(); // convert string array
```

## JOIN

* INNER JOIN : both existing in A and B
* LEFT JOIN  : rows in A and maching records in B
* RIGHT JOIN : all rows in B and maching records in A
* FULL JOIN  : all rows in A and B, maching or not

```SQL
SELECT a.id, a.name, b.icon 
FROM a 
LEFT JOIN b ON a.id=b.id 
WHERE a.id=1;
```

## SELECT

## INSERT

* Insert if not exist
```SQL
INSERT INTO table_listnames (name, address, tele)
SELECT * FROM (SELECT 'John', 'Doe', '022') AS tmp
WHERE NOT EXISTS (
    SELECT name FROM table_listnames WHERE name = 'John'
) LIMIT 1;
```

* Update if exist
```SQL
REPLACE INTO `table` VALUES (5, 'John', 'Doe', SHA1('password')); 
# OR
INSERT INTO Devices(unique_id, time) 
VALUES('$device_id', '$current_time') 
ON DUPLICATE KEY UPDATE time = '$current_time';
```

## DELETE

## Transation

```SQL
SET TRANSATION ISOLATION LEVEL READ COMMITTED;
```

## Enable Slow Selection Log
`show variables like "long%" / "slow%";`

```yml
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/slow.log
long_query_time = 1
```