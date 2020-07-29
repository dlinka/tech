官方示例

    <plugins>
        <plugin interceptor="org.mybatis.example.ExamplePlugin">
            <property name="someProperty" value="100"/>
        </plugin>
    </plugins>
    
    @Intercepts({
            @Signature(
                type= Executor.class,
                method = "update",
                args = {MappedStatement.class, Object.class})
    })
    public class ExamplePlugin implements Interceptor {
        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            // implement pre-processing if needed
            Object returnObject = invocation.proceed();
            // implement post-processing if needed
            return returnObject;
        }
        
        @Override
        public Object plugin(Object target) {
            return Plugin.warp(target, this);
        }
    }

1.进入XMLConfigBuilder的parseConfiguration方法,这步是解析标签

    //解析plugins标签
    pluginElement(root.evalNode("plugins"));
    ↓
    ↓
    //循环每一个plugin标签
    for (XNode child : parent.getChildren()) {
        //按照示例拿到"org.mybatis.example.ExamplePlugin"
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        //底层就是将拦截器对象添加到一个ArrayList集合中
        configuration.addInterceptor(interceptorInstance);
    }
    
2.进入InterceptorChain的pluginAll方法,下面插件场景中看具体哪里会调用的这个方法

    //遍历拦截器
    for (Interceptor interceptor : interceptors) {
      //执行拦截器的plugin方法
      target = interceptor.plugin(target);
    }
    return target;
    
3.进入Interceptor的plugin方法

    return Plugin.wrap(target, this);
    ↓
    ↓
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor); //4
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap)); //5
    }

4.进入getSignatureMap方法
    
    //获取当前拦截器的@Intercepts注解
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    //获取所有Signature
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    for (Signature sig : sigs) {
        Set<Method> methods = signatureMap.get(sig.type());
        if (methods == null) {
            methods = new HashSet<Method>();
            //key为Executor.class
            signatureMap.put(sig.type(), methods);
        }
        Method method = sig.type().getMethod(sig.method(), sig.args());
        //把Executor的update方法放到methods集合中
        methods.add(method);
    }
    return signatureMap;

5.进入invoke方法

    Set<Method> methods = signatureMap.get(method.getDeclaringClass());
    if (methods != null && methods.contains(method)) {
        //如果注解的中方法包含当前方法,进入拦截器的intercept方法
        return interceptor.intercept(new Invocation(target, method, args));
    }

---

插件场景

    //初始化Executor
    executor = (Executor) interceptorChain.pluginAll(executor);

    //初始化StatementHandler
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);

    //初始化ParameterHandler
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    
    //初始化ResultSetHandler
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    
---
