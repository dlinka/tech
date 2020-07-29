官方示例

    SqlSession session = sessionFactory.openSession();
    
1.进入DefaultSqlSessionFactory的openSession方法

    //默认configuration.getDefaultExecutorType()返回值为ExecutorType.SIMPLE
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
    ↓
    ↓
    //获取environment对象的transactionFactory属性
    //默认返回值是JdbcTransactionFactory
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    //new JdbcTransaction(ds, level, autoCommit);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    //new SimpleExecutor(this, transaction);
    final Executor executor = configuration.newExecutor(tx, execType);
    //SqlSession实现类是DefaultSqlSession
    return new DefaultSqlSession(configuration, executor, autoCommit);

---
