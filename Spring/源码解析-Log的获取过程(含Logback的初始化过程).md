maven

```xml
<dependency>
	<groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.16.18</version>
</dependency>
```

---

1.LogFactory#getLog(spring-jcl.jar)

```java
return getLog(clazz.getName());
↓
↓
return LogAdapter.createLog(name);
↓
↓
case SLF4J_LAL:
	return Slf4jAdapter.createLocationAwareLog(name);
↓
↓
Logger logger = LoggerFactory.getLogger(name);
```

2.LoggerFactory#getLogger(slf4j-api.jar)

```java
ILoggerFactory iLoggerFactory = getILoggerFactory();
return iLoggerFactory.getLogger(name); //4
↓
↓
//getILoggerFactory
//初始化
if (INITIALIZATION_STATE == UNINITIALIZED) {
	performInitialization();
}
↓
↓
bind();
↓
↓
StaticLoggerBinder.getSingleton(); //3
```

3.StaticLoggerBinder的静态初始化块(logback-classic.jar)

```java
//静态初始化块
static {
	SINGLETON.init();
}
↓
↓
new ContextInitializer(defaultLoggerContext).autoConfig();
↓
↓
//autoConfig
URL url = findURLOfDefaultConfigurationFile(true);
if (url != null) {
	//根据配置文件去配置
  configureByResource(url);
} else {
  ...
  else{
    //初始化一个默认配置
    BasicConfigurator basicConfigurator = new BasicConfigurator();
    basicConfigurator.setContext(loggerContext);
    basicConfigurator.configure(loggerContext);
  }
}
↓
↓
//findURLOfDefaultConfigurationFile
url = getResource("logback-test.xml", myClassLoader, updateStatus);
url = getResource("logback.groovy", myClassLoader, updateStatus);
return getResource("logback.xml", myClassLoader, updateStatus);
```

4.LoggerContext#getLogger(logback-classic.jar)

```java
if (Logger.ROOT_LOGGER_NAME.equalsIgnoreCase(name)) {
	return root;
}
//从缓存中获取
Logger childLogger = (Logger) loggerCache.get(name);
```





