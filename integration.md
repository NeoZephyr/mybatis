## 集成
### Spring
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.0</version>
</dependency>
```
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
```
配置 `SqlSessionFactoryBean`
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:mybatis-config.xml" />
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations">
        <array>
            <value>classpath:mapper/*.xml</value>
        </array>
    </property>
    <property name="typeAliasesPackage" value="com.pain.pojo" />
</bean>
```
配置 `MapperScannerConfigurer`，自动扫描所有 `Mapper` 接口
```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.pain.mapper" />
</bean>
```

### SpringBoot
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.2.0</version>
</dependency>
```
```java
@Mapper
public interface UserMapper {}
```
配置 MyBatis
```
mybatis.mapperLocation=classpath:mapper/*.xml
mybatis.typeAliasedPackage=com.pain.pojo
```
Mybatis Starter 提供的所有可配置属性都在 `org.mybatis.spring.boot.autoconfigure.MybatisProperties` 类中
```java
@ConfigurationProperties(prefix = MybatisProperties.MYBATIS_PREFIX)
public class MybatisProperties {
    public static final String MYBATIS_PREFIX = "mybatis";
    
    private String configLocation;
    private String[] mapperLocations;
    private String typeAliasesPackage;
    
    @NestedConfigurationProperty
    private Configuration configuration;
}
```
通过 `@ConfigurationProperties` 注解自动将配置文件中的属性组装到对象上，这个注解需要配置与属性匹配的前缀，属性类中驼峰形式的字段在配置
文件中改为横杠和小写连接的形式。此外，类 `MybatisProperties` 还提供了一个嵌套的 `Configuration` 属性，可以直接对 `Configuration` 对
象进行属性配置，例如 `settings` 中的属性可以按照下面的方式进行配置：
```
mybatis.configuration.lazy-loading-enabled=true
mybatis.configuration.aggressive-lazy-loading=true
```
当然，仍然可以通过 mybatis-config.xml 方式去配置 Mybatis，然后在 Spring Boot 的配置中指定该文件的路径即可：
```
mybatis.config-location=classpath:mybatis-config.xml
```