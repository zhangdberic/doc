clientDetails表结构，就使用oauth2的标准表结构，这里不再多说了，
关键在**用户权限(用户授权)表**的设计和不同服务实例的**资源服务配置类**的设计。  

资源服务配置类设计  
----  


一个大型的微服务群，是由多个jvm实例组成的，这里的jvm实例可以理解为一个jar进程或者是docker，一个jvm实例内部可以只包括一个服务，也可以包括多个服务。  
不管内部是多个服务还是一个服务，都要配置一个服务资源类(授权访问类)，需要继承ResourceServerConfigurerAdapter类。  
为了设计一个通用的**服务资源类**，把资源服务类配置分成不同的规则层次。  

1.无需要认证直接访问服务(公共服务)。  
理解为不需要认证直接就可以调用服务。  

2.需要认证才能访问的服务。  
只要认证(登录)后就可以访问服务。

3.需要授权访问的服务。  
不但需要认证(登录)，而且这个认证(用户)还需要由可以访问服务的权限。

```java
    @Override
    public void configure(HttpSecurity http) throws Exception {
      // 无需要认证直接访问服务(公共服务)
      http.authorizeRequests().antMatchers("/xxxx/xxx1","/xxxx/xxx2").permitAll();
      // 需要认证才能访问的服务
      http.authorizeRequests().anyRequest().authenticated();
      // 需要授权访问的服务
      http.authorizeRequests().antMatchers("/oauth/remove_token").hasAuthority("123");
      http.authorizeRequests().antMatchers("/oauth/remove_token").access("hasIpAddress('192.168.5.31')");
    }
```

第一条规则(无需认证直接访问)：这个可以通过配置配置文件来加载，并生成相应的java代码。  

第二条规则(需要认证才能访问的服务)：http.authorizeRequests().anyRequest().authenticated(); 这行代码就可以搞定了。  

第三条规则(需要授权访问的服务)：这个比较复杂，要根据表数据来生成对应的java代码。

用户授权相关表设计  
---- 

> 权限表设计(oauth2_authority)

| authority_id | name     | app       | method | url_ant_patterns | access_spel                  |
| ------------ | -------- | --------- | ------ | ---------------- | ---------------------------- |
| 1            | 增加数据 | dmservice | post   | /data/add        | null                         |
| 2            | 查看数据 | dmservice | *      | /data/*          | null                         |
| 3            | 删除数据 | dmservice | post   | /data/delete     | hasIpAddress('192.168.5.31') |

authority_id为表序号，唯一序列，权限码，对应hasAuthority(authority_id)。  
name对相关的记录命名，描述自动。  
app为对应的应用，这个权限配置隶属于那个应用，不同的应用启动会加载不同的权限规则。  
method限制请求的http_method，对应antMatchers(method)。  
url_ant_patterns匹配的url，对应antMatchers(url_ant_patterns)。  
access_spel使用spring表达式匹配，对应access("hasIpAddress(access_spel)")  

我们重新再来看上面的HttpSecurity规则设定，在应用或服务启动的时候，加载权限表数据，根据规则来创建HttpSecurity对象。  

例如：启动的应用是一个服务dmservice  

加载权限数据  
select * from oauth2_authority where app=dmservice  

第一条记录对应的规则如下：  
http.authorizeRequests().antMatchers(HttpMethod.POST).antMatchers("/data/add").hasAuthority("1");  
第二条记录对应的规则如下：  
http.authorizeRequests().antMatchers("/data/\*").hasAuthority("2");  
第三条记录对应的规则如下：
http.authorizeRequests().antMatchers(HttpMethod.POST).antMatchers("/data/delete").hasAuthority("3").
antMatchers("/data/delete").access("hasIpAddress('192.168.5.31')");  

> 用户表设计(oauth2_user)  

| user_id | username | password   | enabled |
| ------- | -------- | ---------- | ------- |
| 1       | heige    | heige123   | 1       |
| 2       | jiaojie  | jiaojie123 | 1       |
|         |          |            |         |

> 用户权限表(oauth2_user_authority)  

| user_id | authority_id |
| ------- | ------------ |
| 1       | 1            |
| 1       | 2            |
| 1       | 3            |
| 2       | 2            |

通过上面的用户表和用户权限表设计，应用端发送/user请求验证token的同时，也获取到了这个用户对应的user和authorities信息，进而**服务资源类**内的HttpSecurity规则匹配，匹配正确可以访问否则，否则访问拒绝。当然这个过程是spring oauth2 client自动完成的。  


请求/user性能优化
----  
因为每次服务调用都oauth2 client底层都要发送请求/user到oauth2来获取用户和权限数据，因为按照上面的设计权限数据如果很多，那么会影响请求的响应时间而且对oauth2服务器照成压力，因此应该对/user请求进行缓存处理，例如：基于redis缓存，key=token，value=/user返回的json数据，expire=60s，这个过期时间根据应用情况，在配置文件中调整。

补充  
----  
为了在oauth2后台管理器，管理方便，可以再在表oauth2_authority加入两个字段：  
1.parentId 上级id，指定上级id，如果一个id被指定为了上级id那么这个authority就目录(树形结构)了。
2.sortNo 排序吗，指定在同一个目录下，显示的顺序。
