[Spring Security源码解析-自动配置(SecurityFilterAutoConfiguration)](./Spring Security源码解析-自动配置(SecurityFilterAutoConfiguration).md)中解析到DelegatingFilterProxy会做为一个Filter被注入到ServletContext中,所以请求会被DelegatingFilterProxy拦截

---

1.DelegatingFilterProxy#doFilter

```java
Filter delegateToUse = this.delegate;
if (delegateToUse == null) {
  ... 
  WebApplicationContext wac = findWebApplicationContext();
  delegateToUse = initDelegate(wac); //2
  ...
}
invokeDelegate(delegateToUse, request, response, filterChain); ↓
↓
//delegate就是名为springSecurityFilterChain的Bean
//delegate的类型为FilterChainProxy
delegate.doFilter(request, response, filterChain);
↓
↓
doFilterInternal(request, response, chain);
↓
↓
//根据请求获取匹配SecurityFilterChain的Filters
List<Filter> filters = getFilters(fwRequest);
...
VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
vfc.doFilter(fwRequest, fwResponse);
↓
↓
//最后一个Filter
if (currentPosition == size) {
	//继续执行Servlet容器内部的Filter调用链
	originalChain.doFilter(request, response);
}
else {
	currentPosition++;
  //执行下一个Filter
  Filter nextFilter = additionalFilters.get(currentPosition - 1);
  nextFilter.doFilter(request, response, this);
}
```

2.DelegatingFilterProxy#initDelegate

```java
String targetBeanName = getTargetBeanName();
//获取名为springSecurityFilterChain的Bean
Filter delegate = wac.getBean(targetBeanName, Filter.class);
return delegate;
```



