# Redis五大数据类型


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

### HGETALL
* 返回哈希表key中，所有的域和值
* 在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍
```
语法：HGETALL KEY_NAME

示例：
127.0.0.1:6379> HGETALL myhash
1) "field1"
2) "hoo~"
127.0.0.1:6379> HGET myhash field1
"hoo~"
```

### HINCRBY
* 为哈希表key中的域field加上增量increment
* 增量也可以为负数，相当于对指定字段进行减法操作
* 如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令
* 如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 
* 对一个储存字符串值的字段执行 HINCRBY 命令将造成一个错误
* 本操作的值被限制在 64 位(bit)有符号数字表示之内
```
语法：HINCRBY KEY_NAME FIELD_NAME INCR_BY_NUMBER 

示例：
127.0.0.1:6379> HSET myhash field2 5
(integer) 1
127.0.0.1:6379> HINCRBY myhash field2 5
(integer) 10
127.0.0.1:6379> HINCRBY myhash field -3
(integer) -3
127.0.0.1:6379> HINCRBY myhash field2 -3
(integer) 7
127.0.0.1:6379> HGETALL myhash
1) "field1"
2) "hoo~"
3) "field2"
4) "7"
5) "field"
6) "-3"
127.0.0.1:6379> HINCRBY myhash field1 -2
(error) ERR hash value is not an integer
```

### HKEYS
* 返回哈希表key中的所有域
```
语法：HKEYS key 

示例：
127.0.0.1:6379> HKEYS myhash
1) "field1"
2) "field2"
3) "field"
```

### HLEN
* 返回哈希表key中域的数量
```
语法：HLEN KEY_NAME 

示例：
127.0.0.1:6379> HLEN myhash
(integer) 3
```

### HMGET
* 返回哈希表key中，一个或多个给定域的值
* 如果给定域不存在于哈希表，那么返回一个nil值
```
语法：HMGET KEY_NAME FIELD1...FIELDN 

示例：
127.0.0.1:6379> HMGET myhash field1 field2 field3
1) "hoo~"
2) "7"
3) (nil)
```

### HMSET
* 同时将多个field-value对设置到哈希表key中
* 会覆盖哈希表中已存在的域
* key不存在，那么一个空哈希表会被创建并执行HMSET操作
```
语法：HMSET KEY_NAME FIELD1 VALUE1 ...FIELDN VALUEN 

示例：
127.0.0.1:6379> HMSET myhash field3 yoooo field4 mooo
OK
127.0.0.1:6379> HGETALL myhash
 1) "field1"
 2) "hoo~"
 3) "field2"
 4) "7"
 5) "field"
 6) "-3"
 7) "field3"
 8) "yoooo"
 9) "field4"
10) "mooo"
```

### HVALS
* 返回哈希表key中所有域的值
```
语法：HVALS KEY_NAME

示例：
127.0.0.1:6379> HVALS myhash
1) "hoo~"
2) "7"
3) "-3"
4) "yoooo"
5) "mooo"
```
<br>

## List(列表)
* Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
* 它的底层实际是个链表
### LPUSH
* 将一个或多个值value插入到列表key的表头
* 如果有多个value的值，那么各个value值按从左到右的顺序依次插入表头
* key不存在，一个空列表会被创建并执行LPUSH操作
* key存在但不是列表类型，返回错误
```
语法：LPUSH KEY_NAME VALUE1.. VALUEN

示例：
127.0.0.1:6379> LPUSH list1 "yoo" "moo~"
(integer) 2
127.0.0.1:6379> LRANGE list1 0 -1
1) "moo~"
2) "yoo"
```

### LPUSHX
* 将值value插入到列表key的表头，当且仅当key存在且为一个列表
* key不存在时，LPUSHX命令什么都不做
```
语法：LPUSHX KEY VALUE1.. VALUEN

示例：
127.0.0.1:6379> LPUSHX list1 "baaa"
(integer) 3
127.0.0.1:6379> LPUSHX list2 "laaa"
(integer) 0
127.0.0.1:6379> LRANGE list1 0 -1
1) "baaa"
2) "moo~"
3) "yoo"
```

### LPOP
* 移除并返回列表key的头元素
```
语法：LPOP key

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "baaa"
2) "moo~"
3) "yoo"
127.0.0.1:6379> LPOP list1
"baaa"
127.0.0.1:6379> LRANGE list1 0 -1
1) "moo~"
2) "yoo"
```

### LRANGE
* 返回列表key中指定区间内的元素，区间以偏移量start和end指定
* start和end都以0为底
* 可使用负数下标，-1表示列表最后一个元素，-2表示列表倒数第二个元素，以此类推
* start大于列表最大下标，返回空列表
* end大于列表最大下标，end=列表最大下标
```
语法：LRANGE KEY START END

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "moo~"
2) "yoo"
127.0.0.1:6379> LRANGE list1 5 10
(empty list or set)
```

### LREM
* 根据count的值，移除列表中与value相等的元素
* count>0表示从头到尾搜索，移除与value相等的元素，数量为count
* count<0表示从尾到头搜索，移除与value相等的元素，数量为count
* count=0表示移除表中所有与value相等的元素
```
语法：LREM KEY_NAME COUNT VALUE

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "666"
2) "666"
3) "666"
4) "moo~"
5) "yoo"
127.0.0.1:6379> LREM list1 3 666
(integer) 3
127.0.0.1:6379> LRANGE list1 0 -1
1) "moo~"
2) "yoo"
```

### LSET
* 将列表key下标为index的元素值设为value
* index参数超出范围，或对一个空列表进行LSET时，返回错误
```
语法：LSET KEY_NAME INDEX VALUE

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "moo~"
2) "yoo"
127.0.0.1:6379> LSET list1 0 "hoo~"
OK
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
```

### LINDEX
* 返回列表key中，下标为index的元素,可以使用负数下标
```
语法：LINDEX KEY_NAME INDEX_POSITION 

示例：
127.0.0.1:6379> LINDEX list1 0
"hoo~"
```

### LINSERT
* 将值value插入列表key中，位于值pivot前面或者后面
* pivot不存在于列表key时，不执行任何操作
* key不存在，不执行任何操作
* 如果key不是列表类型，返回一个错误
```
语法：LINSERT key BEFORE|AFTER pivot value

示例：
127.0.0.1:6379> LINSERT list1 before "hoo~" "miii~"
(integer) 3
127.0.0.1:6379> LRANGE list1 0 -1
1) "miii~"
2) "hoo~"
3) "yoo"
```

### LLEN
* 返回列表key的长度
* key不存在，则返回0
* 如果key不是列表类型，返回一个错误
```
语法：LLEN KEY_NAME 

示例：
127.0.0.1:6379> LLEN list1
(integer) 3
```

### LTRIM
* 对一个列表进行修剪，让列表只返回指定区间内的元素，不存在指定区间内的都将被移除
```
语法：LTRIM KEY_NAME START STOP

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "miii~"
2) "hoo~"
3) "yoo"
127.0.0.1:6379> LTRIM list1 1 -1
OK
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
```

### RPOP
* 移除并返回列表key的尾元素
```
语法：RPOP KEY_NAME

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
3) "666"
127.0.0.1:6379> RPOP list1
"666"
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
```

### RPOPLPUSH
* 在一个原子时间内，执行两个动作：
  * 将列表source中最后一个元素弹出并返回给客户端
  * 将source弹出的元素插入到列表destination,作为destination列表的头元素
```
语法：RPOPLPUSH SOURCE_KEY_NAME DESTINATION_KEY_NAME

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
3) "666"
127.0.0.1:6379> RPOPLPUSH list1 list2
"666"
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
127.0.0.1:6379> LRANGE list2 0 -1
1) "666"
```

### RPUSH
* 将一个或多个值value插入到列表key的表尾
* key不存在，则会自动创建
* 当key存在但不是列表类型时，返回一个错误。
```
语法：RPUSH KEY_NAME VALUE1..VALUEN

示例：
127.0.0.1:6379> RPUSH list1 666 777
(integer) 4
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
3) "666"
4) "777"
127.0.0.1:6379> RPUSH list99 666
(integer) 1
127.0.0.1:6379> lrange list99 0 -1
1) "666"
```

### RPUSHX
* 将value插入到列表key的末尾，当且仅当key存在并且是一个列表
* key不存在，RPUSHX什么都不做
```
语法：RPUSHX KEY_NAME VALUE1..VALUEN

示例：
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
3) "666"
4) "777"
127.0.0.1:6379> RPUSHX lsit1 888
(integer) 0
127.0.0.1:6379> RPUSHX list1 888
(integer) 5
127.0.0.1:6379> LRANGE list1 0 -1
1) "hoo~"
2) "yoo"
3) "666"
4) "777"
5) "888"
```

### BLPOP、BRPOP、BRPOPLPUSH
* 这三个是几个pop的阻塞版本
* 即没有数据可以弹出的时候将阻塞客户端直到超时或者发现有可以弹出的元素为止
<br>

## Set(集合)
* Redis的Set是string类型的无序集合。它是通过HashTable实现的

