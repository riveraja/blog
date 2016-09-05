#### Testing MySQL Group Replication

###### Recovering single node down instance.

To simulate this, I sent a kill (-9) signal to the mysqld process on node3.

Logged on node1:
```
2016-09-05T03:58:19.549408Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
```
Logged on node2:
```
2016-09-05T03:58:19.148848Z 46 [Note] Aborted connection 46 to db: 'unconnected' user: 'rpl' host: 'gr3' (failed on flush_net())
2016-09-05T03:58:19.546748Z 0 [Note] Plugin group_replication reported: '[GCS] Removing members that have failed while processing new view.'
2016-09-05T03:58:19.547477Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T03:58:19.549311Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
```
A brief moment after node3 is killed, the MEMBER_STATE for node3 will be UNREACHABLE on the other two nodes:
```
[root@sysbenchvm ~]# pssh -h gr-hosts1_2 -l root -i "mysql -e 'select * from performance_schema.replication_group_members;'"
[1] 03:56:07 [SUCCESS] gr1
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	UNREACHABLE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[2] 03:56:07 [SUCCESS] gr2
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	UNREACHABLE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
```
And then eventually will be removed from the group_members list:
```
[root@sysbenchvm ~]# pssh -h gr-hosts1_2 -l root -i "mysql -e 'select * from performance_schema.replication_group_members;'"
[1] 03:56:10 [SUCCESS] gr1
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[2] 03:56:10 [SUCCESS] gr2
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
```

Adding node3 back to the group is as easy as restarting mysqld process.

Logged on node1:
```
2016-09-05T04:01:57.148803Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
2016-09-05T04:01:59.619352Z 0 [Note] Plugin group_replication reported: 'Marking group replication view change with view_id 14730303408913635:11'
2016-09-05T04:01:59.810744Z 0 [Note] Plugin group_replication reported: 'Server 0ee43ea3-723e-11e6-846b-00163ed93f10 was declared online within the replication group'
```
Logged on node2:
```
2016-09-05T04:01:57.148800Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
2016-09-05T04:01:59.520130Z 0 [Note] Plugin group_replication reported: 'Marking group replication view change with view_id 14730303408913635:11'
2016-09-05T04:01:59.599993Z 54 [Note] Start binlog_dump to master_thread_id(54) slave_server(1003), pos(, 4)
2016-09-05T04:01:59.810712Z 0 [Note] Plugin group_replication reported: 'Server 0ee43ea3-723e-11e6-846b-00163ed93f10 was declared online within the replication group'
```
Logged on node3:
```
2016-09-05T04:01:56.448503Z mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2016-09-05T04:01:56.613368Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-09-05T04:01:56.614361Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.14-labs-gr080-log) starting as process 2643 ...
2016-09-05T04:01:56.619140Z 0 [Note] InnoDB: PUNCH HOLE support available
2016-09-05T04:01:56.619210Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2016-09-05T04:01:56.619230Z 0 [Note] InnoDB: Uses event mutexes
2016-09-05T04:01:56.619249Z 0 [Note] InnoDB: GCC builtin __sync_synchronize() is used for memory barrier
2016-09-05T04:01:56.619267Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.3
2016-09-05T04:01:56.619292Z 0 [Note] InnoDB: Using Linux native AIO
2016-09-05T04:01:56.619608Z 0 [Note] InnoDB: Number of pools: 1
2016-09-05T04:01:56.619736Z 0 [Note] InnoDB: Using CPU crc32 instructions
2016-09-05T04:01:56.621252Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2016-09-05T04:01:56.628942Z 0 [Note] InnoDB: Completed initialization of buffer pool
2016-09-05T04:01:56.630756Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2016-09-05T04:01:56.642372Z 0 [Note] InnoDB: Highest supported file format is Barracuda.
2016-09-05T04:01:56.643835Z 0 [Note] InnoDB: Log scan progressed past the checkpoint lsn 155560749
2016-09-05T04:01:56.643878Z 0 [Note] InnoDB: Doing recovery: scanned up to log sequence number 155560758
2016-09-05T04:01:56.644143Z 0 [Note] InnoDB: Doing recovery: scanned up to log sequence number 155560758
2016-09-05T04:01:56.644182Z 0 [Note] InnoDB: Database was not shutdown normally!
2016-09-05T04:01:56.644202Z 0 [Note] InnoDB: Starting crash recovery.
2016-09-05T04:01:56.654098Z 0 [Note] InnoDB: Last MySQL binlog file position 0 50085, file name gr3-bin.000002
2016-09-05T04:01:56.767041Z 0 [Note] InnoDB: Removed temporary tablespace data file: "ibtmp1"
2016-09-05T04:01:56.767190Z 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2016-09-05T04:01:56.767653Z 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2016-09-05T04:01:56.898126Z 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2016-09-05T04:01:56.900589Z 0 [Note] InnoDB: 96 redo rollback segment(s) found. 96 redo rollback segment(s) are active.
2016-09-05T04:01:56.900707Z 0 [Note] InnoDB: 32 non-redo rollback segment(s) are active.
2016-09-05T04:01:56.901798Z 0 [Note] InnoDB: Waiting for purge to start
2016-09-05T04:01:56.952133Z 0 [Note] InnoDB: 5.7.14 started; log sequence number 155560758
2016-09-05T04:01:56.952732Z 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2016-09-05T04:01:56.953053Z 0 [Note] Plugin 'FEDERATED' is disabled.
2016-09-05T04:01:56.953209Z 0 [Note] InnoDB: Buffer pool(s) load completed at 160905  0:01:56
2016-09-05T04:01:56.961097Z 0 [Note] Recovering after a crash using gr3-bin
2016-09-05T04:01:56.961181Z 0 [Note] Starting crash recovery...
2016-09-05T04:01:56.961244Z 0 [Note] Crash recovery finished.
2016-09-05T04:01:56.978522Z 0 [Warning] Failed to set up SSL because of the following SSL library error: SSL context is not usable without certificate and private key
2016-09-05T04:01:56.978938Z 0 [Note] Server hostname (bind-address): '*'; port: 3306
2016-09-05T04:01:56.979115Z 0 [Note] IPv6 is available.
2016-09-05T04:01:56.979144Z 0 [Note]   - '::' resolves to '::';
2016-09-05T04:01:56.979182Z 0 [Note] Server socket created on IP: '::'.
2016-09-05T04:01:57.008423Z 0 [Note] Relay log recovery skipped for group replication channel.
2016-09-05T04:01:57.008483Z 0 [Warning] Recovery from master pos 4 and file  for channel 'group_replication_applier'. Previous relay log pos and relay log file had been set to 4, ./gr3-relay-bin-group_replication_applier.000008 respectively.
2016-09-05T04:01:57.045563Z 0 [Note] Relay log recovery skipped for group replication channel.
2016-09-05T04:01:57.045617Z 0 [Warning] Recovery from master pos 4 and file  for channel 'group_replication_recovery'. Previous relay log pos and relay log file had been set to 4, ./gr3-relay-bin-group_replication_recovery.000001 respectively.
2016-09-05T04:01:57.058236Z 0 [Note] Event Scheduler: Loaded 0 events
2016-09-05T04:01:57.058457Z 0 [Note] /usr/sbin/mysqld: ready for connections.
Version: '5.7.14-labs-gr080-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
2016-09-05T04:01:57.078627Z 2 [Note] Plugin group_replication reported: 'Group communication SSL configuration: group_replication_ssl_mode: "DISABLED"'
2016-09-05T04:01:57.078856Z 2 [Note] Plugin group_replication reported: '[GCS] Added automatically IP ranges 10.0.3.107/24,127.0.0.1/8 to the whitelist'
2016-09-05T04:01:57.079276Z 2 [Note] Plugin group_replication reported: '[GCS] SSL was not enabled'
2016-09-05T04:01:57.079332Z 2 [Note] Plugin group_replication reported: 'Initialized group communication with configuration: group_replication_group_name: "9bd4bf53-61bf-11e6-aa38-00163e772659"; group_replication_local_address: "10.0.3.107:6606"; group_replication_group_seeds: "10.0.3.124:6606,10.0.3.131:6606"; group_replication_bootstrap_group: false; group_replication_poll_spin_loops: 0; group_replication_compression_threshold: 0; group_replication_ip_whitelist: "AUTOMATIC"'
2016-09-05T04:01:57.079870Z 3 [Note] 'CHANGE MASTER TO FOR CHANNEL 'group_replication_applier' executed'. Previous state master_host='<NULL>', master_port= 0, master_log_file='', master_log_pos= 4, master_bind=''. New state master_host='<NULL>', master_port= 0, master_log_file='', master_log_pos= 4, master_bind=''.
2016-09-05T04:01:57.123392Z 2 [Note] Plugin group_replication reported: 'Group Replication applier module successfully initialized!'
2016-09-05T04:01:57.123410Z 6 [Note] Slave SQL thread for channel 'group_replication_applier' initialized, starting replication in log 'FIRST' at position 0, relay log './gr3-relay-bin-group_replication_applier.000010' position: 4
2016-09-05T04:01:57.123642Z 2 [Note] Plugin group_replication reported: 'auto_increment_increment is set to 7'
2016-09-05T04:01:57.123675Z 2 [Note] Plugin group_replication reported: 'auto_increment_offset is set to 1003'
2016-09-05T04:01:57.123798Z 0 [Note] Plugin group_replication reported: 'state 0 action xa_init'
2016-09-05T04:01:57.140709Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.140965Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.141292Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.141474Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.141522Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.141641Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.141856Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.141938Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.142226Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.142334Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.142555Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.142641Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.142672Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.142774Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.142906Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143004Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143033Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.143124Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.143255Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143331Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143359Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.143446Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.143577Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143653Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143680Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.143767Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.143896Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143970Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.143998Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.144107Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.144249Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144325Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144354Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.144440Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.144570Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144645Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144673Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.144758Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.144884Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144965Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.144993Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.145076Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.145201Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145276Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145304Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.145386Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.145513Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145590Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145617Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.145699Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.145827Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145903Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.145932Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.146014Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.146141Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146217Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146245Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.146327Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.146454Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146553Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146581Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.146663Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.146789Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146865Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.146893Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.146975Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.147103Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147180Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147208Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.147289Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.147418Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147496Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147523Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.147605Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.147733Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147810Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.147838Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.107 6606'
2016-09-05T04:01:57.147919Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.107 6606'
2016-09-05T04:01:57.148074Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.148163Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:57.148194Z 0 [Note] Plugin group_replication reported: 'connecting to 10.0.3.124 6606'
2016-09-05T04:01:57.148300Z 0 [Note] Plugin group_replication reported: 'connected to 10.0.3.124 6606'
2016-09-05T04:01:57.148492Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T04:01:58.516836Z 0 [Note] Plugin group_replication reported: 'state 4115 action xa_snapshot'
2016-09-05T04:01:58.517830Z 0 [Note] Plugin group_replication reported: '1473048118.517803 /export/home/pb2/build/sb_0-19878585-1469704387.16/rpm/BUILD/mysql-5.7.14-labs-gr080/mysql-5.7.14-labs-gr080/rapid/plugin/group_replication/libmysqlgcs/src/bindings/xcom/xcom/xcom_base.c:3544 new state x_recover'
2016-09-05T04:01:58.518003Z 0 [Note] Plugin group_replication reported: 'state 4132 action xa_complete'
2016-09-05T04:01:58.518527Z 0 [Note] Plugin group_replication reported: '1473048118.518498 /export/home/pb2/build/sb_0-19878585-1469704387.16/rpm/BUILD/mysql-5.7.14-labs-gr080/mysql-5.7.14-labs-gr080/rapid/plugin/group_replication/libmysqlgcs/src/bindings/xcom/xcom/xcom_base.c:3545 new state x_run'
2016-09-05T04:01:59.520023Z 0 [Note] Plugin group_replication reported: 'Starting group replication recovery with view_id 14730303408913635:11'
2016-09-05T04:01:59.521221Z 8 [Note] Plugin group_replication reported: 'Establishing group recovery connection with a possible donor. Attempt 1/10'
2016-09-05T04:01:59.526967Z 9 [Note] Access denied for user 'UNKNOWN_MYSQL_USER'@'localhost' (using password: NO)
2016-09-05T04:01:59.556812Z 8 [Note] 'CHANGE MASTER TO FOR CHANNEL 'group_replication_recovery' executed'. Previous state master_host='<NULL>', master_port= 0, master_log_file='', master_log_pos= 4, master_bind=''. New state master_host='gr2', master_port= 3306, master_log_file='', master_log_pos= 4, master_bind=''.
2016-09-05T04:01:59.594480Z 8 [Note] Plugin group_replication reported: 'Establishing connection to a group replication recovery donor 64dcef67-723b-11e6-bc5b-00163ecd120f at gr2 port: 3306.'
2016-09-05T04:01:59.595053Z 10 [Warning] Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information.
2016-09-05T04:01:59.596707Z 10 [Note] Slave I/O thread for channel 'group_replication_recovery': connected to master 'rpl@gr2:3306',replication started in log 'FIRST' at position 4
2016-09-05T04:01:59.598407Z 11 [Note] Slave SQL thread for channel 'group_replication_recovery' initialized, starting replication in log 'FIRST' at position 0, relay log './gr3-relay-bin-group_replication_recovery.000001' position: 4
2016-09-05T04:01:59.669987Z 8 [Note] Plugin group_replication reported: 'Terminating existing group replication donor connection and purging the corresponding logs.'
2016-09-05T04:01:59.673153Z 11 [Note] Slave SQL thread for channel 'group_replication_recovery' exiting, replication stopped in log 'gr1-bin.000007' at position 50813
2016-09-05T04:01:59.679943Z 10 [Note] Slave I/O thread killed while reading event for channel 'group_replication_recovery'
2016-09-05T04:01:59.680068Z 10 [Note] Slave I/O thread exiting for channel 'group_replication_recovery', read up to log 'gr1-bin.000007', position 50813
2016-09-05T04:01:59.748947Z 8 [Note] 'CHANGE MASTER TO FOR CHANNEL 'group_replication_recovery' executed'. Previous state master_host='gr2', master_port= 3306, master_log_file='', master_log_pos= 4, master_bind=''. New state master_host='<NULL>', master_port= 0, master_log_file='', master_log_pos= 4, master_bind=''.
2016-09-05T04:01:59.810750Z 0 [Note] Plugin group_replication reported: 'This server was declared online within the replication group'
```
And since each node uses `group_replication_start_on_boot=ON` by default it will try to rejoin the rest of the group as seen above.
```
[root@sysbenchvm ~]# pssh -h gr-hosts-all -l root -i "mysql -e 'select * from performance_schema.replication_group_members;'"
[1] 03:57:27 [SUCCESS] gr3
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	ONLINE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[2] 03:57:27 [SUCCESS] gr1
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	ONLINE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[3] 03:57:27 [SUCCESS] gr2
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	ONLINE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
```

However, a network connection problem on a particular node is not as easy to recover that a killed node.

To simulate I used iptables to block then eventually unblock port 6606, this port is used for GCS (XCOM) group communication.

Blocked node3's port 6606 for 4 seconds (blocking for < 3s does not affect the node communication)

Logged on node1:
```
2016-09-05T05:04:55.790838Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
```
Logged on node2:
```
2016-09-05T05:04:55.788965Z 0 [Note] Plugin group_replication reported: '[GCS] Removing members that have failed while processing new view.'
2016-09-05T05:04:55.789510Z 0 [Note] Plugin group_replication reported: 'cli_err 0'
2016-09-05T05:04:55.790822Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
```
Logged on node3:
```
2016-09-05T05:04:57.478388Z 0 [Note] Plugin group_replication reported: 'getstart group_id 96aba29f'
2016-09-05T05:05:00.479018Z 0 [Note] Plugin group_replication reported: 'state 4185 action xa_terminate'
2016-09-05T05:05:00.479636Z 0 [Note] Plugin group_replication reported: '1473051900.479617 /export/home/pb2/build/sb_0-19878585-1469704387.16/rpm/BUILD/mysql-5.7.14-labs-gr080/mysql-5.7.14-labs-gr080/rapid/plugin/group_replication/libmysqlgcs/src/bindings/xcom/xcom/xcom_base.c:2179 new state x_start'
2016-09-05T05:05:00.479693Z 0 [Note] Plugin group_replication reported: 'state 4115 action xa_exit'
2016-09-05T05:05:00.479824Z 0 [Note] Plugin group_replication reported: 'Exiting xcom thread'
2016-09-05T05:05:00.479913Z 0 [Note] Plugin group_replication reported: '1473051900.479903 /export/home/pb2/build/sb_0-19878585-1469704387.16/rpm/BUILD/mysql-5.7.14-labs-gr080/mysql-5.7.14-labs-gr080/rapid/plugin/group_replication/libmysqlgcs/src/bindings/xcom/xcom/xcom_base.c:2180 new state x_start'
```
The way to recover node3 back to the group cluster is to bootstrap node1 or node2, but first stop group_replication on all nodes.
```
mysql> stop group_replication;
Query OK, 0 rows affected (8.35 sec)
```

Then bootstrap one node:
```
mysql> set global group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> start group_replication;
Query OK, 0 rows affected (1.07 sec)
```

Let the others join one at a time:
```
mysql> start group_replication;
Query OK, 0 rows affected (7.11 sec)
```

This tells us that there are 3 nodes in the cluster:
```
2016-09-05T05:16:46.388150Z 0 [Note] Plugin group_replication reported: 'Starting group replication recovery with view_id 14730524021915363:3'
```

And this tells us that the node has successfully joined the group:
```
2016-09-05T05:16:46.673911Z 0 [Note] Plugin group_replication reported: 'This server was declared online within the replication group'
```

The group_replication_members result on all nodes when node3 was split from the cluster:
```
[root@sysbenchvm ~]# pssh -h gr-hosts-all -l root -i "mysql -e 'select * from performance_schema.replication_group_members;'"
[1] 05:05:10 [SUCCESS] gr2
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[2] 05:05:10 [SUCCESS] gr1
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	ONLINE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	ONLINE
[3] 05:05:10 [SUCCESS] gr3
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE
group_replication_applier	0ee43ea3-723e-11e6-846b-00163ed93f10	gr3	3306	ONLINE
group_replication_applier	144133ce-721d-11e6-a83f-00163e69abdd	gr1	3306	UNREACHABLE
group_replication_applier	64dcef67-723b-11e6-bc5b-00163ecd120f	gr2	3306	UNREACHABLE
```
