# Jedis
(本教程是在Linux运行Redis，Windows运行Eclipse的环境下)
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

## Jedis事务

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
