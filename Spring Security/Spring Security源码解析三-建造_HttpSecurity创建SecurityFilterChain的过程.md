##### HttpSecurity继承关系

<img src="https://user-images.githubusercontent.com/4274041/80091072-178c9f80-8593-11ea-9aa1-a19e9270cee9.png" alt="image" style="zoom:50%;" />

1.WebSecurity#performBuild

```java
for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : securityFilterChainBuilders) {
	securityFilterChains.add(securityFilterChainBuilder.build());
}
↓
↓
//AbstractSecurityBuilder#build
this.object = doBuild();
↓
↓
//一样的模板方法
synchronized (configurers) {
	buildState = BuildState.INITIALIZING;
	beforeInit();
	init();
	buildState = BuildState.CONFIGURING;
	beforeConfigure();
	configure(); //2
	buildState = BuildState.BUILDING;
	O result = performBuild(); //3
	buildState = BuildState.BUILT;
	return result;
}
```

##### FormLoginConfigurer继承关系

![image](https://user-images.githubusercontent.com/4274041/104695044-131e9d00-5747-11eb-8bca-e9a7791cad9e.png)

2.AbstractConfiguredSecurityBuilder#configure

```java
configurer.configure((B) this);
↓
↓
//以FormLoginConfigurer为例
//这里进入AbstractAuthenticationFilterConfigurer#configure
//UsernamePasswordAuthenticationFilter放到HttpSecurity的filters属性中
http.addFilter(filter);
```

3.HttpSecurity#performBuild

```java
//排序filters
filters.sort(comparator);
//构建SecurityFilterChain
return new DefaultSecurityFilterChain(requestMatcher, filters);
```

