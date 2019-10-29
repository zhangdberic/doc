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

spring cloud oauth2在启动的时候，会创建5个SecurityFilterChain接口的的实现类DefaultSecurityFilterChain，DefaultSecurityFilterChain类内包括一个匹配器(OrRequestMatcher)，用于对请求URL进行匹配，还包括一个filters，用于URL匹配成功后执行的过滤器列表。也就是说5个DefaultSecurityFilterChain，对应5个不同的执行路径，那个匹配上了就走那个路径。

1. 第一个DefaultSecurityFilterChain

   匹配路径：

   ```java
   requestMatchers=[Ant [pattern='/css/**'], Ant [pattern='/js/**'], Ant [pattern='/images/**'], Ant [pattern='/webjars/**'], Ant [pattern='/**/favicon.ico'], Ant [pattern='/error']]
   ```

   执行操作：

   无任何操作，也就是oauth2不操作，直接交给tomcat来处理。

2. 第二个DefaultSecurityFilterChain

   匹配路径：

   ```java
   requestMatchers=[Ant [pattern='/oauth/token'], Ant [pattern='/oauth/token_key'], Ant [pattern='/oauth/check_token']]
   ```

   执行操作：

   ![1572309352855](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1572309352855.png)

3. 第二个DefaultSecurityFilterChain

4. 