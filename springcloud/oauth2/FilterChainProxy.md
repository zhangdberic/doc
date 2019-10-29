# FilterChainProxy

FilterChainProxy是整个oauth2请求处理的入口，其继承了spring的GenericFilterBean类，并实现了**javax.servlet.Filter**接口。在oauth2服务器端启动的时候，就会自动装配并启动。

**每个对oauth2服务器的http请求都会经过它。**

FilterChainProxy包括了一个 private List<SecurityFilterChain> filterChains; 属性，列表内的SecurityFilterChain，会被逐个执行。

SecurityFilterChain接口代码如下：

```java
package org.springframework.security.web;

import javax.servlet.Filter;
import javax.servlet.http.HttpServletRequest;
import java.util.*;

public interface SecurityFilterChain {

	boolean matches(HttpServletRequest request);

	List<Filter> getFilters();
}
```

注意这里的List<Filter> getFilters(); Filter还是javax.servletFilter。

spring cloud oauth2在启动的时候，会创建5个SecurityFilterChain接口的的实现类DefaultSecurityFilterChain，DefaultSecurityFilterChain类内包括一个匹配器(RequestMatcher)，用于对请求URL进行匹配，还包括一个filters，用于URL匹配成功后执行的过滤器列表。

5个DefaultSecurityFilterChain，对应5个不同的执行路径，那个匹配上了就走那个路径。

1. 第一个DefaultSecurityFilterChain

   **作用：静态页面处理**

   匹配路径：

   ```java
   requestMatchers=[Ant [pattern='/css/**'], Ant [pattern='/js/**'], Ant [pattern='/images/**'], Ant [pattern='/webjars/**'], Ant [pattern='/**/favicon.ico'], Ant [pattern='/error']]
   ```

   执行操作：

   无任何操作，也就是oauth2不操作，直接交给tomcat来处理。

2. 第二个DefaultSecurityFilterChain

   **作用：oauth2请求处理，/oauth2/token、/oauth2/token_key、/oauth2/check_token请求处理**

   匹配路径：

   ```java
   requestMatchers=[Ant [pattern='/oauth/token'], Ant [pattern='/oauth/token_key'], Ant [pattern='/oauth/check_token']]
   ```

   执行操作：

   WebAsyncManagerIntegrationFilter  # 待以后介绍
   SecurityContextPersistenceFilter # SecurityContextHolder赋值，存放Authentication到上下文件中。
   HeaderWriterFilter # 响应头输出拦截，待以后介绍
   LogoutFilter # 登出
   **BasicAuthenticationFilter** # 拦截Basic Authentication请求，例如：/oauth/token请求、请求头Authentication携带token的请求。具体介绍详见BasicAuthenticationFilter.md。
   RequestCacheAwareFilter # 待以后介绍
   SecurityContextHolderAwareRequestFilter # 待以后介绍 
   AnonymousAuthenticationFilter # 待以后介绍
   SessionManagementFilter # 待以后介绍	
   ExceptionTranslationFilter # 待以后介绍
   FilterSecurityInterceptor # 待以后介绍

3. 第三个DefaultSecurityFilterChain

   **作用：ResourceServerConfiguration（资源服务器）处理**，待理解

   匹配路径：

   无固定的ant匹配路径，匹配操作由类：org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfiguration$NotOAuthRequestMatcher实现。

   执行操作：

   WebAsyncManagerIntegrationFilter
   SecurityContextPersistenceFilter
   HeaderWriterFilter
   LogoutFilter
   **OAuth2AuthenticationProcessingFilter**
   RequestCacheAwareFilter	
   SecurityContextHolderAwareRequestFilter
   AnonymousAuthenticationFilter
   SessionManagementFilter	
   ExceptionTranslationFilter
   FilterSecurityInterceptor

4. 第四个DefaultSecurityFilterChain

   **作用：ManagementWebSecurityAutoConfiguration（Web安全管理器）处理，待理解**

   匹配路径：

   无固定的ant匹配路径，匹配操作由类：org.springframework.boot.actuate.autoconfigure.ManagementWebSecurityAutoConfiguration$LazyEndpointPathRequestMatcher实现。

   执行操作：

   WebAsyncManagerIntegrationFilter
   SecurityContextPersistenceFilter
   HeaderWriterFilter
   **CorsFilter**
   LogoutFilter
   BasicAuthenticationFilter
   RequestCacheAwareFilter	
   SecurityContextHolderAwareRequestFilter
   AnonymousAuthenticationFilter
   SessionManagementFilter	
   ExceptionTranslationFilter
   FilterSecurityInterceptor

5. 第五个DefaultSecurityFilterChain

   **作用：兜底，其匹配 /**，上面不能匹配的请求，都由这个过滤器来处理**

   匹配路径：

   ```java
   requestMatchers=[Ant [pattern='/**']]
   ```

   执行操作：

   WebAsyncManagerIntegrationFilter  # 待以后介绍
   SecurityContextPersistenceFilter # SecurityContextHolder赋值，存放Authentication到上下文件中。
   HeaderWriterFilter # 响应头输出拦截，待以后介绍
   LogoutFilter # 登出
   **BasicAuthenticationFilter** # 拦截Basic Authentication请求，例如：使用请求Authentication携带token的请求。具体介绍详见BasicAuthenticationFilter.md。
   RequestCacheAwareFilter # 待以后介绍
   SecurityContextHolderAwareRequestFilter # 待以后介绍 
   AnonymousAuthenticationFilter # 待以后介绍
   SessionManagementFilter # 待以后介绍	
   ExceptionTranslationFilter # 待以后介绍
   FilterSecurityInterceptor # 待以后介绍
