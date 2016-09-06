#### Testing MySQL Group Replication on low latency network

Setup:
- 3 node MGR Cluster

Sysbench:
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua \
         --oltp_tables_count=5 --oltp_table_size=1000000 \
         --num-threads=50 --mysql-host=10.0.3.124 \
         --mysql-user=sbuser --oltp-read-only=off \
         --max-time=60 \
         --max-requests=0 \
         --report-interval=10 \
         --rand-type=uniform \
         --rand-init=on \
         run

tc qdisc add dev eth0 root netem delay 5ms
tc qdisc replace dev eth0 root netem delay 10ms


