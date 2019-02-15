# Jedis
(本教程是在虚拟机Linux运行Redis，Windows运行Eclipse的环境下)
## 连通测试
创建Dynamic Web Project，导入以下jar包
```
commons-pool-1.6.jar
jedis-2.1.0.jar
```
在src下创建类TestPing
```
package com.guigu.redis.test;

import redis.clients.jedis.Jedis;

public class TestPing {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Jedis jedis = new Jedis("192.168.88.131", 6379);
		System.out.println(jedis.ping());
	}

}
```
运行结果：
```
PONG
```

## 常用API
```
package com.guigu.redis.test;

import java.util.Iterator;
import java.util.*;

import redis.clients.jedis.Jedis;

public class TestAPI {
	public static void main(String[] args) {
		Jedis jedis = new Jedis("192.168.88.131", 6379);
		
		jedis.set("k1", "1");
		jedis.set("k2", "2");
		jedis.set("k3", "3");
		
		System.out.println(jedis.get("k2"));
		
		Set<String> myset = jedis.keys("*");
		System.out.println(myset.size());
		for(Iterator iterator = myset.iterator();iterator.hasNext();) {
			String key = (String)iterator.next();
			System.out.println(key);
		}
		System.out.println("Jedis.exists===>" + jedis.exists("k2"));
		System.out.println(jedis.ttl("k1"));
		
		//String
		jedis.append("k1", "myredis");
		System.out.println(jedis.get("k1"));
		jedis.set("k4", "fafafa");
		System.out.println("****************************");
		jedis.mset("str1", "v1", "str2", "v2", "str3", "v3");
		System.out.println(jedis.mget("str1", "str2", "str3"));
		//list
		System.out.println("****************************");
		jedis.lpush("mylist", "5","6","7","8","9");
		List<String> list = jedis.lrange("mylist", 0, -1);
		for(String element : list) {
			System.out.println(element);
		}
		
		//set
		jedis.sadd("orders", "tm001");
		jedis.sadd("orders", "tm002");
		jedis.sadd("orders", "tm003");
		Set<String> set1 = jedis.smembers("orders");
		for(Iterator iterator = set1.iterator(); iterator.hasNext();) {
			String string = (String)iterator.next();
			System.out.println(string);
		}
		jedis.srem("orders", "tm002");
		System.out.println(jedis.smembers("orders").size());
		
		//hash
		jedis.hset("myhash", "userName", "mm");
		System.out.println(jedis.hget("myhash", "userName"));
		Map<String,String> map = new HashMap<String,String>();
		map.put("telphone", "12345678902");
		map.put("address", "guigu");
		map.put("email", "abc@666.com");
		jedis.hmset("myhash2", map);
		List<String>  result = jedis.hmget("myhash2", "telphone", "email");
		for(String element : result) {
			System.out.println(element);
		}
		
		//zset
		jedis.zadd("myzset", 60d, "v1");
		jedis.zadd("myzset", 70d, "v2");
		jedis.zadd("myzset", 80d, "v3");
		jedis.zadd("myzset", 90d, "v4");
		
		Set<String> s1 = jedis.zrange("myzset", 0, -1);
		for(Iterator iterator = s1.iterator(); iterator.hasNext();) {
			String str = (String)iterator.next();
			System.out.println(str);
		}
		
	}
}
```
运行结果
```
2
5
k3
aa
niubi
k1
k2
Jedis.exists===>true
-1
1myredis
****************************
[v1, v2, v3]
****************************
9
8
7
6
5
tm001
tm002
tm003
2
mm
12345678902
abc@666.com
v1
v2
v3
v4
```
## Jedis事务
```
package com.guigu.redis.test;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

public class TestTX {
	
	public boolean transMethod() throws InterruptedException {
		Jedis jedis = new Jedis("192.168.88.131", 6379);
		int balance; //信用卡可用余额
		int debt; //欠额
		int amtToSubtract = 10;  //本次消费额度
		
		jedis.watch("balance");
		Thread.sleep(7000);
		balance = Integer.parseInt(jedis.get("balance"));
		if(balance < amtToSubtract) {
			jedis.unwatch();
			System.out.println("modify");
			return false;
		}else {
			System.out.println("**************transaction");
			Transaction transaction = jedis.multi();
			transaction.decrBy("balance", amtToSubtract);
			transaction.incrBy("debt", amtToSubtract);
			transaction.exec();
			balance = Integer.parseInt(jedis.get("balance"));
			debt = Integer.parseInt(jedis.get("debt"));
			
			System.out.println("*********" + balance);
			System.out.println("*********" + debt);
			return true;
		}
	}
	
	public static void main(String[] args) throws InterruptedException {
		
		TestTX test = new TestTX();
		boolean retValue = test.transMethod();
		System.out.println("Main RetValue====>" + retValue);
	}
}
```
正常运行
```
**************transaction
*********90
*********10
Main RetValue====>true
```

在运行后，迅速将balance的值改为8，结果如下
```
modify
Main RetValue====>false
```

在运行后，迅速改变balance的值(>=10)，如果如下
```
**************transaction
*********66
*********0
Main RetValue====>true
```
## Jedis主从复制
```
package com.guigu.redis.test;

import redis.clients.jedis.Jedis;

public class TestMS {
	public static void main(String[] args) {
		Jedis jedis_M = new Jedis("192.168.88.131", 6379);
		Jedis jedis_S = new Jedis("192.168.88.131", 6380);
		
		jedis_S.slaveof("192.168.88.131", 6379);
		
		jedis_M.set("niubi", "666");
		
		String result = jedis_S.get("niubi");
		System.out.println(result);
	}
}
```
运行结果：(注意，第一次运行结果可能为null，因为内存太快了，运行第二遍确认)
```
666
```

## JedisPool
JedisPoolUtil:
```
package com.guigu.redis.test;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class JedisPoolUtil {
	
	private static volatile JedisPool jedisPool = null;
	
	private JedisPoolUtil() {}
	
	public static JedisPool getJedisPoolInstance() {

		if(null == jedisPool) {
			synchronized(JedisPoolUtil.class) {
				if(null == jedisPool) {
					JedisPoolConfig poolConfig = new JedisPoolConfig();
					
					poolConfig.setMaxActive(1000);
					poolConfig.setMaxIdle(32);
					poolConfig.setMaxWait(100*1000);
					poolConfig.setTestOnBorrow(true);
					
					jedisPool = new JedisPool(poolConfig, "192.168.88.131", 6379);
				}
			}
		}
		return  jedisPool;
	}
	
	public static void release(JedisPool jedisPool, Jedis jedis) {
		if(null != jedis) {
			jedisPool.returnResourceObject(jedis);
		}
	}
}	
```
TestPool:
```
package com.guigu.redis.test;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class TestPool {
	public static void main(String[] args) {
		
		JedisPool jedisPool = JedisPoolUtil.getJedisPoolInstance();	
		Jedis jedis = null;
		
		try 
		{
			jedis = jedisPool.getResource();
			
			jedis.set("aa", "11");		
		}catch(Exception e){
			e.printStackTrace();
		}finally {
			JedisPoolUtil.release(jedisPool, jedis);
		}
	}
}

```
运行结果：
```
127.0.0.1:6379> get "aa"
"11"
```
