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

## mysqlsh

```SQL
# connect
$ mysqlsh user@host/db_name
> \connect user@host/db_name

# exec
> \source /mydb.sql
```
