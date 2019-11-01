在生产环境下运行oauth2需要满足如下几点：

* 基于数据库存储clientDetials、userDetails。  

* 访问clientDetails和userDetails都需要缓存支撑(redis)。  

* 基于redis存放token。  

* 复杂的权限表设计，参见[表设计](https://github.com/zhangdberic/doc/blob/master/springcloud/oauth2/oauth2%E5%90%8E%E5%8F%B0%E8%A1%A8%E7%BB%93%E6%9E%84%E8%AE%BE%E8%AE%A1.md)。  

* 独立的oauth2数据管理器(界面)、提供oauth2数据操作服务，参见[oauth2管理器](https://github.com/zhangdberic/doc/blob/master/springcloud/oauth2/%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86%E5%99%A8%E8%AE%BE%E8%AE%A1.md)。    


