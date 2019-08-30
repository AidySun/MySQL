# MySQL

* ORM - object relation mapping
  * Pthon - sqlalchemy

## Execute Path

* Connector
  * long-time transaction will cause OOM
  * solutions for long-time transaction
    * disconnect termly
    * `mysql_reset_connect` (after v5.7) will reset to status as just connected. No reconnection and authentication needed.
* Cache  -  Analyzer
  * Cache may not effective enough. Any update to a table would invalid all its cache.
  * set cache manually
    ```SQL
    # set query_cache_type = DEMAND
    select SQL_CACHE * from T where id=10;
    ```
* Analyzer
  * lexical analysis
  * grammatical analysis
* Optimization
  * handle index, join
* Execution
  * check permission to table 

## Transaction Isolation
* ACID - atomicity, consistency, isolation, durability
* Isolation types
  1. read uncommitted
    * no view
  2. read committed
    * view is created at the beginning of the execution of SQL
    * default setting of Oracle 
  3. repeatable read
    * view is created at the beginning of transaction
  4. serializable
    * no view? using locks

```SQL
show variables like 'transaction_isolation';
# READ-COMMITTED
```

ISOLATION | DIRTY READ | NON REPEATABLE READ | PHANTOM READ
:--:|:--:|:--:|:--:
read-uncommitted | Y | Y | Y
read-committed | N | Y | Y
repeatable-read | N | N | Y
serializable | N | N | N

* Problems (all occurs in the same transation)
  * Dirty Read - read uncommitted data
  * Non Repeatable Read - within one transation, for the same row, two selections get diff values (because of UPDATE)
  * Phantom Read - for the same selection, executes two times get diff count of records (because of INSERT/DELETE)
    * e.g. `select sum(x) from table1;`

* Implementation of Isolation
Each update would record one rollback operation. _from the latest status, it can get to previous status using rollback operation. rollback operations would be deleted when there is no SQL would uses it, that is when there is no earlier read-view than the rollback log_
  * this is why long-time transaction would cause OOM

* MVCC - multi-version concurrency control

* Ways to Launch Transaction
  1. start and commit transaction obversely
  2. `set autocommit=1` is recommended
    * commit work and chain would commit the transaction and start next transaction, this would avoid extra begin

* Get long-time transaction longer than 60 seconds
  `select * from information_schema.innodb_trx where TIME_TO_SEC(tiediff(now(), trx_started))>60` 

## Index

* Implementation (data model of database)
  1. Hash table
    * good for fixed value, insertion
    * not good for range-based query
  2. Sorted array
    * good for fixed value, range-based query
      * binary search
    * suitable for static data
    * bad performance for insertion
  3. Binary search tree
    * good for search and insertion `O(logn)`
    * multi-way tree is more suitable for database
  4. jump list, LSM tree...

* Index of InnoDB
  * InnoDB uses B+ tree to implement its index model.
  * `int` type index in InnoDB, using multiple tree (about 1200).
  * **each index is a B+ tree**

* primary key index (clustered index)
* secondary index (non-primary key)
  ```sql
  CREATE TABLE T (
    id INT PRIMARY KEY, -- primary/clustered index
    k INT NOT NULL,
    name varchar(32),
    INDEX (k)            -- secondary/non-primary index
  ) engine=InnoDB;
  -- The index of table `T` is `(k, id)`.
  ```
  * diff between primary and non-primary index
    * **primary index** stores the entire row data in table
    * **non-primary index** stores the primary index of the row
    * therefore, non-primary index needs to get primary index first, then get data using primary index

* auto-increment primary key 
  ```SQL
  NOT NULL PRIMARY KEY AUTO_INCREMENT
  ```
* Self-incremental primary key is good for performance and storage.
* why auto-increment primary key is recommended generally?
  1. performance - auto-increment has better performance than non-auto-increment key, because B+ tree needs to keep the index in order, when inserting one row in the middle, should move some rows like array.

  2. space - auto-increment key is int, which is smaller than general
       non-int primary key (e.g. ID num).
  * there are also some cases auto-increment primary key is not necessary. Like `key-value` data.

* Covering Index & Joint Index
  * getting the results from index B+ tree directly, without visiting rows.
  * is a general way to improve performance.
  * e.g. `select ID from T where k between 1 and 10` on index `(k, ID)`
  * Order of Joint Index 
    * if having joint index `(a, b)`, it's unnecessary to set index `a`
    * besides, if `a` is longer than `b`, `(a,b)` is good for space.

* Composite index
  * left-prefix principle, e.g. `cardid`
    * `select * from T where id like '150%'` can use the index on `id`
  * `key ('id', 'name')` means `order by id, name`, order by id first.

* index condition pushdown (since version 5.6)
  * filter index first, less access to real table data

* When is necessary to rebuild index?
  * data page may have 'hole' because of deleting, paging
  * rebuild index will create a new index and reorder data, improve the data page performance and shrink table size.
  * `alter table T engine=InnoDB` is another solution to get the same result. _(TODO)_

* THINKING
  ```SQL
  CREATE TABLE `geek` (
    `a` int(11) NOT NULL,
    `b` int(11) NOT NULL,
    `c` int(11) NOT NULL,
    `d` int(11) NOT NULL,
    PRIMARY KEY (`a`,`b`),
    KEY `c` (`c`),
    KEY `ca` (`c`,`a`),
    KEY `cb` (`c`,`b`)
  ) ENGINE=InnoDB;
  ```
  * index `ca` is unnecessary, because `ca` means order by `c` first then by `a`, at the same time `ab` is the primary key, therefore, `cb` has the same order of `c` with `ca`, keeping `cb` is enough.

## Cursor

## Lock
1. Global Lock
    * FTWRL - flush tables with read lock
    * entire database is read-only
    * used for database backup
    * used by engine myISAM, because it does NOT support transaction
      * For engine InnoDB, it supports transaction, therefore it can start a repeatable read transaction to create a view for backup, which wouldn't lock database for updates.
    ```
    Why global lock is preferred rather than 'set global readonly = true'?
      1) sometimes, global readonly is used in business logic 
         E.g. identify the database is primary or secondary
      2) when exception happens, MySQL would release the global lock when using FTWRL, 
         while global readonly won't be set to false automatically
    ```

    * DML _(data manipulation language)_ - add/delete/update/query data
    * DDL _(data definition language)_ - change table structure (alter table)

2. Table Level Lock
    * table lock
      * `lock tables t1 read, t2 write;`
    * MDL - meta data lock
      * not used manually, it is added to tables automatically
      * when add/delete/update/read a table, MDL read lock is added; when changing a table structure, MDL write lock is added
      * read locks are not mutually exclusive(互斥的); read locks and write locks, between write locks are mutually exclusive

    **Note:** MDLs in transaction are created at the beginning of the execution, but won't be released after the execution, they would be release after the committing of transaction.
      * How to change the structure of a small table without blocking(阻塞)?
        * avoid long-time transaction. Long-time transaction would hold the MDL locks. 
          * Before processing DDL, stop DDL first, or, kill long-time transactions which are stored in table `innodb_trx` of db `information_schema`
        * when `kill` doesn't work. set timeout in `alter table` syntax, give up if it cannot get write lock and try later

3. Row Lock


## Types

### DATETIME vs TIMESTAMP

> MySQL converts TIMESTAMP values from the current time zone to UTC for storage, 
> and back from UTC to the current time zone for retrieval. (This does not occur 
> for other types such as DATETIME.) By default, the current time zone for each 
> connection is the server's time. The time zone can be set on a per-connection basis.

### Unicode

* utf8mb4 - 4-byte UTF-8 unicode encoding
* utf8mb3 - 3-byte UTF-8 unicode encoding
  * utf8 is an alias of utf8mb3
* TEXT - 64(K)?


















