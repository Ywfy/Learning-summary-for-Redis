# Redis发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。<br>
Redis 客户端可以订阅任意数量的频道。<br>
下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/message/pubsub1.png)<br>
当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/message/pubsub2.png)<br>


## 命令
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-Redis/blob/master/message/ml.png)<bt>

## 示例
例1：
```
终端1:
127.0.0.1:6379> SUBSCRIBE c1 c2 c3
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "c1"
3) (integer) 1
1) "subscribe"
2) "c2"
3) (integer) 2
1) "subscribe"
2) "c3"
3) (integer) 3

终端2：
127.0.0.1:6379> PUBLISH c2 hello-redis
(integer) 1

终端1：
2) "c2"
3) "hello-redis"


```

例2：
```
终端1：
127.0.0.1:6379>  PSUBSCRIBE new*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "new*"
3) (integer) 1

终端2：
127.0.0.1:6379> PUBLISH new1 redis2015
(integer) 1

终端1：
1) "pmessage"
2) "new*"
3) "new1"
4) "redis2015"

终端2：
127.0.0.1:6379> PUBLISH new13 redis2018
(integer) 1

终端1：
1) "pmessage"
2) "new*"
3) "new13"
4) "redis2018"
```
