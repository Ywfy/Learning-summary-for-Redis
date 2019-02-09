# Redis基础知识
## 单进程
* redis是以单进程模型来处理客户端的请求。对读写等事件的响应 是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率
* epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本， 它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率

## 默认16个数据库，类似数组下表从零开始，初始默认使用零号库
查看redis.conf<br>

![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/Basic/db.png)<br>

## 统一密码管理，16个库都是同样密码
```
在linux下默认是关闭的,即不需要密码
```

## 重复设置键，会被覆盖
```
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> set k2 vb
OK
127.0.0.1:6379> get k2
"vb"
```

## Redis的命令不区分大小写，Redis的Key区分大小写

## Redis客户端输入命令可以使用Tab补全

## Redis索引都是从零开始
## 默认端口是6379

## 常用命令

### select：修改使用的数据库
```
127.0.0.1:6379> select 7
OK
```

### dbsize：查看当前数据库的key的数量
```
127.0.0.1:6379> dbsize
(integer) 1
```

### keys：查找当前数据库的key
```
127.0.0.1:6379> keys *
1) "k1"

127.0.0.1:6379> keys k?
1) "k1"
```
从第二个例子可以看出，keys命令支持占位符

### flushdb：清空当前库
```
127.0.0.1:6379> FLUSHDB
OK
127.0.0.1:6379> keys *
(empty list or set)
```

### Flushall：通杀全部库
```
这个就是清空所有的数据库，千万不能乱用！
```

### TIME
一个包含两个字符串的列表： 第一个字符串是当前时间(以 UNIX 时间戳格式表示，单位为秒)，而第二个字符串是当前这一秒钟已经逝去的微秒数。
```
127.0.0.1:6379> TIME
1) "1549690708"
2) "999486"
```

###  exists key的名字，判断某个key是否存在
```
127.0.0.1:6379> set k1 va
OK
127.0.0.1:6379> set k2 vb
OK
127.0.0.1:6379> set k3 vc
OK
127.0.0.1:6379> EXISTS k1
(integer) 1
127.0.0.1:6379> EXISTS k12
(integer) 0
```

###  move key db 将key移动到别的数据库
```
127.0.0.1:6379> keys *
1) "k3"
2) "k2"
3) "k1"
127.0.0.1:6379> move k3 2
(integer) 1
127.0.0.1:6379> keys k3
(empty list or set)
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> keys k3
1) "k3"
```

###  expire key 秒钟：为给定的key设置过期时间<br>ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期
```
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
127.0.0.1:6379> ttl k2
(integer) -1
127.0.0.1:6379> EXPIRE k2 10
(integer) 1
127.0.0.1:6379> ttl k2
(integer) 5
127.0.0.1:6379> get k2
"vb"
127.0.0.1:6379> ttl k2
(integer) -2
127.0.0.1:6379> get k2
(nil)
127.0.0.1:6379> keys *
1) "k1"
```

###  type key 查看key是什么类型
```
127.0.0.1:6379> type k2
string
```

### DEL key [key ...] 删除
```
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> set k4 v4
OK
127.0.0.1:6379> del k3 k4
(integer) 2
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
```

