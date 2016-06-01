#### Setting up Multi-source replication on existing MySQL master servers with GTID

Environment:
- two servers running mysql 5.6 configured with GTID
- one server installed with mysql 5.7 configured with GTID

```
MySQL Config:
gtid-mode=ON
enforce-gtid-consistency
log-bin=mysql-bin
server-id={}
log-slave-updates
[master-info-repository=TABLE]
[relay-log-info-repositoty=TABLE]
```

