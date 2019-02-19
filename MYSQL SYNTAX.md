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

## Notes

* How to store array?
```cpp
// There is no array type in database.
// In source code, using following methods to convert between string and array
implode(); // convert array to string
explode(); // convert string array
```
