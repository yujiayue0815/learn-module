# Maven

## Getting Started Guide

### 创建Maven项目

```shell
mvn -B archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4
```

### 编译项目

```shell
mvn compile	
```

### 测试

```shell
mvn test
```

### 编译但不测试

```shell
mvn test-compile
```

### 打包

```shell
mvn package
```

### 包安装到本地仓库

```shell
mvn install
```

### 生成项目Maven文档

```shell
mvn site	
```

### 清除构建的数据目录

```shell
mvn clean
```

### 过滤资源文件

```xml
 <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
 </resources>
```

### 替换资源参数

```shell
mvn process-resources
```

### 可用内置参数

| 名称                       | 说明      |
| -------------------------- | --------- |
| ${project.name}            | 项目名称  |
| ${project.version}         | 项目版本  |
| ${project.bulid.finalName} | jar包名称 |

### 引入外部资源文件

```xml
  <build>
    <filters>
      <filter>src/main/filters/filter.properties</filter>
    </filters>
  </build>
```

Pom资源配置

```xml
  <properties>
    <my.filter.value>hello</my.filter.value>
  </properties>
```

### 参数出传递方式

```shell
mvn process-resources "-Dcommand.line.prop=hello again"
```

