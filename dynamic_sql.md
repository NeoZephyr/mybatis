## 动态 `sql`
### `where`
如果标签包含的元素中有返回值，就插入一个 `where`，如果 `where` 后面的字符串时以 `AND` 和 `OR` 开头的，就将它们剔除
```xml
<select id="selectByUser" resultType="com.pain.pojo.User">
    select * from user
    <where>
        <if test="username != null and username != ''">
            and username like '%${username}%'
        </if>
        <if test="email != null and email != ''">
            and email = #{email}
        </if>
    </where>
</select>
```
`choose` 标签实现 `if/else` 逻辑
```xml
<select id="selectByIdOrUsername" resultType="com.pain.pojo.User">
    select * from user
    <where>
        <choose>
            <when test="id != null">
                and id = #{id}
            </when>
            <when test="username != null and username != ''">
                and username = #{username}
            </when>
            <otherwise>
                and 1 = 2
            </otherwise>
        </choose>
    </where>
</select>
```

### `set`
如果该标签包含的元素有返回值，就插入一个 `set`，如果 `set` 后面的字符串是以逗号结尾的，则将逗号剔除
```xml
<update id="updateByIdSelective">
    update user
    <set>
        <if test="username != null and username != ''">
            username = #{username},
        </if>
        <if test="email != null and email != ''">
            email = #{email},
        </if>
        id = #{id},
    </set>
    where id = #{id}
</update>
```

### `trim`
`where` 和 `set` 标签都可以用 `trim` 标签实现
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
</trim>
```
```xml
<trim prefix="SET" suffixOverrides=",">
</trim>
```
`prefix`：当 `trim` 元素内包含内容时，会给内容增加 `prefix` 指定的前缀
`prefixOverrides`：当 `trim` 元素内包含内容时，会把内容中匹配的前缀字符串去掉
`suffix`：当 `trim` 元素内包含内容时，会给内容增加 `suffix` 指定的后缀
`suffixOverrides`：当 `trim` 元素内包含内容时，会把内容中匹配的后缀字符串去掉

### `foreach`
```xml
<select id="selectByIdList" resultType="com.pain.pojo.User">
    select * from user
    where id in
    <foreach collection="list" open="(" close=")" separator="," item="item">
        #{item}
    </foreach>
</select>
```
从 Mybatis3.3.1 开始，Mybatis 开始支持批量新增回写主键值的功能。这个功能要求数据库主键值为自增类型，同时还要求该数据库提供的 JDBC 驱动
可以支持返回批量插入的主键值
```xml
<insert id="batchInsert" useGeneratedKeys="true" keyProperty="id">
    insert into user
    (username, password, email, birth)
    values
    <foreach collection="list" item="user" separator=",">
        (
        #{user.username},
        #{user.password},
        #{user.email},
        #{user.birth}
        )
    </foreach>
</insert>
```
`foreach` 实现动态更新
```xml
<update id="updateByMap">
    update user
    set
    <foreach collection="user" item="val" index="key" separator=",">
        ${key} = #{val}
    </foreach>
    where id = #{user.id}
</update>
```

### `bind`
`bind` 标签可以使用 OGNL 表达式创建一个变量并将其绑定到上下文中。使用 `bind` 不仅可以防止因为更换数据库而修改 SQL，也能预防 `SQL` 注入
```xml
<!--参数名称必须为 value-->
<select id="selectByUsername" parameterType="string" resultType="user">
    select * from user where username like '%${value}%'
</select>
```
```xml
<!--bind 预防 SQL 注入-->
<select id="selectByUsernameCand" parameterType="string" resultType="user">
    <bind name="usernameLike" value="'%' + username + '%'" />
    select * from user where username like #{usernameLike}
</select>
```
