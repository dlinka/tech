Spring4

默认日志依赖

```java
[INFO] +- org.springframework:spring-context:jar:4.3.5.RELEASE:compile
[INFO] |  +- org.springframework:spring-core:jar:4.3.5.RELEASE:compile
[INFO] |  |  \- commons-logging:commons-logging:jar:1.2:compile
```

1.LogFactory#getLog

```java
//org.apache.commons.logging.LogFactory这个类在commons-logging包中
return getFactory().getInstance(clazz);
```

2.LogFactoryImpl#getInstance

```java
//org.apache.commons.logging.impl.LogFactoryImpl
return getInstance(clazz.getName());
↓
↓
Log instance = (Log) instances.get(name);
if (instance == null) {
    //创建Log
    instance = newInstance(name);
    instances.put(name, instance);
}
return instance;
↓
↓
if (logConstructor == null) {
	instance = discoverLogImplementation(name);
}
↓
↓
//按照如下顺序加载,找到就不在找了
//org.apache.commons.logging.impl.Log4JLogger(log4j)
//org.apache.commons.logging.impl.Jdk14Logger(jul)
//org.apache.commons.logging.impl.Jdk13LumberjackLogger
//org.apache.commons.logging.impl.SimpleLog
for(int i=0; i<classesToDiscover.length && result == null; ++i) {
    result = createLogFromClass(classesToDiscover[i], logCategory, true);
}
return result;
```

---

Spring5

日志依赖

```java
[INFO] +- org.springframework:spring-context:jar:5.2.6.RELEASE:compile
[INFO] |  +- org.springframework:spring-core:jar:5.2.6.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-jcl:jar:5.2.6.RELEASE:compile
```

1.LogFactory#getLog

```java
//org.apache.commons.logging.LogFactory这个类在spring-jcl包中
return getLog(clazz.getName());
↓
↓
return LogAdapter.createLog(name);
```

2.LogAdapter#createLog

```java
//根据logApi的值选择初始化对应Log
switch (logApi) {
	case LOG4J:
		return Log4jAdapter.createLog(name);
	case SLF4J_LAL:
		return Slf4jAdapter.createLocationAwareLog(name);
	case SLF4J:
		return Slf4jAdapter.createLog(name);
	default:
		return JavaUtilAdapter.createLog(name);
}
```

3.LogAdapter#static块

```java
static {
	if (isPresent(LOG4J_SPI)) { //log4j 2.x
		if (isPresent(LOG4J_SLF4J_PROVIDER) && isPresent(SLF4J_SPI)) { //log4jv2.x的slf4j桥接
			logApi = LogApi.SLF4J_LAL;
		}
		else {
			logApi = LogApi.LOG4J; //log4j 2.x
		}
	}
	else if (isPresent(SLF4J_SPI)) {
    logApi = LogApi.SLF4J_LAL; //slf4j版本>=1.3
	}
	else if (isPresent(SLF4J_API)) {
		logApi = LogApi.SLF4J; //slf4j版本<1.3
	}
	else {
		//默认使用jul
		logApi = LogApi.JUL;
	}
}
```

