1.WebSecurityConfiguration#setFilterChainProxySecurityConfigurer

```java
@Autowired(required = false)
public void setFilterChainProxySecurityConfigurer(
	ObjectPostProcessor<Object> objectPostProcessor,
	@Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}")
		List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers) //2
) {
	//objectPostProcessor实现类为AutowireBeanFactoryObjectPostProcessor
  //AutowireBeanFactoryObjectPostProcessor的作用是把对象放到Spring容器中
	webSecurity = objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));
  ...
	//这里把自定义的WebSecurityConfigurerAdapter放到WebSecurity中
	for (SecurityConfigurer<Filter, WebSecurity> webSecurityConfigurer : webSecurityConfigurers) {
		webSecurity.apply(webSecurityConfigurer);
	}
}
```

2.AutowiredWebSecurityConfigurersIgnoreParents#getWebSecurityConfigurers

```java
List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers = new ArrayList<>();
//从Spring容器中获取自定义的WebSecurityConfigurerAdapter
Map<String, WebSecurityConfigurer> beansOfType = beanFactory.getBeansOfType(WebSecurityConfigurer.class);
for (Entry<String, WebSecurityConfigurer> entry : beansOfType.entrySet()) {
	webSecurityConfigurers.add(entry.getValue());
}
return webSecurityConfigurers;
```