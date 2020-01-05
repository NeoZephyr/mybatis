## 拦截器插件
### 配置
在 Spring 配置文件中配置
```xml
<property name="plugins">
    <array>
        <bean class="com.github.pagehelper.PageHelper">
            <property name="properties">
                <value>
                    dialect=mysql
                </value>
            </property>
        </bean>
    </array>
</property>
```

### 原理
MyBatis 的 plugin 实现了 `Interceptor` 接口，
PageHelper 使用 `ThreadLocal` 获取到同一线程中的变量信息。PageHelper 通过拦截器获取到同一线程中的预编译好的 SQL 语句之后将 SQL 语句包装成具有分页功能的 SQL 语句，并将其再次赋值给下一步操作，所以实际执行的 SQL 语句就是有了分页功能

Mybatis 定义了一个插件接口 `org.apache.ibatis.plugin.Interceptor`
```java
public interface Interceptor {

  Object intercept(Invocation var1) throws Throwable;
  
  // 生成动态代理对象
  Object plugin(Object var1);
  
  // 使用插件时设置参数值
  void setProperties(Properties var1);
}
```

```java
public Object intercept(Invocation invocation) throws Throwable {
  if (this.autoRuntimeDialect) {
    SqlUtil sqlUtil = this.getSqlUtil(invocation);
    return sqlUtil.processPage(invocation);
  } else {
    if (this.autoDialect) {
      this.initSqlUtil(invocation);
    }

    return this.sqlUtil.processPage(invocation);
  }
}

public Object processPage(Invocation invocation) throws Throwable {
  try {
    Object result = _processPage(invocation);
    return result;
  } finally {
    clearLocalPage();
  }
}

private Object _processPage(Invocation invocation) throws Throwable {
  Object[] args = invocation.getArgs();
  Page page = null;
  if(this.supportMethodsArguments) {
    page = this.getPage(args);
  }

  RowBounds rowBounds = (RowBounds)args[2];
  if(this.supportMethodsArguments && page == null || !this.supportMethodsArguments && getLocalPage() == null && rowBounds == RowBounds.DEFAULT) {
    return invocation.proceed();
  } else {
    if(!this.supportMethodsArguments && page == null) {
      page = this.getPage(args);
    }

    return this.doProcessPage(invocation, page, args);
  }
}
```
```java
// 对 Executor 进行拦截
public Object plugin(Object target) {
  return target instanceof Executor?Plugin.wrap(target, this):target;
}
```
```java
public void setProperties(Properties p) {
  this.checkVersion();
  String dialect = p.getProperty("dialect");
  String runtimeDialect = p.getProperty("autoRuntimeDialect");
  if(StringUtil.isNotEmpty(runtimeDialect) && runtimeDialect.equalsIgnoreCase("TRUE")) {
    this.autoRuntimeDialect = true;
    this.autoDialect = false;
    this.properties = p;
  } else if(StringUtil.isEmpty(dialect)) {
    this.autoDialect = true;
    this.properties = p;
  } else {
    this.autoDialect = false;
    this.sqlUtil = new SqlUtil(dialect);
    this.sqlUtil.setProperties(p);
  }
}
```

### 分页插件使用
```java
// 获取第1页，10 条内容，默认查询总数 count
// 设置分页信息保存到 threadlocal 中
PageHelper.startPage(1, 10);

// 紧跟着的第一个 select 方法会被分页，contryMapper 会被 PageInterceptor 截拦,截拦器会从 threadlocal 中取出分页信息，把分页信息加到 sql 语句中，实现了分页查旬
List<Country> list = countryMapper.selectIf(1);

// 分页时，实际返回的结果 list 类型是 Page<E>，如果想取出分页信息，需要强制转换为 Page<E>
Page page = (Page)list;
page.getPageNum();
page.getPageSize();
page.getPages();
page.getTotal();
page.size();
```