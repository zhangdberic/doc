**1.发送/user请求无法获取(Principal user)问题。**  
> 描述：这个是版本的BUG，目前Spring Boot 1.5.22.RELEASE + Spring Cloud Edgware.SR6组合会出现标题的问题。  
> 解决方案：调整oauth2资源过滤器的顺序，如下：
```yml  
security:
  oauth2:
    resource:
      filter-order: 3    
```  


