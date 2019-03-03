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
```
* check db status
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
