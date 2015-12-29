#### Setting up MySQL 5.6 replication with one Master and two Slaves (GTID and non-GTID)

*This should work on popular Linux Operating Systems like Ubuntu or CentOS.*

-------

**Download and install MySQL or Percona Server 5.6 binaries or install the APT/YUM repositories.**

Download and install binaries following instructions [here](http://dev.mysql.com/doc/refman/5.6/en/linux-installation.html) or [here](https://www.percona.com/doc/percona-server/5.6/installation.html).


**Setup Master for replication.**

Add the following lines on my.cnf under [mysqld]

```
[mysqld]
server-id=1
log-bin=mysql-bin
```

For GTID-based replication, add the following lines as well

```
gtid-mode=ON
enforce-gtid-consistency
```

Start MySQL service on Master and create a replication user

```
GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'%' IDENTIFIED BY 'replpass'; 
```

**Setup Slave for replication**

Add the same options on slave's my.cnf under [mysqld] but change server-id

```
[mysqld]
server-id=2
log-bin=mysql-bin
```

For GTID-based replication, add the same options as above.

++++ UNFINISHED ++++
