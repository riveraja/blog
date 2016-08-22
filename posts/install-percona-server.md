## MySQL Group Replication

#### Setting up a 3-node group replication cluster

Start mysqld instance on the bootstrap node (grprepl1) with:

```
group_replication_bootstrap_group = ON
```

After initial startup, change mysql root temporary password and then stop then start group replication.

```
STOP GROUP_REPLICATION; START GROUP_REPLICATION;
```

Execute 'SHOW MASTER STATUS' to take note of the Executed GTID Set.

```
mysql> show master status;
+---------------------+----------+--------------+------------------+----------------------------------------------------------------------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                                                                |
+---------------------+----------+--------------+------------------+----------------------------------------------------------------------------------+
| grprepl1-bin.000002 |      750 |              |                  | 553987cc-6768-11e6-888a-00163ed83514:1-2, 9bd4bf53-61bf-11e6-aa38-00163e772659:1 |
+---------------------+----------+--------------+------------------+----------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

Create the replication user on the bootstrapped node:

```
mysql> create user 'rplusr'@'10.0.3.%' identified by 'rp1_Pwd#0';
Query OK, 0 rows affected (0.01 sec)

mysql> grant replication slave on *.* to 'rplusr'@'10.0.3.%';
Query OK, 0 rows affected (0.00 sec)
```

Check group replication status:

```
mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14717610423503000:1
                         MEMBER_ID: 553987cc-6768-11e6-888a-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 2
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 553987cc-6768-11e6-888a-00163ed83514:1-2,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-2
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:3
1 row in set (0.00 sec)

mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 553987cc-6768-11e6-888a-00163ed83514 | grprepl1    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
1 row in set (0.00 sec)
```

On the second node, start mysqld instance and change mysql root temporary password.

```
mysql> reset master;
Query OK, 0 rows affected (0.02 sec)

mysql> change master to master_user='rplusr',master_password='rp1_Pwd#0' for channel 'group_replication_recovery';
Query OK, 0 rows affected, 2 warnings (0.05 sec)

mysql> set @@global.gtid_purged='553987cc-6768-11e6-888a-00163ed83514:1-2,9bd4bf53-61bf-11e6-aa38-00163e772659:1';
Query OK, 0 rows affected (0.00 sec)

mysql> stop group_replication; start group_replication;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (5.65 sec)
```

Check group replication status on node1.

```
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 19654e63-6769-11e6-bda8-00163e4aaba7 | grprepl2    |        3306 | ONLINE       |
| group_replication_applier | 553987cc-6768-11e6-888a-00163ed83514 | grprepl1    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
2 rows in set (0.00 sec)

mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14717610423503000:2
                         MEMBER_ID: 553987cc-6768-11e6-888a-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 2
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 553987cc-6768-11e6-888a-00163ed83514:1-2,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-3
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:3
1 row in set (0.00 sec)
```

On node3 do the same steps performed in node2.

```
mysql> reset master;
Query OK, 0 rows affected (0.05 sec)

mysql> change master to master_user='rplusr',master_password='rp1_Pwd#0' for channel 'group_replication_recovery';
Query OK, 0 rows affected, 2 warnings (0.07 sec)

mysql> set @@global.gtid_purged='553987cc-6768-11e6-888a-00163ed83514:1-2,9bd4bf53-61bf-11e6-aa38-00163e772659:1';
Query OK, 0 rows affected (0.00 sec)

mysql> stop group_replication; start group_replication;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (2.33 sec)
```

Check group replication status on node1 once more.

```
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 19654e63-6769-11e6-bda8-00163e4aaba7 | grprepl2    |        3306 | ONLINE       |
| group_replication_applier | 553987cc-6768-11e6-888a-00163ed83514 | grprepl1    |        3306 | ONLINE       |
| group_replication_applier | 579d2839-676a-11e6-b88e-00163ee2bfdb | grprepl3    |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
3 rows in set (0.00 sec)

mysql> select * from performance_schema.replication_group_member_stats\G
*************************** 1. row ***************************
                      CHANNEL_NAME: group_replication_applier
                           VIEW_ID: 14717610423503000:3
                         MEMBER_ID: 553987cc-6768-11e6-888a-00163ed83514
       COUNT_TRANSACTIONS_IN_QUEUE: 0
        COUNT_TRANSACTIONS_CHECKED: 2
          COUNT_CONFLICTS_DETECTED: 0
COUNT_TRANSACTIONS_ROWS_VALIDATING: 0
TRANSACTIONS_COMMITTED_ALL_MEMBERS: 553987cc-6768-11e6-888a-00163ed83514:1-2,
9bd4bf53-61bf-11e6-aa38-00163e772659:1-4
    LAST_CONFLICT_FREE_TRANSACTION: 9bd4bf53-61bf-11e6-aa38-00163e772659:3
1 row in set (0.00 sec)
```
