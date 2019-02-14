# 主从复制

## 什么是主从复制
主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主

## 用途
读写分离，容灾恢复。<br>

## 示例：
我们通过在一台虚拟机上启用三个不同的端口(6379，6380，6381)来模拟三台服务器<br>
### 建立redis6379.conf,redis6380.conf,redis6381.conf，设为守护进程，进行端口号，pidfile文件名，logfile文件名,Dump.rdb名字的更改
* redis6379.conf
```
daemonize yes

pidfile /var/run/redis6379.pid

port 6379

logfile "6379.log"

dbfilename dump6379.rdb
```
* redis6380.conf
```
daemonize yes

pidfile /var/run/redis6380.pid

port 6380

logfile "6380.log"

dbfilename dump6380.rdb
```
* redis6381.conf
```
daemonize yes

pidfile /var/run/redis6381.pid

port 6381

logfile "6381.log"

dbfilename dump6381.rdb
```

### 启动Redis，在三个终端启动Redis
```
cd /usr/local/redis/bin
./redis-server /myredis/redis6379.conf
./redis-cli -p 6379
```
其他两个就不赘述了

### 查看端口号信息
```
[jw@localhost bin]$ ps -ef|grep redis
root      62279      1  0 17:34 ?        00:00:00 ./redis-server *:6380
root      62862  37044  0 17:34 pts/1    00:00:00 ./redis-cli -p 6380
root      63344      1  0 17:35 ?        00:00:00 ./redis-server *:6381
root      63483 115289  0 17:35 pts/2    00:00:00 ./redis-cli -p 6381
root      74974      1  0 17:46 ?        00:00:00 ./redis-server *:6379
root      75099  44303  0 17:46 pts/0    00:00:00 ./redis-cli -p 6379
jw        76943  42135  0 17:48 pts/3    00:00:00 grep --color=auto redis
```

### 三台机子起始都是主机身份
```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

127.0.0.1:6381> info replication
# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

### 架构模式
* 一主两从
```
127.0.0.1:6379> set k1 1
OK
127.0.0.1:6379> set k2 2
OK
127.0.0.1:6379> set k3 3
OK
127.0.0.1:6379> get k3
"3"

127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK

127.0.0.1:6381> SLAVEOF 127.0.0.1 6379
OK

127.0.0.1:6379> set str1 yohoo~~~~
OK

127.0.0.1:6380> get str1
"yohoo~~~~"
127.0.0.1:6380> get k2
"2"

127.0.0.1:6381> get str1
"yohoo~~~~"
127.0.0.1:6381> get k2
"2"
```
查看主从信息
```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=846,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=846,lag=1
master_repl_offset:846
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:845


127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:720
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0


127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:902
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```
只有主机能写，从机只能读
```
127.0.0.1:6379> set k5 5
OK

127.0.0.1:6380> set k5 55
(error) READONLY You can't write against a read only slave.

127.0.0.1:6381> set k5 555
(error) READONLY You can't write against a read only slave.
```
当主机宕机后，从机会原地待命,当主机重新连上后，一切照旧
```
主机宕机
127.0.0.1:6379> shutdown
not connected> exit


127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1784
master_link_down_since_seconds:13
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1784
master_link_down_since_seconds:22
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0


当然数据是还在的
127.0.0.1:6380> keys *
1) "k3"
2) "str1"
3) "k5"
4) "k1"
5) "k2"

主机重新连接
[root@localhost bin]# ./redis-server /myredis/redis6379.conf 
[root@localhost bin]# ./redis-cli -p 6379
127.0.0.1:6379> set k7 7
OK

127.0.0.1:6380> get k7
"7"

127.0.0.1:6381> get k7
"7"
```
从机宕机，即每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件
```
127.0.0.1:6380> shutdown
not connected> exit

127.0.0.1:6379> set k8 8
OK
127.0.0.1:6379> get k8
"8"

[root@localhost bin]# ./redis-server /myredis/redis6380.conf 
[root@localhost bin]# ./redis-cli -p 6380
127.0.0.1:6380> get k8
(nil)

当然此时6381还是正常的
127.0.0.1:6381> get k8
"8"

6380重新连接master
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6380> get k8
"8"
```
* 薪火相传
    * 去中心化，一个接一个,上一个Slave可以是下一个slave的Master，可以有效减轻master的写压力
    * 中途变更转向:会清除之前的数据，重新建立拷贝最新的
```
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380
OK

127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=2012,lag=1
master_repl_offset:2012
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:2011
127.0.0.1:6379> set k9 9
OK

127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:2077
slave_priority:100
slave_read_only:1
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=164,lag=1
master_repl_offset:164
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:163
127.0.0.1:6380> get k9
"9"

127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:192
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6381> get k9
"9"
```

* 反客为主
    * 在主机宕机后，我们希望重新设立一个主机
    * SLAVEOF no one 使当前数据库停止与其他数据库的同步，转成主数据库
```
127.0.0.1:6380> SLAVEOF no one
OK
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=1004,lag=1
master_repl_offset:1004
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:1003

此时6381还是作为6380的从机。而6379掉线了，重连后会作为单独的体系，和6380、6381没有关联了。
```

### 复制原理
* slave启动成功连接到master后会发送一个sync命令
* Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
* 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
* 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
* 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

### 现在最常用的模式：哨兵模式(sentinel)
反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

示例:
将架构模式改回一主二从模式，6379为主，6380,6381为从<br>
自定义的/myredis目录下新建sentinel.conf文件，名字绝不能错<br>
修改sentinel.conf
```
 # sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
 
 sentinel monitor host6379 127.0.0.1 6379 1
```
启动哨兵
```
cd /usr/local/redis/bin
./redis-sentinel /myredis/sentinel.conf 
```

6379主机宕机
```
127.0.0.1:6379> shutdown
not connected> exit
```

等一会儿后，哨兵终端页面出现变化
```
39941:X 14 Feb 19:24:41.519 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6379
39941:X 14 Feb 19:24:41.593 # +failover-end master host6379 127.0.0.1 6379
39941:X 14 Feb 19:24:41.593 # +switch-master host6379 127.0.0.1 6379 127.0.0.1 6380
39941:X 14 Feb 19:24:41.593 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ host6379 127.0.0.1 6380
39941:X 14 Feb 19:24:41.593 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ host6379 127.0.0.1 6380
39941:X 14 Feb 19:25:11.672 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ host6379 127.0.0.1 6380
```

重新连接6379，查看三台主机的主从信息
```
[root@localhost bin]# ./redis-server /myredis/redis6379.conf 
[root@localhost bin]# ./redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:11590
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=12136,lag=0
slave1:ip=127.0.0.1,port=6379,state=online,offset=12136,lag=0
master_repl_offset:12136
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:12135

127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:12402
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

显然6380被投票成为了新的主机，而6379和6381成为6380的从机
```
一组sentinel能同时监控多个Master

## 缺点
由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙时，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重





