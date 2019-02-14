# RDB快照(snapshotting)

## 简介
  RDB是Redis用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。
  
## 触发方式
RDB有两种触发方式，分别是自动触发和手动触发。
### 自动触发
在redis.conf配置文件中的 SNAPSHOTTING 下
```
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```
* 文件已经说得很清楚，通过该配置项可以设置RDB的触发条件，多少时间内若有至少几次修改则保存
* 若不需要持久化功能，则直接将save全部注释掉就行了
<br>

顺便简要说下后面的相关项
* stop-writes-on-bgsave-error yes:当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。默认值为yes，这可以提醒用户持久化操作失败了
* rdbcompression yes:对于存储到磁盘中的快照，是否进行压缩,默认值为yes。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大
* rdbchecksum yes：在存储快照后，是否使用CRC64算法来进行数据校验，默认值为yes，追求性能可以考虑关闭
* dbfilename dump.rdb：快照文件名，默认为dump.rdb
* dir ./ :设置快照文件的存放路径,默认是和当前配置文件保存在同一目录

### 手动触发
手动触发RDB的命令有两个:save和bgsave
#### save
* 该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止
* 显然，当内存中的数据非常大的时候，这是致命而不可接受的
#### bgsave
* 执行该命令时，Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求。Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短
* 可以使用lastsave命令获取最后一次成功执行快照的时间
* 基本上 Redis 内部所有的RDB操作都是采用 bgsave 命令

## 恢复
将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可，redis就会自动加载文件数据至内存了。Redis 服务器在载入 RDB 文件期间，会一直处于阻塞状态，直到载入工作完成为止。<br>
查询redis的安装目录可以使用config命令
```
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/redis/bin"
```

## 优劣
### 优势
* 适合大规模的数据恢复
* 对数据完整性和一致性要求不高
### 劣势
* 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改
* fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

## 动态停止RDB方法
```
redis-cli config set save ""
```

