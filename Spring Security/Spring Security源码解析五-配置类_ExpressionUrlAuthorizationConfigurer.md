WebSecurityConfigurerAdapter

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
	//Specify that URLs are allowed by any authenticated user.
  //只要登录了,访问任何URL都不需要验证
  http
    .authorizeRequests() //1
    	.anyRequest() //2
     		.authenticated(); //3
}
```

##### 初始化Configurer

1.进入authorizeRequests()

```java
return getOrApply(new ExpressionUrlAuthorizationConfigurer<>(context)).getRegistry();
↓
↓
this.REGISTRY = new ExpressionInterceptUrlRegistry(context);
```

<img src="https://user-images.githubusercontent.com/4274041/105671209-4c13fa00-5f1d-11eb-9f54-51842d7cf700.png" alt="image" style="zoom:50%;" />

2.进入anyRequest()

```java
//进入AbstractRequestMatcherRegistry
//ANY_REQUEST的实现类是AnyRequestMatcher,matches方法直接返回true
C configurer = requestMatchers(ANY_REQUEST);
return configurer;
↓
↓
return chainRequestMatchers(Arrays.asList(requestMatchers));
↓
↓
//进入AbstractConfigAttributeRequestMatcherRegistry
return chainRequestMatchersInternal(requestMatchers);
↓
↓
return new AuthorizedUrl(requestMatchers);
```

3.进入authenticated()

```java
//进入AuthorizedUrl
return access(authenticated);
↓
↓
//createList方法会生成List<SecurityConfig>对象
interceptUrl(requestMatchers, SecurityConfig.createList(attribute));
return ExpressionUrlAuthorizationConfigurer.this.REGISTRY;
↓
↓
//进入ExpressionUrlAuthorizationConfigurer#interceptUrl
for (RequestMatcher requestMatcher : requestMatchers) {
	REGISTRY.addMapping(new AbstractConfigAttributeRequestMatcherRegistry.UrlMapping(requestMatcher, configAttributes));
}
```

---

##### 初始化Filter

1.ExpressionUrlAuthorizationConfigurer的父类AbstractInterceptUrlConfigurer#configure

```java
FilterInvocationSecurityMetadataSource metadataSource = createMetadataSource(http);
FilterSecurityInterceptor securityInterceptor = createFilterSecurityInterceptor(http, metadataSource, http.getSharedObject(AuthenticationManager.class));
```

2.ExpressionUrlAuthorizationConfigurer#createMetadataSource

```java
LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> requestMap = REGISTRY.createRequestMap();
return new ExpressionBasedFilterInvocationSecurityMetadataSource(requestMap, getExpressionHandler(http));
↓
↓
super(processMap(requestMap, expressionHandler.getExpressionParser()));
↓
↓
RequestMatcher request = entry.getKey();
ArrayList<ConfigAttribute> attributes = new ArrayList<>(1);
//return new RequestVariablesExtractorEvaluationContextPostProcessor(request);
AbstractVariableEvaluationContextPostProcessor postProcessor = createPostProcessor(request);
attributes.add(new WebExpressionConfigAttribute(parser.parseExpression(expression), postProcessor));
requestToExpressionAttributesMap.put(request, attributes);
```

![image](https://user-images.githubusercontent.com/4274041/105717654-5bb23380-5f5b-11eb-9c7d-299ff2fe9fb0.png)

3.ExpressionUrlAuthorizationConfigurer#getExpressionHandler

```java
DefaultWebSecurityExpressionHandler defaultHandler = new DefaultWebSecurityExpressionHandler();
//AuthenticationTrustResolver实现类为AuthenticationTrustResolverImpl
AuthenticationTrustResolver trustResolver = http.getSharedObject(AuthenticationTrustResolver.class);
if (trustResolver != null) {
	defaultHandler.setTrustResolver(trustResolver);
}
```

---

