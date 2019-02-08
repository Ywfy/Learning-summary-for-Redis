# Redis基础知识
## 单进程
* redis是以单进程模型来处理客户端的请求。对读写等事件的响应 是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率
* epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本， 它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率

## 默认16个数据库，类似数组下表从零开始，初始默认使用零号库
查看redis.conf<br>

![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/Basic/db.png)<br>



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

## 统一密码管理，16个库都是同样密码
```
在linux下默认是关闭的,即不需要密码
```

## Redis索引都是从零开始
## 默认端口是6379