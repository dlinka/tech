1.WebSecurityConfigurerAdapter#init

```java
final HttpSecurity http = getHttp();
↓
↓
http = new HttpSecurity(objectPostProcessor, authenticationBuilder, sharedObjects);
//一般都会使用默认配置
if (!disableDefaults) {
  http
	.csrf().and()
	.addFilter(new WebAsyncManagerIntegrationFilter())
	.exceptionHandling().and()
	.headers().and()
	.sessionManagement().and()
	.securityContext().and()
	.requestCache().and()
	.anonymous().and()
	.servletApi().and()
	.apply(new DefaultLoginPageConfigurer<>()).and()
	.logout();
}
//这里一般都会调用自定义WebSecurityConfigurerAdapter类的configure方法
configure(http); //2
```

2.WebSecurityConfigurerAdapter#configure

```java
http
	.authorizeRequests()
		.anyRequest().authenticated()
		.and()
	.formLogin().and()
	.httpBasic();
↓
↓
↓
↓
//.formLogin()为例
//HttpSecurity#formLogin
return getOrApply(new FormLoginConfigurer<>()); //3
↓
↓
return apply(configurer);
↓
↓
add(configurer);
↓
↓
this.configurers.put(clazz, configs);
```

3.FormLoginConfigurer构造方法

```java
super(new UsernamePasswordAuthenticationFilter(), null);
↓
↓
this.authFilter = authenticationFilter;
```

