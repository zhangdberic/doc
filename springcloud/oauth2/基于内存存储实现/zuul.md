如果令牌要穿透zuul传递的话，还需要设置一下yml  

```yml  
zuul: 
  sensitive-headers: Cookie,Set-Cookie
```  
zuul默认是设置了Cookie,Set-Cookie,Authorization三个请求头都为敏感请求头，也就是sensitive-headers内配置的请求头不会经过zuul转发到服务。
