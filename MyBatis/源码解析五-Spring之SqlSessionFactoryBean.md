官方示例

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>

1.SqlSessionFactoryBean实现了InitializingBean,进入afterPropertiesSet()方法

    this.sqlSessionFactory = buildSqlSessionFactory();
    ↓
    ↓
    //跟之前一样,都是先初始化Configuration对象
    Configuration configuration;
    ...
    configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
    ...
    //跟之前一样,build方法会生成一个DefaultSqlSessionFactory对象做为返回值
    return this.sqlSessionFactoryBuilder.build(configuration);

2.SqlSessionFactoryBean实现了FactoryBean,进入getObject()方法

    //直接返回上一步生成的DefaultSqlSessionFactory
    return this.sqlSessionFactory;

---    
