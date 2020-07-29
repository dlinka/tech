1.userMapper是一个代理对象,所以进入MapperProxy的invoke方法

    //new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
    ↓
    ↓
    case SELECT:
        ...
        else {
            //解析参数
            Object param = method.convertArgsToSqlCommandParam(args);
            //command.getName()返回"com.cr.dao.UserDao.selectUserByUid"
            result = sqlSession.selectOne(command.getName(), param);
        }
    ↓
    ↓
    //进入DefaultSqlSession的selectOne方法
    List<T> list = this.<T>selectList(statement, parameter);
    ↓
    ↓
    return this.selectList(statement, parameter, RowBounds.DEFAULT);
    ↓
    ↓
    //这里使用到的mappedStatements属性就是MyBatis源码解析一里介绍的
    MappedStatement ms = configuration.getMappedStatement(statement);
    //这一步的executor就是构建SqlSession中的executor
    //executor的实现类是CachingExecutor
    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    ↓
    ↓
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    ↓
    ↓
    //delegate的实现类是SimpleExecutor
    //这里会进入其父类BaseExecutor的query方法
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    ↓
    ↓
    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    ↓
    ↓
    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    ↓
    ↓
    //这里进入SimpleExecutor的doQuery方法
    //handler实现类为RoutingStatementHandler,内部代理了PreparedStatementHandler对象
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    stmt = prepareStatement(handler, ms.getStatementLog()); //2
    return handler.<E>query(stmt, resultHandler);
    ↓
    ↓
    return delegate.<E>query(statement, resultHandler);
    ↓
    ↓
    //JDBC登场了!!!
    //进入PreparedStatementHandler的query方法
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();

2.进入SimpleExecutor的prepareStatement方法

    Statement stmt;
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout()); //3
    ↓
    ↓
    //transaction实现类是JdbcTransaction
    Connection connection = transaction.getConnection();
    ↓
    ↓
    openConnection();
    ↓
    ↓
    //数据源获取连接
    connection = dataSource.getConnection();

3.进入RoutingStatementHandler的prepare方法

    //delegate实现类是PreparedStatementHandler
    return delegate.prepare(connection, transactionTimeout);
    ↓
    ↓
    //进入父类BaseStatementHandler的prepare方法
    statement = instantiateStatement(connection);
    ↓
    ↓
    //进入PreparedStatementHandler的instantiateStatement方法
    return connection.prepareStatement(sql);

---
