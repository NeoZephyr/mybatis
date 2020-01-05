## Mybatis Generator

### 配置
参考 `generator.xml` 文件

### 运行

#### 编码运行
```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.3</version>
</dependency>
```
```java
public class Generator {
    public static void main(String[] args){
      List<String> warnings = new ArrayList<String>();
      boolean overwrite = true;
      
      // 读取配置文件
      InputStream is = Generator.class.getResourceAsStream("generator.xml");
      ConfigurationParser cp = new ConfigurationParser(warnings);
      Configuration config = cp.parseConfiguration(is)
      is.close();
      
      DefaultShellCallback callback = new DefaultShellCallback(overwrite);
      
      // 创建 MBG，执行生成代码
      MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
      myBatisGenerator.generate(null);
      
      // 输出警告信息
      for (String warning : warnings) {
          System.out.println(warning);
      }
    }
}
```

#### 命令行运行
```sh
java -Dfile.encoding=UTF-8 -jar mybatis-generator-core-1.3.3.jar -configfile generator.xml -overwrite
```

#### Maven Plugin 方式运行
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.3</version>
    <configuration>
        <configurationFile>
            ${basedir}/src/main/resources/generator.xml
        </configurationFile>
        <overwrite>true</overwrite>
        <verbose>true</verbose>
    </configuration>
</plugin>
```
```sh
mvn mybatis-generator:generate
```