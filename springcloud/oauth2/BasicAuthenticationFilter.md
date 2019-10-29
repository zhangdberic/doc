# BasicAuthenticationFilter

**完整的类路径：**

org.springframework.security.web.authentication.www.BasicAuthenticationFilter



**作用：**根据请求头Authorization的basic值来获取对应的Authentication，这里的basic值为用户名:密码(username:password)。

例如：请求/oauth/token，来获取access_token，请求头：Authentication的basic client:secret。



**源代码解析：**

String header = request.getHeader("Authorization");

获取Authorization的basic值，并解析出username和password，这里为clientId和secret。



再根据请求的username、password、remoteAddress、sessionId，创建UsernamePasswordAuthenticationToken对象，其实现了Authentication接口。



再以UsernamePasswordAuthenticationToken对象为参数，调用authenticationManager对象的authenticate方法来获取认证信息对象。



这里的authenticationManager接口实现类org.springframework.security.authentication.ProviderManager，其内有，private List<AuthenticationProvider> providers;和private AuthenticationManager parent;，首先逐个匹配providers，如果那个AuthenticationProvider支持，就使用这个AuthenticationProvider来获取Authentication，如果providers内的AuthenticationProvider都不支持，再使用parent来获取Authentication。如果parent都不能获取到Authentication，则抛出异常ProviderNotFoundException。



其中/oauth/token请求，对应的AuthenticationProvider为DaoAuthenticationProvider，具体的DaoAuthenticationProvider代码解析参见，，

**注意：DaoAuthenticationProvider.userDetailsService属性，对应的类是ClientDetailsUserDetailsService，其实现了UserDetailsService接口，并对ClientDetailsService进行了包装。**



获取到正确的UsernamePasswordAuthenticationToken后(Authentication)，其会赋值上下文对象，SecurityContextHolder.getContext().setAuthentication(authResult);



