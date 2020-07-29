1.spring.factories

```java
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration
```

2.SecurityFilterAutoConfiguration

```java
public class SecurityFilterAutoConfiguration {
	@Bean
	public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(SecurityProperties securityProperties) {
		//将springSecurityFilterChain包装成Filter注入ServletContext中
    DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean("springSecurityFilterChain");
		return registration;
	}
}
```

---

DelegatingFilterProxyRegistrationBean的继承关系

![DelegatingFilterProxyRegistrationBean继承关系](https://user-images.githubusercontent.com/4274041/80004215-a26a8d00-84f4-11ea-9bf6-68df919a770b.png)

1.RegistrationBean#onStartup

**DelegatingFilterProxyRegistrationgBean父类RegistrationBean实现了ServletContextInitializer接口**

**容器启动的时候会调用ServletContextInitializer实现类的onStartup方法**

```java
register(description, servletContext);
```

2.DynamicRegistrationBean#register

```java
D registration = addRegistration(description, servletContext);
```

3.AbstractFilterRegistrationBean#addRegistration

```java
Filter filter = getFilter();
//Filter注入到ServletContext中
return servletContext.addFilter(getOrDeduceName(filter), filter);
```

4.DelegatingFilterProxyRegistrationBean#getFilter

```java
//DelegatingFilterProxy是一个Filter
//DelegatingFilterProxy本质是一个代理类,代理名springSecurityFilterChain
return new DelegatingFilterProxy(this.targetBeanName, getWebApplicationContext()) {
		@Override
		protected void initFilterBean() throws ServletException {}
};
```

