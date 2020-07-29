##### DefaultPasswordEncoderAuthenticationManagerBuilder的继承关系<img src="https://user-images.githubusercontent.com/4274041/104911048-39517080-59c5-11eb-8a63-ab8715d78fe5.png" alt="image" style="zoom:50%;"/>

1.WebSecurityConfigurerAdapter#setApplicationContext

```java
authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(objectPostProcessor, passwordEncoder);
```

2.WebSecurityConfigurerAdapter#getHttp

```java
//这里构建parentAuthenticationManager属性
AuthenticationManager authenticationManager = authenticationManager(); //3
authenticationBuilder.parentAuthenticationManager(authenticationManager);
http = new HttpSecurity(objectPostProcessor, authenticationBuilder, sharedObjects);
```

3.WebSecurityConfigurerAdapter#authenticationManager

```java
if (!authenticationManagerInitialized) {
  //如果没有重写configure,这个方法会把disableLocalConfigureAuthenticationBldr变为true
  configure(localConfigureAuthenticationBldr);
	if (disableLocalConfigureAuthenticationBldr) {
		authenticationManager = authenticationConfiguration.getAuthenticationManager();
	}
}
↓
↓
//authenticationManagerBuilder方法会生成一个AuthenticationManagerBuilder
AuthenticationManagerBuilder authBuilder = this.applicationContext.getBean(AuthenticationManagerBuilder.class);
//enableGlobalAuthenticationAutowiredConfigurer
//initializeUserDetailsBeanManagerConfigurer
//initializeAuthenticationProviderBeanManagerConfigurer
//上面三个方法会注入三个相关类型的Bean到globalAuthConfigurers属性中
for (GlobalAuthenticationConfigurerAdapter config : globalAuthConfigurers) {
	authBuilder.apply(config);
}
//构建parentAuthenticationManager
authenticationManager = authBuilder.build();
```

4.InitializeUserDetailsBeanManagerConfigurer#init

```java
auth.apply(new InitializeUserDetailsManagerConfigurer());
```

5.InitializeUserDetailsManagerConfigurer#configure

```java
//UserDetailsServiceAutoConfiguration#inMemoryUserDetailsManager方法会生成一个UserDetailsService
UserDetailsService userDetailsService = getBeanOrNull(UserDetailsService.class);
DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
provider.setUserDetailsService(userDetailsService);
//加入到AuthenticationManagerBuilder的authenticationProviders属性
auth.authenticationProvider(provider);
```

6.AuthenticationManagerBuilder#performBuild

```java
ProviderManager providerManager = new ProviderManager(authenticationProviders, parentAuthenticationManager);
return providerManager;
```