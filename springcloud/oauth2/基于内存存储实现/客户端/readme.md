*客户端pom.xml*  

```xml  
		<!-- spirng oauth2 client -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security.oauth</groupId>
			<artifactId>spring-security-oauth2</artifactId>
		</dependency>
```  

*application.yml加入验证令牌地址*  

```yml  
# oauth2 client
security:
  oauth2:
    resource:
      user-info-uri: http://localhost:8020/auth/user
```

*修改Application.java加入@EnableResourceServer*  
```java  
@SpringBootApplication
@EnableResourceServer
public class Sample1ServiceApplication {
	
	public static void main(String[] args) {
		SpringApplication.run(Sample1ServiceApplication.class, args);
	}

}
```  

*增加资源服务配置类*  
定义谁可以访问我的服务  
```java  
package com.sc.sample1service;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;

@Configuration
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
	
	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().anyRequest().authenticated();
	}

}
```  

以上配置单一服务调用时没问题的，但如果是服务之间的调用，则还是会抛出401错误(没有授权)  

*传递性*  

传递性，如果一个服务要调用另一个服务，则需要传递Authorization请求头。

如果使用的feign rpc访问调用，则无需任何特殊的配置，直接就支持Authorization请求头传递。

如果是基于RestTemplate发送请求，则要替换为OAuth2RestTemplate，如下：
```java  
	
	@Bean
	public OAuth2RestTemplate oauth2RestTemplate(OAuth2ClientContext oauth2ClientContext,OAuth2ProtectedResourceDetails details ) {
		return new OAuth2RestTemplate(details,oauth2ClientContext);
	}
```   


```java  
	@Autowired
	private OAuth2RestTemplate oauth2RestTemplate;
	
	public Organization getOrganization(String organizationId) {
		ResponseEntity<Organization> restExchange = oauth2RestTemplate.exchange("http://localhost/organizationservice/v1/organizations/{organizationId}", HttpMethod.GET, null, Organization.class, organizationId);
		return restExchange.getBody();
	}
```



