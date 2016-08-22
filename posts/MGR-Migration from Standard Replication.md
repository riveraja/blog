## MySQL Group Replication

#### Offline migration from standard MySQL replication to MySQL Group Replication

1. FTWRL on master server, and make sure the replicas are in sync
2. Stop all applications and shutdown each mysql node
3. Bootstrap one node, take note of 'SHOW MASTER STATUS' output
4. Start other nodes and run 'RESET MASTER', 'SET @@global.gtid_purged={master_status}', 'CHANGE MASTER TO ... FOR CHANNEL', stop and start GROUP_REPLICATION
5. Edit my.cnf on bootstrapped node and set 'group_replication_bootstrap_group = OFF'

#### Online migration from standard MySQL replication to MySQL Group Replication

Assumptions:
- Original packages installed are Oracle MySQL 5.7.14 Community Edition without the Group Replication plugin
- GTID is enabled
- Standard replication without replication filtering

1. Shutdown mysqld instance on node2
2. Upgrade node2's mysql packages to 5.7.14 with group replication plugin
3. Start mysqld instance on node2 with `group_replication_bootstrap_group = ON`
4. Start group_replication `STOP GROUP_REPLICATION; START GROUP_REPLICATION;` this is the sequence since we have `group_replication_start_on_boot = ON`

5. Shutdown mysqld instance on node3

6. Wait for a minute and execute `STOP SLAVE; SHOW SLAVE STATUS\G START SLAVE;` on node2, note the Executed_Gtid_Set value on node2 as gtid_set

```
Executed_Gtid_Set: 0440256a-6828-11e6-a097-00163ed83514:1-7351,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-2,
c8690318-6828-11e6-88a7-00163e4aaba7:1
```

7. Upgrade node3's mysql packages to 5.7.14 with group replication plugin and add `skip-slave-start` in it's my.cnf
8. Start mysqld instance on node3, start replication on node3

`START SLAVE UNTIL SQL_AFTER_GTIDS='0440256a-6828-11e6-a097-00163ed83514:1-7351';`

8) Check `SHOW SLAVE STATUS\G` on node3 and once it shows `Slave_SQL_Running: No` execute `STOP SLAVE; RESET SLAVE ALL; RESET MASTER;` (on node3)
9) Execute `CHANGE MASTER TO MASTER_USER='rplusr',MASTER_PASSWORD='rp1_Pwd#0' FOR CHANNEL 'group_replication_recovery';` on node3
10) Set gtid_purged on node3
`SET @@GLOBAL.GTID_PURGED='0440256a-6828-11e6-a097-00163ed83514:1-7351, 9bd4bf53-61bf-11e6-aa38-00163e772659:1-2, c8690318-6828-11e6-88a7-00163e4aaba7:1'`

11) Start group replication on node3 with `STOP GROUP_REPLICATION; START GROUP REPLICATION;`
```
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 6f69332a-684d-11e6-8689-00163ee2bfdb | grprepl3    |        3306 | ONLINE       |
| group_replication_applier | c8690318-6828-11e6-88a7-00163e4aaba7 | grprepl2    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
2 rows in set (0.00 sec)

mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:2
                         MEMBER_ID: 6f69332a-684d-11e6-8689-00163ee2bfdb
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 10
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-9993,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-3,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 0440256a-6828-11e6-a097-00163ed83514:9993
1 row in set (0.00 sec)
```
12) Execute `FLUSH TABLES WITH READ LOCK` on the standard master node, also execute `SHOW MASTER STATUS;` optionally.
13) Execute `SHOW SLAVE STATUS\G` and `SELECT * FROM PERFORMANCE_SCHEMA.REPLICATION_GROUP_MEMBER_STATS\G` on node2 and node3. At this point master server and the group replication cluster (node2 and node3) is in sync
Standard Master
```
mysql> show master status;
+---------------------+----------+--------------+------------------+----------------------------------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                            |
+---------------------+----------+--------------+------------------+----------------------------------------------+
| grprepl1-bin.000002 | 66174477 |              |                  | 0440256a-6828-11e6-a097-00163ed83514:1-10003 |
+---------------------+----------+--------------+------------------+----------------------------------------------+
1 row in set (0.00 sec)
```
Node2 Group Replication node
```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:2
                         MEMBER_ID: c8690318-6828-11e6-88a7-00163e4aaba7
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 7531
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 119
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-9993,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-3,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 0440256a-6828-11e6-a097-00163ed83514:10003
1 row in set (0.00 sec)
```
Node3 Group Replication node
```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:2
                         MEMBER_ID: 6f69332a-684d-11e6-8689-00163ee2bfdb
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 20
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 119
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-9993,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-3,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 0440256a-6828-11e6-a097-00163ed83514:10003
1 row in set (0.00 sec)
```
14) Move all applications to point on node2, once all applications are pointed to node2, stop mysqld instance on node1
15) Optionally, set `super_read_only=1` on node1 as well.
16) Shutdown mysqld instance on node1 to prepare it to join the group replication cluster
17) Upgrade mysqld packages on node1 to 5.7.14 with group replication plugin
18) On node2, execute `STOP SLAVE; RESET SLAVE ALL;`
19) Start mysqld instance on node1, execute `STOP GROUP_REPLICATION` then `RESET MASTER;` and the same `CHANGE MASTER TO` in step 9.
20) Set gtid_purged value on node1 from info in step 13.
```
mysql> set @@global.gtid_purged='0440256a-6828-11e6-a097-00163ed83514:1-9993, 9bd4bf53-61bf-11e6-aa38-00163e772659:1-3, c8690318-6828-11e6-88a7-00163e4aaba7:1';
Query OK, 0 rows affected (0.01 sec)
```
21) Start group replication on node1 and wait until `MEMBER _STATE` changes to ONLINE from RECOVERING:
```
mysql> start group_replication;
Query OK, 0 rows affected (2.03 sec)

mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:5
                         MEMBER_ID: 0440256a-6828-11e6-a097-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 1
        COUNT_TRANSACTIONS_CHECKED: 0
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 
    LAST_CONFLICT_FREE_TRANSACTION: 
1 row in set (0.00 sec)

mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 0440256a-6828-11e6-a097-00163ed83514 | grprepl1    |        3306 | RECOVERING   |
| group_replication_applier | 6f69332a-684d-11e6-8689-00163ee2bfdb | grprepl3    |        3306 | ONLINE       |
| group_replication_applier | c8690318-6828-11e6-88a7-00163e4aaba7 | grprepl2    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
3 rows in set (0.00 sec)

mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 0440256a-6828-11e6-a097-00163ed83514 | grprepl1    |        3306 | ONLINE       |
| group_replication_applier | 6f69332a-684d-11e6-8689-00163ee2bfdb | grprepl3    |        3306 | ONLINE       |
| group_replication_applier | c8690318-6828-11e6-88a7-00163e4aaba7 | grprepl2    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
3 rows in set (0.00 sec)
```
22) Finally check group member status on all nodes.
Node1
```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:5
                         MEMBER_ID: 0440256a-6828-11e6-a097-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 0
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 
    LAST_CONFLICT_FREE_TRANSACTION: 
1 row in set (0.00 sec)

mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:5
                         MEMBER_ID: 0440256a-6828-11e6-a097-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 460
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 2243
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-11875,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-631,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:1091
1 row in set (0.00 sec)
```
Node2
```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:5
                         MEMBER_ID: c8690318-6828-11e6-88a7-00163e4aaba7
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 10489
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 2243
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-11875,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-631,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:1091
1 row in set (0.01 sec)
```
Node3
```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14718609255192390:5
                         MEMBER_ID: 6f69332a-684d-11e6-8689-00163ee2bfdb
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 2978
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 2243
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 0440256a-6828-11e6-a097-00163ed83514:1-11875,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-631,
c8690318-6828-11e6-88a7-00163e4aaba7:1
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:1091
1 row in set (0.01 sec)
```
