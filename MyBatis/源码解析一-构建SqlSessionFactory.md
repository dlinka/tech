官方示例

    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

1.进入SqlSessionFactoryBuilder的build方法

    return build(inputStream, null, null);
    ↓
    ↓
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    return build(parser.parse());
    ↓
    ↓
    return new DefaultSqlSessionFactory(config);

2.进入上面XMLConfigBuilder的parse方法

    //解析配置文件中的configuration标签
    parseConfiguration(parser.evalNode("/configuration"));
    ↓
    ↓
    //解析配置文件中的mappers标签
    mapperElement(root.evalNode("mappers"));
    ↓
    ↓
    //<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
    //假设配置文件按照上面配置,则会执行如下代码片段
    if(resource != null && url == null && mapperClass == null) {
        InputStream inputStream = Resources.getResourceAsStream(resource);
        XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
        mapperParser.parse();
    }
    ↓
    ↓
    configurationElement(parser.evalNode("/mapper")); //3
    ...
    bindMapperForNamespace(); //4

3.进入XMLMapperBuilder的configurationElement方法

    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    ↓
    ↓
    buildStatementFromContext(list, null);
    ↓
    ↓
    //循环XML中定义的select,insert,update,delete标签
    for (XNode context : list) {
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        statementParser.parseStatementNode();
    }
    ↓
    ↓
    //进入XMLStatementBuilder的parseStatementNode方法
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
    ↓
    ↓
    //最终结果就是生成MappedStatement对象放到configuration对象的mappedStatements属性中(Map<String, MappedStatement> mappedStatements)
    configuration.addMappedStatement(statement);

4.进入XMLMapperBuilder的bindMapperForNamespace方法

    //boundType为Class类型对象
    configuration.addMapper(boundType);
    ↓
    ↓
    //mapperRegistry是configuration的属性
    mapperRegistry.addMapper(type);
    ↓
    ↓
    //knownMappers是一个Map类型
    //key就是接口的Class对象
    knownMappers.put(type, new MapperProxyFactory<T>(type));

---
