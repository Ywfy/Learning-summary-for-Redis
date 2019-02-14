# AOF(Append Only File)

## 简介
Redis的持久化方式之一RDB是通过保存数据库中的键值对来记录数据库的状态。而另一种持久化方式 AOF 则是通过保存Redis服务器所执行的写命令来记录数据库状态。
```
127.0.0.1:6379> set news 666
OK
127.0.0.1:6379> SADD str2 777
(integer) 1
127.0.0.1:6379> LPUSH list3 888
(integer) 1
```
对于上面的例子，最后在保存的时候：
* RDB是将news,str2,list3保存到RDB文件中
* AOF则是将执行的 set,sadd,lpush 三个命令保存到AOF文件中

## AOF配置
在redis.conf配置文件的APPEND ONLY MODE下进行配置
```
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

```
* appendonly no :默认值为no，也就是说redis默认使用的是rdb方式持久化，如果想要开启AOF持久化方式，需要将appendonly修改为 yes
* appendfilename "appendonly.aof" :AOF文件名，默认为"appendonly.aof"
* appendfsync everysec :AOF持久化策略的配置,共有三个可选
    * 不同步：appendfsync no   从不同步
    * 每修改同步：appendfsync always   同步持久化，每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好
    * 每秒同步：appendfsync everysec   出产默认推荐，异步操作，每秒记录。如果一秒内宕机，有数据丢失
* no-appendfsync-on-rewrite no:重写时是否可以运用Appendfsync，用默认no即可，确保数据完整性
* auto-aof-rewrite-percentage 100 ：默认值为100。aof自动重写配置，当目前aof文件大小达到上一次重写的aof文件大小的百分之多少进行重写
* auto-aof-rewrite-min-size 64mb ：设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写



## 启动AOF
* 修改默认的appendonly no，改为yes
* 将有数据的aof文件复制一份保存到对应目录(config get dir)
* 启动redis，此时会自动进行重新加载<br>
注：若AOF文件被写坏，可以通过以下命令修复，然后再重新启动redis
```
redis-check-aof --fix appendonly.aof
```
补充：
* 哪怕启动了AOF，RDB备份仍然会发挥作用，dump.rdb文件还是会存在，此时启动redis时，会读取AOF备份文件恢复而不理会dump.rdb
* 实际上，redis-check-aof --fix就是通过dump.rdb来修复aof备份文件

## AOF重写
* 由于AOF持久化是Redis不断将写命令记录到AOF文件中，随着Redis不断的进行，文件会越来越大。为避免出现此种情况，Redis新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集
* 可是使用命令bgrewriteaof来手动触发
```
127.0.0.1:6379> LPUSH list5 111
(integer) 1
127.0.0.1:6379> LPUSH list5 222
(integer) 2
127.0.0.1:6379> LPUSH list5 333
(integer) 3
127.0.0.1:6379> LPUSH list5 444
(integer) 4
```
对于上述例子，若不重写，那么AOF文件将保存四条lpush命令，若重写，则只保存一条如下命令：
```
LPUSH list5 111 222 333 444
```
* 原理：AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)， 遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件， 而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件
* 触发机制：Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发
* AOF 重写缓冲区：
   * 子进程在进行 AOF 重写期间，服务器进程依然在处理其它命令，这新的命令有可能也对数据库进行了修改操作，使得当前数据库状态和重写后的 AOF 文件状态不一致。
　 * 为了解决这个数据状态不一致的问题，Redis 服务器设置了一个 AOF 重写缓冲区，这个缓冲区是在创建子进程后开始使用，当Redis服务器执行一个写命令之后，就会将这个写命令也发送到 AOF 重写缓冲区。当子进程完成 AOF 重写之后，就会给父进程发送一个信号，父进程接收此信号后，就会调用函数将 AOF 重写缓冲区的内容都写到新的 AOF 文件中。

## AOF优劣
### 优势
* AOF 持久化的方法提供了多种的同步频率，即使使用默认的同步频率每秒同步一次，Redis 最多也就丢失 1 秒的数据而已
* AOF 文件的格式可读性较强，这也为使用者提供了更灵活的处理方式。例如，如果我们不小心错用了 FLUSHALL 命令，在重写还没进行时，我们可以手工将最后的 FLUSHALL命令去掉，然后再使用AOF来恢复数据
### 劣势
* 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
* aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同

## 使用性能建议
因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。


如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。


如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构

