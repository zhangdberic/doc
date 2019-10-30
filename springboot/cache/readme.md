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
spring:  
  redis:  
    host: 192.168.5.36  
    port: 6379  
    
[redis参数配置文档]()

好文章介绍
https://blog.csdn.net/cpongo3/article/details/89882195

