# SQL Must Know

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
