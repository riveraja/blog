MySQL Group Replication

#### Installation

Download package installer from: http://labs.mysql.com/

Test environment:
OS version: CentOS 7 64bit
MySQL version package: 

```
wget http://downloads.mysql.com/snapshots/pb/mysql-group-replication-0.8.0-labs/mysql-5.7.14-labs-gr080-el7-x86_64.rpm.tar.gz

tar xf mysql-5.7.14-labs-gr080-el7-x86_64.rpm.tar.gz

yum localinstall mysql-community-libs-5.7.14-1.labs_gr080.el7.x86_64.rpm mysql-community-client-5.7.14-1.labs_gr080.el7.x86_64.rpm mysql-community-common-5.7.14-1.labs_gr080.el7.x86_64.rpm mysql-community-server-5.7.14-1.labs_gr080.el7.x86_64.rpm mysql-community-devel-5.7.14-1.labs_gr080.el7.x86_64.rpm mysql-community-libs-compat-5.7.14-1.labs_gr080.el7.x86_64.rpm
```

Contents of /etc/my.cnf
```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
server-id=3
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# replication and binlog related options
binlog-row-image = MINIMAL
binlog-rows-query-log-events = ON
log-bin-trust-function-creators = TRUE
expire-logs-days = 7
max-binlog-size = 1G
relay-log-recovery = ON
slave-parallel-type = LOGICAL_CLOCK
slave-preserve-commit-order = ON
slave-rows-search-algorithms = 'INDEX_SCAN,HASH_SCAN'
sync-master-info = 1000
sync-relay-log = 1000

# group replication pre-requisites & recommendations
log-bin=grprepl3-bin
binlog-format = ROW
gtid-mode = ON
enforce-gtid-consistency = ON
log-slave-updates = ON
master-info-repository = TABLE
relay-log-info-repository = TABLE
binlog-checksum = NONE
slave-parallel-workers = 0
# prevent use of non-transactional storage engines
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE"
  
# group replication specific options
plugin-load = group_replication.so
group_replication = FORCE_PLUS_PERMANENT
transaction-write-set-extraction = XXHASH64
group_replication_start_on_boot = ON
group_replication_bootstrap_group = OFF
group_replication_group_name = 9bd4bf53-61bf-11e6-aa38-00163e772659
group_replication_local_address = '10.0.3.214:6606'
group_replication_group_seeds = '10.0.3.205:6606,10.0.3.61:6606'
```
