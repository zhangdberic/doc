# SecurityContextPersistenceFilter

安全上下文保持过滤器，其作用，把SecurityContext对象保存到SecurityContextHolder静态类中，使业务代码可以通过SecurityContextHolder静态类来获取认证(Authentication)信息。

SecurityContextHolder其持有这个ThreadLocal<SecurityContext>的静态域，基于线程变量来实现上下文对象存储。

SecurityContextPersistenceFilter过滤器，就是起到SecurityContextHolder的赋值和清理工作。
