1.spring.factories

```java
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

2.SecurityAutoConfiguration

```java
@EnableConfigurationProperties(SecurityProperties.class)
@Import({ 
	SpringBootWebSecurityConfiguration.class, //3
	WebSecurityEnablerConfiguration.class, //4
	SecurityDataConfiguration.class
})
public class SecurityAutoConfiguration {}
```

3.SpringBootWebSecurityConfiguration

**如果没有自定义配置类,这个类为其提供一个默认的配置类**

**自定义的配置类需要继承WebSecurityConfigurerAdapter**

```java
@ConditionalOnClass(WebSecurityConfigurerAdapter.class)
@ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)
public class SpringBootWebSecurityConfiguration {
	@Configuration(proxyBeanMethods = false)
	static class DefaultConfigurerAdapter extends WebSecurityConfigurerAdapter {}
}
```

4.WebSecurityEnablerConfiguration

```java
@ConditionalOnBean(WebSecurityConfigurerAdapter.class)
@ConditionalOnMissingBean(name = "springSecurityFilterChain)
@EnableWebSecurity //5
public class WebSecurityEnablerConfiguration {}
```

5.EnableWebSecurity

```java
@Import({
  WebSecurityConfiguration.class, //6
	SpringWebMvcImportSelector.class,
	OAuth2ImportSelector.class
})
@EnableGlobalAuthentication //7
public @interface EnableWebSecurity {}
```

6.WebSecurityConfiguration

```java
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {
	//注入名为springSecurityFilterChain的Bean
  @Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
  public Filter springSecurityFilterChain() throws Exception {}
}
```

7.EnableGlobalAuthentication

```java
@Import(AuthenticationConfiguration.class)
public @interface EnableGlobalAuthentication {}
```

8.AuthenticationConfiguration

```java
public class AuthenticationConfiguration {
	@Bean
	public AuthenticationManagerBuilder authenticationManagerBuilder(
    ObjectPostProcessor<Object> objectPostProcessor, ApplicationContext context) {}
}
```

