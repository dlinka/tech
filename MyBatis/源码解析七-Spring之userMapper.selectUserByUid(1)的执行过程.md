1.userMapper是一个代理对象,执行方法会进入MapperProxy的invoke方法

    //此处跟之前唯一的不同就是sqlSession的实现类,这里是SqlSessionTemplate
    return mapperMethod.execute(sqlSession, args);
    ↓
    ↓
    result = sqlSession.selectOne(command.getName(), param);
    ↓
    ↓
    //sqlSessionProxy是一个代理对象
    //执行sqlSessionProxy任何方法都会进入SqlSessionInterceptor的invoke方法
    return this.sqlSessionProxy.<T> selectOne(statement, parameter);
    ↓
    ↓
    SqlSession sqlSession = getSqlSession(
          SqlSessionTemplate.this.sqlSessionFactory,
          SqlSessionTemplate.this.executorType,
          SqlSessionTemplate.this.exceptionTranslator);
    ...
    Object result = method.invoke(sqlSession, args);
    ↓
    ↓
    //这行代码很熟悉吧!!!
    //就是源码解析二中通过sessionFactory获取SqlSession的逻辑
    //之后的执行可以参考之前的源码解析系列
    session = sessionFactory.openSession(executorType);

---
