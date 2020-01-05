## 基本配置
### 日志
```sh
# 全局配置
log4j.rootLogger=ERROR, stdout

# mybatis 日志配置 
log4j.logger.com.pain.mapper=TRACE

# 控制台输出配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

### `mybatis-config.xml`
根据该配置文件创建 `SqlSessionFactory`

## 增删改查
### `select`
如果使用 resultType 来设置返回结果的类型，需要在 SQL 中为所有列名和属性名不一致的列设置别名
```xml
<select id="selectByIdWithType" parameterType="int" resultType="user">
    select * from user where id = #{id}
</select>
```
```xml
<select id="selectByIdWithMap" parameterType="int" resultMap="UserMap">
    select * from user where id = #{id}
</select>
```

如果查询结果不仅要包含 `user` 信息，还要包含与 `user` 关联的其它部分信息
1. 方法一：设置 resultType 为 userExtend
```java
public class UserExtend extends User {
    // 加上需要包含的其它信息
}
```

2. 方法二：如果要扩展的字段较多，在 `User` 中添加这些字段的包裹对象，`resultType` 还是 `user`
```java
public class User {
    private ExtentWrapper extent;
}
```

当查询接口有多个参数时，添加 `@Param` 注解，Mybatis 回自动将参数封装为 `Map` 类型
```
public List<User> selectByUsernameAndEmail(@Param("username") String username,
                                           @Param("email") String email);
```
```xml
<select id="selectByUsernameAndEmail" parameterType="map" resultType="user">
    select * from user where username = #{username}
    and email = #{email}
</select>
```

### `insert`
`useGeneratedKeys` 默认为 `false`，如果设置为 `true`，会使用 JDBC 的 `getGeneratedKeys` 方法来取出数据库内部生成的主键，赋值给
`keyProperty` 配置的 `id` 属性
```xml
<insert id="insertUser" parameterType="user" useGeneratedKeys="true" keyProperty="id">
    insert into user
    (username, password, email, birth)
    values
    (#{username}, #{password}, #{email}, #{birth})
</insert>
```
上面的回写主键的方法只适用于支持，主键自增的数据库，下面这种则适用于所有数据库
```xml
<!--mysql 类型-->
<insert id="insertUser" parameterType="user">
    insert into user
    (username, password, email, birth)
    values
    (#{username}, #{password}, #{email}, #{birth})
    <selectKey keyColumn="id" keyProperty="id" resultType="int" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
</insert>
```
```xml
<!--oracle 类型-->
<insert id="insertUser" parameterType="user">
    <selectKey keyColumn="id" keyProperty="id" resultType="int" order="BEFORE">
        SELECT SEQ_ID.nextval from dual
    </selectKey>
    insert into user
    (username, password, email, birth)
    values
    (#{username}, #{password}, #{email}, #{birth})
</insert>
```