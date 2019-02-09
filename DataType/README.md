# Redis五大数据类型

### List(列表)
* Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
* 它的底层实际是个链表

### Set(集合)
* Redis的Set是string类型的无序集合。它是通过HashTable实现的

### Zset(sorted set:有序集合)
* Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。
* redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。
<br>

<strong>参考博客:http://www.runoob.com/redis/redis-tutorial.html</strong><br>
<strong>详细了解推荐查看网站:http://redisdoc.com/</strong>
<br><br>

## Redis字符串(String)
* string是redis最基本的类型，一个key对应一个value
* string类型是二进制安全的。意思是redis的string可以包含任何数据,比如jpg图片或者序列化的对象 
* string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M
### SET
* 将字符串值value关联到key
* key已关联则覆盖，无视类型
* 原本key带有生存时间TTL，那么TTL被清除
```
127.0.0.1:6379> set k3 v3
OK
```

### GET
* 返回key关联的字符串值
* key不存在则返回nil
* key存储的不是字符串，返回错误，因为GET只用于处理字符串
```
127.0.0.1:6379> get k3
(nil)
127.0.0.1:6379> get k2
"vb"
```

### MSET
* 同时设置一个或多个key-value键值对
* 某个给定key已经存在，那么MSET新值会覆盖旧值
* 如果上面的覆盖不是希望的，那么使用MSETNX命令，所有key都不存在才会进行关联
* MSET是一个原子性操作，所有key都会在同一时间被设置，不会存在有些更新有些没更新的情况
```
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
127.0.0.1:6379> MSET k3 v3 k4 v4
OK
127.0.0.1:6379> keys *
1) "k4"
2) "k3"
3) "k2"
4) "k1"
```

### MGET
* 返回一个或多个给定key对应的value
* 某个key不存在那么这个key返回nil
```
127.0.0.1:6379> MGET k1 k2 k7
1) "va"
2) "vb"
3) (nil)
```

### SETEX
* 将value关联到key
* 设置key生存时间为seconds，单位为秒
* 如果key对应的value已经存在，则覆盖旧值
* SET也可以设置失效时间，但是不同在于SETEX是一个原子操作，即关联值与设置生存时间同一时间完成
```
127.0.0.1:6379> SETEX k6 10 v6
OK
127.0.0.1:6379> ttl k6
(integer) 5
127.0.0.1:6379> get k6
"v6"
127.0.0.1:6379> ttl k6
(integer) -2
127.0.0.1:6379> get k6
(nil)
```
顺带说一下Redis的失效策略:<br>
(1)被动触发：GET的时候检查一下key是否失效<br>
(2)主动触发：后台每1秒跑10次定时任务(通过redis.conf的hz参数配置，默认为10)，随机选择100个设置了过期时间的key，对过期的key进行失效

### SETNX
* 将key的值设置为value，当且仅当key不存在
* 若给定的key已经存在，SETNX不做任何动作
```
127.0.0.1:6379> SETNX k1 v1
(integer) 0
127.0.0.1:6379> SETNX k6 v6
(integer) 1
```

### INCR/DECR
* key中存储的数字值+1/-1，返回增减之后的值
* key不存在，那么key的值被初始化为0再执行INCR/DECR操作
* 如果值包含错误类型或者字符串不能被表示为数字，那么返回错误
* 值限制在64位有符号数字表示之内，即-9223372036854775808~9223372036854775807
```
127.0.0.1:6379> set k5 5
OK
127.0.0.1:6379> INCR k5
(integer) 6
127.0.0.1:6379> DECR k5
(integer) 5
127.0.0.1:6379> INCR k10
(integer) 1
127.0.0.1:6379> DECR k12
(integer) -1
```

### INCRBY/DECRBY
* 将key所存储的值加上增量/减去减量，返回操作之后的值
* 其余同INCR/DECR
```
127.0.0.1:6379> INCRBY k10 6
(integer) 7
127.0.0.1:6379> DECRBY k12 6
(integer) -7
```

## Hash(哈希，类似java里的Map)
* Redis hash 是一个键值对集合
* Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象
* 类似Java里面的Map<String,Object>
### HSET 
* 将哈希表key中的域field的值设为value
* key不存在，一个新的Hash表被创建
* field已经存在，旧的值被覆盖
* 如果字段是哈希表中的一个新建字段，并且设置成功，返回1；如果是已存在且旧值被覆盖，则返回0
```
语法：HSET KEY_NAME FIELD VALUE 

示例：
127.0.0.1:6379> HSET myhash field1 "yoo"
(integer) 1
127.0.0.1:6379> HGET myhash field1
"yoo"
127.0.0.1:6379> HSET myhash field1 "hoo~"
(integer) 0
```

### HGET
* 返回哈希表key中给定域field的值,若给定的字段或key不存在则返回nil
```
语法:HGET key_name field_name

示例：
127.0.0.1:6379> HGET myhash field1
"yoo"
```

### HDEL
* 删除哈希表key中的一个或多个指定域
* 不存在的域将被忽略
```
语法: HDEL key_name field1 field2... fieldN

示例：
127.0.0.1:6379> HSET myhash field2 "yooho~"
(integer) 1
127.0.0.1:6379> HDEL myhash field2
(integer) 1
127.0.0.1:6379> HDEL myhash field2
(integer) 0
```

### HEXISTS
* 查看哈希表key中，给定域field是否存在，存在返回1，不存在返回0
```
语法：HEXISTS key_name field_name

示例：
127.0.0.1:6379> HEXISTS myhash field1
(integer) 1
127.0.0.1:6379> HEXISTS myhash field2
(integer) 0
```
