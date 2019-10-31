spring boot+@Cacheable+redis的基本配置
----------

开启springboot的cache步骤如下：

1.pom.xml加入redis起步依赖
```xml  
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>	
```


2.启动Application加入@EnableCaching源注释
```java
@SpringBootApplication
@EnableCaching  //开启缓存
public class DemoApplication{
 
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
 
}
```

3.使用@Cacheable等相关cache源注释标注要缓存处理的类
```java
package com.sc.oauth2;

import java.util.List;

import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.security.oauth2.provider.ClientAlreadyExistsException;
import org.springframework.security.oauth2.provider.ClientDetails;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.ClientRegistrationException;
import org.springframework.security.oauth2.provider.ClientRegistrationService;
import org.springframework.security.oauth2.provider.NoSuchClientException;

/**
 * 使用缓存包装ClientDetailsService, ClientRegistrationService
 * @author zhangdb
 *
 */
@CacheConfig(cacheNames = { "oauth2ClientDetailsServiceCache" })
public class ClientDetailsServiceCacheDecorator implements ClientDetailsService, ClientRegistrationService {
	/** 原始ClientDetailsService*/
	private final ClientDetailsService clientDetailsService;
	/** 原始clientRegistrationService*/
	private final ClientRegistrationService clientRegistrationService;

	public ClientDetailsServiceCacheDecorator(ClientDetailsService clientDetailsService, ClientRegistrationService clientRegistrationService) {
		super();
		this.clientDetailsService = clientDetailsService;
		this.clientRegistrationService = clientRegistrationService;
	}

	@Override
	public void addClientDetails(ClientDetails clientDetails) throws ClientAlreadyExistsException {
		this.clientRegistrationService.addClientDetails(clientDetails);
	}

	@CacheEvict(key = "#clientDetails.clientId")
	@Override
	public void updateClientDetails(ClientDetails clientDetails) throws NoSuchClientException {
		this.clientRegistrationService.updateClientDetails(clientDetails);
	}

	@CacheEvict(key = "#clientId")
	@Override
	public void updateClientSecret(String clientId, String secret) throws NoSuchClientException {
		this.clientRegistrationService.updateClientSecret(clientId, secret);
	}

	@CacheEvict(key = "#clientId")
	@Override
	public void removeClientDetails(String clientId) throws NoSuchClientException {
		this.clientRegistrationService.removeClientDetails(clientId);
	}

	@Cacheable(key = "listClientDetails")
	@Override
	public List<ClientDetails> listClientDetails() {
		return this.clientRegistrationService.listClientDetails();
	}

	@Cacheable(key = "#clientId")
	@Override
	public ClientDetails loadClientByClientId(String clientId) throws ClientRegistrationException {
		return this.clientDetailsService.loadClientByClientId(clientId);
	}

}

```

4.application.yml加入redis配置
```yml
spring:
  redis:
    host: 192.168.5.36
    port: 6379
```    

    
[redis参数配置文档]()

以上的配置就可以正确的运行spring boot+@Cacheable+redis的项目了。

实际项目中一些问题的解决方案
----------

> 1.设置缓存的过期时间
默认情况下的RedisCacheManager不会设置缓存过期时间，也就是说缓存永久生效。这里我们基于spring的bean后置处理来设置相关的过期时间属性。  

> 2.设置缓存key的序列化器(字符串序列化器)  
RedisTemplate默认的序列化器是jdk自带的序列化，其在性能上、跨平台、调试(可视性)上都不好，因此这里使用字符串key的序列化。


如下的代码，使用bean后置处理器，对spring自动配置的默认bean进行一些加工，已满足以上的需求：
```java
package com.sc.oauth2;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

@Component
@EnableCaching
@ConfigurationProperties(prefix = "spring.redis")
@EnableConfigurationProperties(RedisCacheBeanPostProcessor.class)
public class RedisCacheBeanPostProcessor implements BeanPostProcessor {
	/** 缓存对象默认的过期时间 */
	private long defaultExpireTime = 0;
	/** 指定某个缓存对象(cacheName)的过期时间 */
	private Map<String, Long> expires = new HashMap<>();

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		Object processAfterBean = bean;
		processAfterBean = this.redisTemplate(processAfterBean, beanName);
		processAfterBean = this.cacheManager(processAfterBean, beanName);
		return processAfterBean;
	}

	protected Object redisTemplate(Object bean, String beanName) {
		if (bean instanceof RedisTemplate) {
			RedisTemplate<?, ?> redisTemplate = (RedisTemplate<?, ?>) bean;
			// 注意key的序列化器,使用的StringSerializer,这就要求缓存的key类型必须是String类型,特别是使用@Cacheable时,要注意参数类型和 ''+#xxx.id 转换
			redisTemplate.setKeySerializer(redisTemplate.getStringSerializer());
			redisTemplate.setHashKeySerializer(redisTemplate.getStringSerializer());
		}
		return bean;
	}

	protected Object cacheManager(Object bean, String beanName) {
		if (bean instanceof RedisCacheManager) {
			RedisCacheManager cacheManager = (RedisCacheManager) bean;
			cacheManager.setDefaultExpiration(this.defaultExpireTime);
			cacheManager.setExpires(this.expires);
		}
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	public long getDefaultExpireTime() {
		return defaultExpireTime;
	}

	public void setDefaultExpireTime(long defaultExpireTime) {
		this.defaultExpireTime = defaultExpireTime;
	}

	public Map<String, Long> getExpires() {
		return expires;
	}

	public void setExpires(Map<String, Long> expires) {
		this.expires = expires;
	}

}

```

application.yml配置如下：

```yml
spring:
  redis:
    defaultExpireTime: 30
    expires:
      oauth2ClientDetailsServiceCache : 60
```
这里的defaultExpireTime配置定义了默认的过期时间，expires定义了某个cacheName的过期时间。

> 3.@CacheEvict(allEntries = true)的性能问题

我们看一下代码(org.springframework.data.redis.cache.RedisCache)：
```java
	public void clear() {
		redisOperations.execute(cacheMetadata.usesKeyPrefix() ? new RedisCacheCleanByPrefixCallback(cacheMetadata)
				: new RedisCacheCleanByKeysCallback(cacheMetadata));
	}
```
其会根据是否使用了key前缀字符串，来使用不同的清除方法：
>> 使用了key前缀，则使用基于redis+lua的方式来清除key前缀对应的缓存，如下代码()：
```java
		private static final byte[] REMOVE_KEYS_BY_PATTERN_LUA = new StringRedisSerializer().serialize(
				"local keys = redis.call('KEYS', ARGV[1]); local keysCount = table.getn(keys); if(keysCount > 0) then for _, key in ipairs(keys) do redis.call('del', key); end; end; return keysCount;");
```
如果要是集群连接的情况下，更糟糕，其会使用原始的```java keys keyPfrefix* ```查找，然后遍历删除对应的key。

>> 没有使用key前缀，则使用zRange命令一次获取PAGE_SIZE(128)个key，返回再逐个删除对应的key，反复循环。
```java
			int offset = 0;
			boolean finished = false;

			do {
				// need to paginate the keys
				Set<byte[]> keys = connection.zRange(metadata.getSetOfKnownKeysKey(), (offset) * PAGE_SIZE,
						(offset + 1) * PAGE_SIZE - 1);
				finished = keys.size() < PAGE_SIZE;
				offset++;
				if (!keys.isEmpty()) {
					connection.del(keys.toArray(new byte[keys.size()][]));
				}
			} while (!finished);

			connection.del(metadata.getSetOfKnownKeysKey());
```

好文章介绍
https://blog.csdn.net/cpongo3/article/details/89882195

