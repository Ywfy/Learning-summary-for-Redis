# 事务(transaction)

## 概念
可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其它命令插入，不许加塞

## 常用命令
|序号|命令及描述|
|:---|:---|
|1|DISCARD<br>取消事务，放弃执行事务块内的所有命令|
|2|EXEC<br>执行所有事务块内的命令|
|3|MULTI<br>标记一个事务块的开始|
|4|UNWATCH<br>取消WATCH命令对所有key的监视|
|5|WATCH key[key...]<br>监视一个(或多个)key，如果在事务执行之前这个(或这些)key被其他命令所改动，那么事务将被打断|

## 事务三阶段
* 开启(MULTI)
* 入队
* 执行(EXEC)

## 事务三特性
* 单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行， 也就不存在”事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
* 不保证原子性：redis同一个事务中如果有一条命令执行失败(注意是执行失败，在入队时出现error的话最后EXEC是无法执行的)，其后的命令仍然会被执行，没有回滚

## 使用场景：
* 正常执行
```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 666
QUEUED
127.0.0.1:6379> set k2 777
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 888
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
3) "777"
4) OK
```
* 放弃事务
```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 1
QUEUED
127.0.0.1:6379> set k2 2
QUEUED
127.0.0.1:6379> set k3 3
QUEUED
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> get k2
"777"
```
* 一致性(注意此时是在加入队列时报错)
```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 1
QUEUED
127.0.0.1:6379> set k2 2
QUEUED
127.0.0.1:6379> setgwe k3 3
(error) ERR unknown command 'setgwe'
127.0.0.1:6379> set k4 4
QUEUED
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k2
"777"
```
* 冤头债主(注意此时在加入队列时没报错，是在执行后报错)
```
127.0.0.1:6379> set k1 a
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> INCR k1
QUEUED
127.0.0.1:6379> set k2 2
QUEUED
127.0.0.1:6379> set k3 3
QUEUED
127.0.0.1:6379> set k6 6
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range
2) OK
3) OK
4) OK
127.0.0.1:6379> get k6
"6"
```
* WATCH监控
类似乐观锁
```
终端1：
127.0.0.1:6379> set aa 5
OK
127.0.0.1:6379> set bb 5
OK
127.0.0.1:6379> WATCH aa
OK

终端2：
127.0.0.1:6379> set aa 10
OK

终端1：
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> DECRBY aa 3
QUEUED
127.0.0.1:6379> INCRBY bb 3
QUEUED
127.0.0.1:6379> EXEC
(nil)
```
* 一旦执行exec或者unwatch，之前加的监控锁都将被取消
