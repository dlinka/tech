##### WebSecurity的继承关系
<img src="https://user-images.githubusercontent.com/4274041/80091046-0b084700-8593-11ea-874e-5188b905af86.png" alt="image" style="zoom:50%;" />

1.WebSecurityConfiguration#springSecurityFilterChain

```java
return webSecurity.build();
↓
↓
//AbstractSecurityBuilder#build
this.object = doBuild();
↓
↓
//AbstractConfiguredSecurityBuilder#doBuild
synchronized (configurers) {
	buildState = BuildState.INITIALIZING;
	beforeInit();
	init(); //2
	buildState = BuildState.CONFIGURING;
	beforeConfigure();
	configure();
	buildState = BuildState.BUILDING;
	O result = performBuild(); //3
	buildState = BuildState.BUILT;
	return result;
}
```

2.AbstractConfiguredSecurityBuilder#init

```java
for (SecurityConfigurer<O, B> configurer : configurers) {
	configurer.init((B) this);
}
↓
↓
//WebSecurityConfigurerAdapter#init
//创建HttpSecurity
final HttpSecurity http = getHttp();
//把HttpSecurity放入WebSecurity的securityFilterChainBuilders属性中
web.addSecurityFilterChainBuilder(http).postBuildAction(() -> {
});
```

3.WebSecurity#performBuild

```java
int chainSize = ignoredRequests.size() + securityFilterChainBuilders.size();
List<SecurityFilterChain> securityFilterChains = new ArrayList<>(chainSize);
...
for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : securityFilterChainBuilders) {
	//HttpSecurity构建SecurityFilterChain
  securityFilterChains.add(securityFilterChainBuilder.build());
}
FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
...
Filter result = filterChainProxy;
...
return result;
```

