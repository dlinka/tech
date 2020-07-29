#####FilterSecurityInterceptor继承关系<img src="https://user-images.githubusercontent.com/4274041/105690532-167c0a80-5f37-11eb-8c07-790068065f99.png" alt="image" style="zoom:50%;" />

1.FilterSecurityInterceptor#doFilter

```java
FilterInvocation fi = new FilterInvocation(request, response, chain);
invoke(fi);
↓
↓
InterceptorStatusToken token = super.beforeInvocation(fi);
↓
↓
this.accessDecisionManager.decide(authenticated, object, attributes);
↓
↓
//AffirmativeBased#decide
int result = voter.vote(authentication, object, configAttributes);
↓
↓
//WebExpressionVoter#vote
//这里剩下就是SPEL的东西
return ExpressionUtils.evaluateAsBoolean(weca.getAuthorizeExpression(), ctx) ? ACCESS_GRANTED : ACCESS_DENIED;
```

2.SecurityExpressionRoot

```java
//SPEL表达式根据初始化的字符串"authenticated"会反射调用isAuthenticated方法
//跟踪下DEBUG调用栈就知道怎么回事了
//ReflectivePropertyAccessor#findGetterForProperty这个方法先查找getAuthenticated,没有找到再查找isAuthenticated方法
return !isAnonymous();
↓
↓
return trustResolver.isAnonymous(authentication);
↓
↓
//anonymousClass实现类是AnonymousAuthenticationToken
//用户登录的时候authentication实现类是UsernamePasswordAuthenticationToken
//这里返回false,上面第1步最后就返回true,代表已经验证的用户(ACCESS_GRANTED)
return anonymousClass.isAssignableFrom(authentication.getClass());
```

