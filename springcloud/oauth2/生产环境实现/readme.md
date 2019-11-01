1.基于数据库存储clientDetials、userDetails。
2.访问clientDetails和userDetails都需要缓存支撑(redis)。  
3.基于redis存放token。  
4.复杂的权限表设计，参见[]()。
5.独立的oauth2数据管理器。
6.独立的oauth2数据库管理服务。
