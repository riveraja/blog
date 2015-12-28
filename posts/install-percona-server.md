#### Using Percona Server Linux Generic tarballs on CentOS 6

--- Download the binary tarball from [Percona Downloads page](https://www.percona.com/downloads/Percona-Server-5.6/).

Make sure to download the correct binary tarball depending on your systems OpenSSL version.

```
openssl version
OpenSSL 1.0.1e-fips 11 Feb 2013
```
This means I have to download Percona-Server-5.6.27-rel76.0-Linux.x86_64.ssl101.tar.gz from the site. In almost all cases you'd have to download this binary.

--- Uncompress the file with tar

```# tar -zxf Percona-Server-5.6.27-rel76.0-Linux.x86_64.ssl101.tar.gz```

--- Install libaio and numactl, these are needed.

```# yum install libaio numactl```

--- Create the data directory, default is /var/lib/mysql but you can make your own

```# mkdir /srv/data -p```

--- Move uncompressed Percona Server binary to /opt/mysql

```# mv Percona-Server-5.6.27-rel76.0-Linux.x86_64.ssl101 /opt/mysql```

--- Create a mysql system user and group user, then change ownership of the datadir to mysql

```
# groupadd mysql
# useradd -r -g mysql mysql
# chown -R mysql:mysql /srv/data
```

--- Initialize the data directory using mysql_install_db

```/opt/mysql/scripts/mysql_install_db --user=mysql --datadir=/srv/data```

--- Once everything went through without errors, create a my.cnf file.

```
[mysqld]
datadir=/srv/data
user=mysql
pid-file=/srv/data/mysqld.pid
socket=/srv/data/mysql.sock
innodb-file-per-table
```

--- Start MySQL

```# /opt/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf &```

--- Check error log and you should see something like this if everything is OK

```
151227 23:56:33 mysqld_safe Starting mysqld daemon with databases from /srv/data
2015-12-27 23:56:33 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2015-12-27 23:56:33 0 [Note] /opt/mysql/bin/mysqld (mysqld 5.6.27-76.0) starting as process 1100 ...
2015-12-27 23:56:33 1100 [Note] Plugin 'FEDERATED' is disabled.
2015-12-27 23:56:33 1100 [Note] InnoDB: Using atomics to ref count buffer pool pages
2015-12-27 23:56:33 1100 [Note] InnoDB: The InnoDB memory heap is disabled
2015-12-27 23:56:33 1100 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2015-12-27 23:56:33 1100 [Note] InnoDB: Memory barrier is not used
2015-12-27 23:56:33 1100 [Note] InnoDB: Compressed tables use zlib 1.2.3
2015-12-27 23:56:33 1100 [Note] InnoDB: Using Linux native AIO
2015-12-27 23:56:33 1100 [Note] InnoDB: Using CPU crc32 instructions
2015-12-27 23:56:33 1100 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2015-12-27 23:56:33 1100 [Note] InnoDB: Completed initialization of buffer pool
2015-12-27 23:56:33 1100 [Note] InnoDB: Highest supported file format is Barracuda.
2015-12-27 23:56:33 1100 [Note] InnoDB: 128 rollback segment(s) are active.
2015-12-27 23:56:33 1100 [Note] InnoDB: Waiting for purge to start
2015-12-27 23:56:33 1100 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.27-rel76.0 started; log sequence number 1626017
2015-12-27 23:56:33 1100 [Note] RSA private key file not found: /srv/data//private_key.pem. Some authentication plugins will not work.
2015-12-27 23:56:33 1100 [Note] RSA public key file not found: /srv/data//public_key.pem. Some authentication plugins will not work.
2015-12-27 23:56:33 1100 [Note] Server hostname (bind-address): '*'; port: 3306
2015-12-27 23:56:33 1100 [Note] IPv6 is available.
2015-12-27 23:56:33 1100 [Note]   - '::' resolves to '::';
2015-12-27 23:56:33 1100 [Note] Server socket created on IP: '::'.
2015-12-27 23:56:33 1100 [Note] Event Scheduler: Loaded 0 events
2015-12-27 23:56:33 1100 [Note] /opt/mysql/bin/mysqld: ready for connections.
Version: '5.6.27-76.0'  socket: '/srv/data/mysql.sock'  port: 3306  Percona Server (GPL), Release 76.0, Revision 5498987
```

##### --- Add one more instance on the same machine

--- Create another data directory and give ownership to mysql system user

```
# mkdir /srv/data2/
# chown -R mysql:mysql /srv/data2
```

--- Initialize the data directory as follows

`# /opt/mysql/scripts/mysql_install_db --user=mysql --datadir=/srv/data2`

--- Create a new my.cnf file for the new data directory (eg /etc/my2.cnf)

```
[mysqld]
datadir=/srv/data2
user=mysql
pid-file=/srv/data2/mysqld.pid
socket=/srv/data2/mysql.sock
innodb-file-per-table
port=3307
```
Make sure to specify a different port so it won't conflict with the first instance which uses port 3306 as default port.

--- Start MySQL instance

`# /opt/mysql/bin/mysqld_safe --defaults-file=/etc/my2.cnf &`
