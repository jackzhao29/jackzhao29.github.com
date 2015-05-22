---
layout: post
title:  "使用Spring的spring-data-redis插件开发Redis客户端"
keywords: "redis,spring"
description: "Spring把专门的数据操作独立封装在spring-data系列中，spring-data-redis是针对Redis的独立封装，主要是将jedis、jredis、rjc以及srp等Redis Client进行了封装，同时支持事务"
category: redis
tags: [redis]
---
### 简述
Spring把专门的数据操作独立封装在spring-data系列中，spring-data-redis是针对Redis的独立封装，主要是将jedis、jredis、rjc以及srp等Redis Client进行了封装，同时支持事务。
### 客户端配置文件
#### 对象池配置 

```
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<property name="maxActive" value="${redis.pool.maxActive}" />
	<property name="maxIdle" value="${redis.pool.maxIdle}" />
	<property name="maxWait" value="${redis.pool.maxWait}" />
</bean>
```
#### JedisConnectionFactory配置文件

```
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
<property name="masterHost" value="${redis.master.host}"/>
	<property name="masterPort" value="${reids.master.port}"/>
	<property name="timeOut" value="${redis.pool.timeout}" />
    <property name="poolConfig" ref="jedisPoolConfig" />
</bean>
```
#### RedisTemplate类的配置文件

```
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate" p:connection-factory-ref="jedisConnectionFactory" />
```
### 示例代码

```
//接口类
public class CacheService{
    //传入参数final标示，为了禁止方法内修改
	public void add(final String key,String value);
	public void get(final String key);
}
//接口实现类
public class CacheServiceImpl implements CacheService{
	@Resource
	Private RedisTemplate redisTemplate;
	@Override
	public void add(final String key,String value){
		//实现会掉，方便控制事务
		redisTemplate.execute(new RedisCallback<Object>(){
			@Override
			public Object doInRedis(RedisConnection conn) throws DataAccessException{
			    //序列化参数
			    byte[] key=redisTemplate.getStringSerializer().serialize("key_"+key);
			    byte[] value=redisTemplate.getStringSerializer().serialize("value_"+value);
				//redis的set方法
				conn.set(key,value);
				return null;
			}
		});
	}
	
   @Override
   public String get(final String key){
   		return redisTemplate.execute(new RedisCallback<String>{
   			@Override
   			public String doInRedis(RedisConnection conn){
   				byte[] key=redisTemplate.getStringSerializer().serialize("key_"+key);
   				if(conn.exists(key)){
   					byte[] value=redisTemplate.getStringSerializer().deserialize(value);
   				}
   			}
   		});
   }
}
```
更多的使用方法参考[Spring Data Redis](http://docs.spring.io/spring-data/data-redis/docs/current/reference/html/)


