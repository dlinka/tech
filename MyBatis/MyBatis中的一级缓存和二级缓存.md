一级缓存

    SqlSession级别的缓存

    SqlSession session = getSqlSessionFactory().openSession();
    UserMapper userMapper = session.getMapper(UserMapper.class);
    User user1 = userMapper.selectUserByUserId(1);
    User user2 = userMapper.selectUserByUserId(1);
    System.out.println(user1 == user2);  //true

一级缓存失效

    //SqlSession不同
    SqlSessionFactory sessionFactory = getSqlSessionFactory();
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    UserMapper userMapper1 = session1.getMapper(UserMapper.class);
    UserMapper userMapper2 = session2.getMapper(UserMapper.class);
    User user1 = userMapper1.selectUserByUserId(1);
    User user2 = userMapper2.selectUserByUserId(1);
    System.out.println(user1 == user2);  //false

    //执行增删改操作
    SqlSession session = getSqlSessionFactory().openSession();
    UserMapper userMapper = session.getMapper(UserMapper.class);
    User user1 = userMapper.selectUserByUserId(1);
    userMapper.updateUserNameByUserId(1, "cr");
    User user2 = userMapper.selectUserByUserId(1);
    System.out.println(user1 == user2);  //false

    //调用clearCache()方法
    SqlSession session = getSqlSessionFactory().openSession();
    UserMapper userMapper = session.getMapper(UserMapper.class);
    User user1 = userMapper.selectUserByUserId(1);
    session.clearCache();
    User user2 = userMapper.selectUserByUserId(1);
    System.out.println(user1 == user2);  //false

---

二级缓存

    namespace级别的缓存

    <setting name="cacheEnabled" value="true"/>
    <cache></cache>
    Serializable

    SqlSessionFactory sessionFactory = getSqlSessionFactory();
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    UserMapper userMapper1 = session1.getMapper(UserMapper.class);
    UserMapper userMapper2 = session2.getMapper(UserMapper.class);
    User user1 = userMapper1.selectUserByUserId(1);
    session1.close();
    User user2 = userMapper2.selectUserByUserId(1);
    session2.close();

二级缓存失效

    //数据默认先放在一级缓存中,只有会话提交或者关闭以后,一级缓存中的数据才会转移到二级缓存中
    SqlSessionFactory sessionFactory = getSqlSessionFactory();
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    UserMapper userMapper1 = session1.getMapper(UserMapper.class);
    UserMapper userMapper2 = session2.getMapper(UserMapper.class);
    User user1 = userMapper1.selectUserByUserId(1);
    User user2 = userMapper2.selectUserByUserId(1);
    session1.close();
    session2.close();

---

一级缓存不影响,二级缓存关闭
 
    <setting name="cacheEnabled" value="false"/>

---

一级缓存不影响,不使用二级缓存

    <select id="selectUserByUserId" resultType="User" useCache="false"></select>

---

清空一级缓存,也清空二级缓存

    <select id="selectUserByUserId" resultType="User" flushCache="true"></select>

---

clearCache()方法对二级缓存没有影响

    SqlSessionFactory sessionFactory = getSqlSessionFactory();
    SqlSession session1 = sessionFactory.openSession();
    SqlSession session2 = sessionFactory.openSession();
    UserMapper userMapper1 = session1.getMapper(UserMapper.class);
    UserMapper userMapper2 = session2.getMapper(UserMapper.class);
    User user1 = userMapper1.selectUserByUserId(1);
    session1.close();
    session2.clearCache();
    User user2 = userMapper2.selectUserByUserId(1);
    session2.close();

---
