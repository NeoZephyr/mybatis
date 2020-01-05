## 插件
Mybatis 允许在已映射语句执行过程中的某一点进行拦截调用

### 拦截器
```java
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;
    Object plugin(Object target);
    void setProperties(Properties properties);
}
```
其中，`setProperties` 方法可以用来传递插件的参数
```xml
<plugins>
    <plugin interceptor="com.pain.plugin.LogInterceptor">
        <property name="" value="" />
        <property name="" value="" />
    </plugin>
</plugins>
```
`plugin` 方法，参数 `target` 就是拦截器要拦截的对象，该方法在创建被拦截的接口实现类时被调用。这个接口通常实现代码如下：
```
public Object plugin(Object target) {
   return Plugin.wrap(target, this); 
}
```
`Plugin.wrap` 方法会自动判断拦截器的签名和被拦截对象的接口是否匹配，只有匹配的情况下才会使用动态代理拦截目标对象
`intercept` 方法是 Mybatis 运行时要执行的拦截方法
```
public Object intercept(Invocation invocation) throws Throwable {

    // 获取当前被拦截的对象
    Object target = invocation.getTarget();
    
    // 获取当前被拦截的方法
    Method method = invocation.getMethod();
    
    // 获取被拦截方法中的参数
    Object[] args = invocation.getArgs();
    
    // 相当于执行 method.invoke(target, args)
    Object result = invocation.proceed();
    
    return result;
}
```
如果配置多个拦截器，Mybatis 会遍历所有拦截器，按顺序执行拦截器的 `plugin` 方法，被拦截的对象就会被层层代理

#### 拦截器签名
`@Intercepts(org.apache.ibatis.plugin.Intercepts)` 和签名注解 `@Signature(org.apache.ibatis.plugin.Signature)` 用来配置拦截
器要拦截的接口中的方法。其中，`@Intercepts` 注解中的属性是一个 `@Signature` 数组，可以在同一个拦截器中同时拦截不同的接口和方法

### `Executor`
在所有 `INSERT`, `UPDATE`, `DELETE` 执行时被调用
```
int update(MappedStatement ms, Object parameter) throws SQLException
```
在所有 `SELECT` 查询方法执行时被调用
```
<E> list<E> query(MappedStatement ms,
                    Object parameter,
                    RowBounds rowBounds,
                    ResultHandler resultHandler) throws SQLException
```

### `ParameterHandler`
在所有数据库方法设置 SQL 参数时调用
```
void setParameters(PreparedStatement ps) throws SQLException
```

### `ResultSetHandler`

### `StatementHandler`

### 插件示例
```java
@Intercepts(
        @Signature(
                type = ResultSetHandler.class,
                method = "handleResultSets",
                args = {Statement.class}
        )
)
public class CameInterceptor implements Interceptor {
    public Object intercept(Invocation invocation) throws Throwable {
        List<Object> list = (List<Object>) invocation.proceed();
        // process
        return list;
    }
    
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }
    
    public void setProperties(Properties properties) {}
}
```
使用插件
```xml
<plugins>
    <plugin interceptor="com.pain.interceptor.CameInterceptor" />
</plugins>
```