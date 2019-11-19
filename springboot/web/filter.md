添加过滤器
==========  


例如：  

``` java
	@Bean
	@ConditionalOnProperty(value="server.response.content-length",matchIfMissing=false)
	public FilterRegistrationBean filterRegistrationBean() {
		FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
		filterRegistrationBean.setFilter(new Filter() {
			@Override
			public void init(FilterConfig filterConfig) throws ServletException {
			}

			@Override
			public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
				ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper((HttpServletResponse) response);
		        chain.doFilter(request, responseWrapper);
		        responseWrapper.copyBodyToResponse();
			}

			@Override
			public void destroy() {
			}
		});
		List<String> urls = new ArrayList<String>();
		urls.add("/*");
		filterRegistrationBean.setUrlPatterns(urls);
 		filterRegistrationBean.setName("response-content-length");
		filterRegistrationBean.setOrder(1);
		return filterRegistrationBean;
	}
```  

声明一个FilterRegistrationBean，这个FilterRegistrationBean就是一个代理外壳，负责注册过滤器。

@ConditionalOnProperty(value="server.response.content-length",matchIfMissing=false)  
这个是一个配置技巧，可以根据外部配置决定这个是否注册这个过滤器。  
