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



