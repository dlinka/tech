##### HttpSecurity中的configurers

<img src="https://user-images.githubusercontent.com/4274041/80299613-548da780-87c8-11ea-8d0e-c6e1f07f19a9.png" alt="image" style="zoom:50%;"/>

1.AnonymousConfigurer#init

```java
if (authenticationProvider == null) {
  //getKey返回UUID
	authenticationProvider = new AnonymousAuthenticationProvider(getKey());
}
http.authenticationProvider(authenticationProvider);
↓
↓
//AuthenticationManagerBuilder的authenticationProviders集合中加入AnonymousAuthenticationProvider
getAuthenticationRegistry().authenticationProvider(authenticationProvider);
```

2.HttpSecurity#beforeConfigure

```java
setSharedObject(AuthenticationManager.class, getAuthenticationRegistry().build());
↓
↓
//DefaultPasswordEncoderAuthenticationManagerBuilder的父类AuthenticationManagerBuilder#performBuild
ProviderManager providerManager = new ProviderManager(authenticationProviders, parentAuthenticationManager);
return providerManager;
```

