# MySQL

## Execute Path
* Connector
  * long-time transation will cause OOM
  * solutions for long-time transation
    * disconnect termly
    * `mysql_reset_connect` (after v5.7) will reset to status as just connected. No reconnection and authentication needed.
* Cache  -  Analyzer
  * Cache may not effective enouth. Any update to a table would invalid all its cache.
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
    * view is created at the begining of the execution of SQL
    * default setting of Oracle 
  3. repeatable read
    * view is created at the begining of transation
  4. serializable
    * no view? using locks

```SQL
show variables like 'transation_isolation';
# READ-COMMITTED
```
* Implementation of Isolation
Each update would record one rollback operation. _from the latest status, it can get to previous status using rollback operation. rollback operations would be deleted when there is no SQL would uses it, that is when there is no earlier read-view than the rollback log_
  * this is why long-time transation would cause OOM

* MVCC - multi-version concurrency control

* Ways to Launch Transation
  1. start and commit transation obversely
  2. `set autocommit=1` is recommended
    * commit work and chain would commit the transation and start next transation, this would avoid extral begin

* Get long-time transation longer than 60 seconds
  `select * from information_schema.innodb_trx where TIME_TO_SEC(tiediff(now(), trx_started))>60` 

## Index
* Implemention (data model of database)
  1. Hash table
    * good for fixed value, insertion
    * not good for range-based query
  2. Sorted array
    * good for fixed value, range-based query
      * binary search
    * suitable for static data
    * bad performace for insertion
  3. Binary search tree
    * good for search and insertion `O(logn)`
    * multi-way tree is more suitable for database
  4. jump list, LSM tree...

* Index of InnoDB
InnoDB uses B+ tree to implement its index model.

* key index (culstered index)
* secondary index

Self-increamental key is good for performance and storage.

* Composite index
  * left-prefix principle
    * `select * from T where id like '150%'` can use the index on `id`
  * `key ('id', 'name')` means `order by id, name`

* index condition pushdown (since version 5.6)
  * filter index first, less access to real table data

## Lock


























