官方示例

    <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>

1.首先进入MapperFactoryBean的父类SqlSessionDaoSupport的setSqlSessionFactory方法


    //SqlSession在前面的源码解析中实现类是DefaultSqlSession
    //这里SqlSession的实现类是SqlSessionTemplate
    this.sqlSession = new SqlSessionTemplate(sqlSessionFactory);
    ↓
    ↓
    this(sqlSessionFactory, sqlSessionFactory.getConfiguration().getDefaultExecutorType());
    ↓
    ↓
    this(sqlSessionFactory, executorType,
        new MyBatisExceptionTranslator(
            sqlSessionFactory.getConfiguration().getEnvironment().getDataSource(), true));
    ↓
    ↓
    this.sqlSessionFactory = sqlSessionFactory;
    ...
    this.sqlSessionProxy = (SqlSession) newProxyInstance(
        SqlSessionFactory.class.getClassLoader(),
        new Class[] { SqlSession.class },
        new SqlSessionInterceptor());

2.MapperFactoryBean实现了FactoryBean接口,进入getObject()方法

    //getSqlSession()方法返回上一步创建的SqlSessionTemplate对象
    return getSqlSession().getMapper(this.mapperInterface);
    ↓
    ↓
    //进入SqlSessionTemplate的getMapper()方法
    //以前生成的MapperProxy传入的是DefaultSqlSession
    //现在生成的MapperProxy传入的是SqlSessionTemplate
    return getConfiguration().getMapper(type, this);

---

关于SqlSessionTemplate的官方API的一段描述,这个对象可以单例存在

![image](https://user-images.githubusercontent.com/4274041/81561452-c7507280-93c5-11ea-8aa5-4f8e7c9b26bf.png)

---
