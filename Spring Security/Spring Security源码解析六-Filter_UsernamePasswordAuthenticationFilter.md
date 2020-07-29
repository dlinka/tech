1.FormLoginConfigurer#configure(这里调用父类AbstractAuthenticationFilterConfigurer#configure)

```java
//将AuthenticationManager注入到Filter
authFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));
...
F filter = postProcess(authFilter);
http.addFilter(filter);
```

2.UsernamePasswordAuthenticationFilter#attemptAuthentication

```java
//首先进入父类AbstractAuthenticationProcessingFilter#doFilter
authResult = attemptAuthentication(request, response);
↓
↓
//这时候进入子类UsernamePasswordAuthenticationFilter#attemptAuthentication
String username = obtainUsername(request);
String password = obtainPassword(request);
UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
return this.getAuthenticationManager().authenticate(authRequest);
↓
↓
//ProviderManager#authenticate
for (AuthenticationProvider provider : getProviders()) {
	result = provider.authenticate(authentication);
}
//进入父AuthenticationManager
if (result == null && parent != null) {
	result = parentResult = parent.authenticate(authentication);
}
↓
↓
//父AuthenticationManager一样的逻辑
//这里直接进入前面构造的DaoAuthenticationProvider的父类AbstractUserDetailsAuthenticationProvider#authenticate
user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication); //3
...
additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication); //4
```

3.DaoAuthenticationProvider#retrieveUser

```java
UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
```

4.DaoAuthenticationProvider#additionalAuthenticationChecks

```java
//获取传入进来的password
String presentedPassword = authentication.getCredentials().toString();
//passwordEncoder的初始化在DaoAuthenticationProvider的构造函数内
//默认的校验就是判断是否相等使用NoOpPasswordEncoder#matches
if (!passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
	//如果不匹配抛出异常
  throw new BadCredentialsException(messages.getMessage(
					"AbstractUserDetailsAuthenticationProvider.badCredentials",
					"Bad credentials"));
}
```



