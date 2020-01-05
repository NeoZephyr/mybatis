## 高级结果映射

### 一对一映射
```java
public class User {
    private Role role;
}
```

#### 直接使用 SQL 语句查询
```xml
<select id="selectUserWithRoleByUserId" resultType="user">
    select
      u.id,
      u.username,
      u.password,
      u.email,
      u.birth,
      
      r.id "role.id",
      r.name "role.name"
      from user u
      inner join user_role ur on u.id = ur.user_id
      inner join role r on r.id = ur.role_id
      where u.id = #{userId}
</select>
```

#### `resultMap` 配置一对一映射
为避免不同表中存在相同的列，所有可能重名的列都增加了 "role_" 前缀
```xml
<resultMap id="userRoleMap" type="com.pain.pojo.User">
    <id column="id" property="id" />
    <result column="username" property="username" />
    <result column="password" property="password" />
    <result column="email" property="email" />
    <result column="birth" property="birth" />

    <result column="role_id" property="role.id" />
    <result column="role_name" property="role.name" />
</resultMap>
```
```xml
<select id="selectUserWithRoleByUserId" resultMap="userRoleMap">
    select
      u.id,
      u.username,
      u.password,
      u.email,
      u.birth,
      
      r.id role_id,
      r.name role_name
      from user u
      inner join user_role ur on u.id = ur.user_id
      inner join role r on r.id = ur.role_id
      where u.id = #{userId}
</select>
```

利用 `resultMap` 映射继承，继承 `userMap`，添加 `role` 特有配置
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <result column="role_id" property="role.id" />
    <result column="role_name" property="role.name" />
</resultMap>
```

利用 `association` 标签进一步简化配置，配置 `columnPrefix` 之后可以在子标签 `result` 中的 `column` 省略前缀
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <association property="role" columnPrefix="role_" javaType="role">
        <result column="id" property="id" />
        <result column="name" property="name" />    
    </association>

</resultMap>
```

利用已存在的 `roleMap` 映射简化 `association` 标签配置
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <association property="role" columnPrefix="role_" resultMap="roleMap" />
</resultMap>
```
如果 `roleMap` 不在 `UserMapper.xml` 中，而是在 `RoleMapper.xml` 中
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <association property="role" columnPrefix="role_" resultMap="com.pain.mapper.RoleMapper.roleMap" />
</resultMap>
```

#### 嵌套查询
`association` 标签中的 `column` 将主查询中的列结果作为嵌套查询的参数
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <association property="role" column="{id=role_id,name=role_name}" select="selectRoleById" />
</resultMap>
```
```xml
<select id="selectUserWithRoleByUserId" resultMap="userRoleMap">
    select
      u.id,
      u.username,
      u.password,
      u.email,
      u.birth,
      
      ur.role_id
      from user u
      inner join user_role ur on u.id = ur.user_id
      where u.id = #{userId}
</select>
```
```xml
<select id="selectRoleById" resultMap="roleMap">
    select * from role where id = #{id}
</select>
```

使用嵌套查询，如果主查询查出 N 条数据，则子查询会对 N 条结果各自执行一次查询，出现 N + 1 问题。由于不一定会使用到 `Role`，可以使用延迟加
载避免 N + 1 问题。这样，只有当调用 `getRole()` 方法时才会执行嵌套查询去获取数据。
```xml
<resultMap id="userRoleMap" extends="userMap" type="com.pain.pojo.User">
    <association property="role"
        fetchType="lazy"
        select="selectRoleById"
        column="{id=role_id,name=role_name}" />
</resultMap>
```

需要注意的是，在 Mybatis 全局配置中，有一个参数 `aggressiveLazyLoading`。该参数为 `true` 对任意延迟属性的调用会使带有延迟属性的对象完
整加载。反之，每种属性都将按需加载。Mybatis 3.4.5 版本开始默认设置为 `false`
```xml
<settings>
    <setting name="aggressiveLazyLoading" value="false" />
</settings>
```

在将 `aggressiveLazyLoading` 设置为 `false` 的情况下，如果需要将所有数据都加载进来，可以通过参数 `lazyLoadTriggerMethods` 解决，默
认值为 "equals,clone,hashCode,toString"，调用其中方法会加载全部的延迟加载数据。
```
user.equals(null);
```

### 一对多映射
#### 方法一
`id` 标签非常重要，因为 mybatis 会根据 `id` 标签判断查询结果是否相同。若为相同的结果，只会保存第一个结果。Mybatis 会对嵌套查询的每一级
对象都进行属性比较。先比较顶层对象，若 `user` 部分相同，就继续比较 `role` 部分，若 `role` 部分不同，就会增加一个 `role`；若两个 `role`
相同，就会保留前一个。若 `role` 还有下层，继续按照该规则进行比较。
```xml
<resultMap id="UserWithRole" type="com.pain.pojo.User">
    <id column="id" property="id" />
    <result column="username" property="username" />
    <result column="password" property="password" />
    <result column="email" property="email" />
    <result column="birth" property="birth" />

    <collection property="roles" ofType="Role" columnPrefix="role_">
        <id column="id" property="id" />
        <result column="name" property="name" />
    </collection>
</resultMap>
```
```xml
<select id="selectUserWithRoleByUserId" resultMap="UserWithRole">
    select
      u.id,
      u.username,
      u.password,
      u.email,
      u.birth,
      r.id role_id,
      r.name role_name
      from user u
      inner join user_role ur on u.id = ur.user_id
      inner join role r on r.id = ur.role_id
      where u.id = #{userId}
</select>
```

#### 方法二
```xml
<resultMap id="RoleMap" type="com.pain.pojo.Role">
    <id column="id" property="id" />
    <result column="name" property="name" />
</resultMap>
```
```xml
<resultMap id="UserWithRole" type="com.pain.pojo.User" extends="UserMap">
    <collection property="roles" columnPrefix="role_" resultMap="RoleMap" />
</resultMap>
```
```xml
<select id="selectUserWithRoleByUserId" resultMap="UserWithRole">
    select
      u.id,
      u.username,
      u.password,
      u.email,
      u.birth,
      r.id role_id,
      r.name role_name
      from user u
      inner join user_role ur on u.id = ur.user_id
      inner join role r on r.id = ur.role_id
      where u.id = #{userId}
</select>
```

#### 多层嵌套
```xml
<resultMap id="privilegeMap" type="com.pain.pojo.Privilege">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="url" column="url" />
</resultMap>
```
```xml
<resultMap id="roleWithPrivilegeMap" type="com.pain.pojo.Role" extends="RoleMap">
    <collection property="privileges" columnPrefix="privilege_" resultMap="privilegeMap" />
</resultMap>
```
```xml
<resultMap id="UserWithRole" type="com.pain.pojo.User" extends="UserMap">
    <collection property="roles" columnPrefix="role_" resultMap="roleWithPrivilegeMap" />
</resultMap>
```
注意 column 别名的前缀
```xml
<select id="selectUserWithRoleByUserId" resultMap="UserWithRole">
    select
    u.id,
    u.username,
    u.password,
    u.email,
    u.birth,
    r.id role_id,
    r.name role_name,
    p.id role_privilege_id,
    p.name role_privilege_name,
    p.url role_privilege_url
    from user u
    inner join user_role ur on u.id = ur.user_id
    inner join role r on r.id = ur.role_id
    inner join role_privilege rp on r.id = rp.role_id
    inner join privilege p on rp.privilege_id = p.id
    where u.id = #{userId}
</select>
```

#### 嵌套查询
roleId => privilege
```xml
<select id="selectPrivilegeByRoleId" resultMap="privilegeMap">
    select
    p.id,
    p.name,
    p.url
    from privilege p
    inner join role_privilege rp
    on rp.privilege_id = p.id
    where rp.role_id = #{roleId}
</select>
```
```xml
<resultMap id="roleWithPrivilege" type="com.pain.pojo.Role" extends="RoleMap">
    <!--roleId 是 select 指定查询方法中的参数-->
    <!--id 是当前查询 selectRoleByUserId 查询出的角色的 id-->
    <collection property="privileges" fetchType="lazy" column="{roleId=id}"
                select="selectPrivilegeByRoleId" />
</resultMap>
```
userId => role
```xml
<select id="selectRoleByUserId" resultMap="roleWithPrivilege">
    select
    r.id,
    r.name
    from role r
    inner join user_role ur on ur.role_id = r.id
    where ur.user_id = #{userId}
</select>
```
```xml
<resultMap id="userWithRole" type="com.pain.pojo.User" extends="UserMap">
    <collection property="roles" fetchType="lazy" column="{userId=id}"
                select="selectRoleByUserId" />
</resultMap>
```
以上使用延迟加载，避免 N + 1 问题
```xml
<select id="selectUserById" resultMap="userWithRole">
    select
    u.id,
    u.username,
    u.password,
    u.email,
    u.birth
    from user u
    where u.id = #{userId}
</select>
```