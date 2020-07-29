官方示例

    UserMapper userMapper = session.getMapper(UserMapper.class);

1.进入DefaultSqlSession的getMapper方法

    return configuration.<T>getMapper(type, this);
    ↓
    ↓
    //这里使用到的mapperRegistry属性就是MyBatis源码解析一里介绍的
    //type就是接口的Class类型对象
    return mapperRegistry.getMapper(type, sqlSession);
    ↓
    ↓
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    //这步就是整个MyBatis的核心!!!
    return mapperProxyFactory.newInstance(sqlSession);
    ↓
    ↓
    //MapperProxy实现了动态代理中的InvocationHandler接口
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
    ↓
    ↓
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);

---
