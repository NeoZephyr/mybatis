## Mybatis 缓存

### 一级缓存
Mybatis 的一级缓存存在于 `SqlSession` 的生命周期中，在同一个 `SqlSession` 中查询时，Mybatis 会把执行的方法和参数通过算法生成缓存的键
值，将键值和查询结果存入一个 `Map` 对象中。如果不想使用一级缓存，可以通过 `flushCache` 属性控制，示例如下：
```xml
<select id="selectById" flushCache="true" resultMap="userMap">
    select * from user where id = #{id}
</select>
```
设置参数 `flushCache` 之后，会在查询数据前清空当前的一级缓存。另外，任何的 `insert`，`update`，`delete` 操作都会清空一级缓存。

### 二级缓存
#### 配置
Mybatis 在全局配置 `settings` 中有参数 `cacheEnabled` 作为二级缓存的全局开关，默认为 `true`。若该参数设置为 `false`，即使有后面的二
级缓存配置，也不会生效。
```xml
<settings>
    <setting name="cacheEnabled" value="true" />
</settings>
```

Mybatis 的二级缓存是和命名空间绑定的，即二级缓存需要配置在 `Mapper` 映射文件中。在映射文件中，命名空间就是 XML 根节点 `mapper` 的 
`namespace` 属性。而爱 `Mapper` 映射文件中开启二级缓存，只需要添加 `<cache/>` 元素即可

#### 默认二级缓存
映射语句文件中的所有 `SELECT` 语句将会被缓存；映射语句文件中的所有 `INSERT`, `UPDATE`, `DELETE` 语句会刷新缓存；缓存会使用 LRU 算法
来回收；缓存会存储对象的 1024 个引用

#### 缓存属性
1. `eviction` 即回收策略
    1.1 LRU，移除最长时间不被使用的对象，这是默认值
    1.2 FIFO，按照对象进入缓存的顺序移除
    1.3 SOFT，移除基于垃圾回收器状态和软引用规则的对象
    1.4 WEAK，更积极地移除基于垃圾回收器状态和弱引用规则的对象
2. `flushInterval`，即刷新间隔。单位为毫秒
3. `size`，引用数目
4. `readOnly`，设置是否只读。只读的缓存会给所有调用者返回缓存对象相同实例；可读写缓存会返回缓存对象拷贝，可能会慢一些，但更安全。默认值为 `false`

#### 注意
当调用 `close` 方法关闭 `SqlSession` 时，`SqlSession` 才会保存查询数据到二级缓存中；此外，要避免无意义地修改数据（从二级缓存中查询到
结果，修改之后并未提交，这样缓存不会清空，而缓存与数据库中的数据就不一致来），避免缓存与数据库的数据不一致；最后，由于每一个 `Mapper` 映射
文件都有自己的二级缓存，不同 `Mapper` 的二级缓存互不影响。但由于多表联合查询的存在，其查询结果一定存在某个命名空间下的映射文件中，而这些表
的增、删、改、查通常不在一个映射文件中，当有数据变化时，多表查询的缓存未必被清空，这样导致来脏数据。这种情况，可以使用参照索引，即让关联的表
使用同一个二级缓存，配置如下：
```xml
<mapper namespace="com.pain.mapper.UserMapper">
    <cache-ref namespace="com.pain.mapper.RoleMapper" />
</mapper>

```

#### 缓存实现
默认基于 `Map` 实现的内存缓存，此外还可以集成 `EhCache` 以及 `Redis`