---
title: SPI机制
categories: 
- Java基础
---

**SPI简介**

SPI 全称为 Service Provider Interface，是一种服务发现机制。

SPI本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类，这样运行时可以动态的为接口替换实现类。

它通过在ClassPath路径下的META-INF/services文件夹查找文件，自动加载文件里所定义的类。

这一机制为很多框架扩展提供了可能，比如在Dubbo、JDBC中都使用到了SPI机制。

**JDK SPI 机制**

当某个应用引入了该 jar 包且需要使用该服务时，JDK SPI 机制就可以通过查找这个 jar 包的 META-INF/services/ 中的配置文件来获得具体的实现类名，进行实现类的加载和实例化，最终使用该实现类完成业务功能。

 **JDK SPI 的基本使用方式：**

1、首先创建一个**database-driver**工程，并创建**DataBaseDriver**接口：

```tsx
public interface DataBaseDriver {

    String connect(String host);
}
```

2、创建**mysql-driver**工程

3、引入**database-driver**工程依赖，并实现**DataBaseDriver**接口：

```java
public class MysqlDriver implements DataBaseDriver {

    @Override
    public String connect(String host) {
        return "begin build Mysql connect："+host;
    }
}
```

4、在**mysql-driver**工程的 resources/META-INF/services 目录下添加一个名为 `com.yibo.spi.DataBaseDriver`的文件，这是 JDK SPI 需要读取的配置文件，具体内容如下：

```css
com.yibo.spi.MysqlDriver
```

5、创建**oracle-driver**工程

6、引入**database-driver**工程依赖，并实现**DataBaseDriver**接口：

```java
public class OracleDriver implements DataBaseDriver{

    @Override
    public String connect(String host) {
        return "begin build Oracle connect："+host;
    }
}
```

7、在**oracle-driver**工程的 resources/META-INF/services 目录下添加一个名为 `com.yibo.spi.DataBaseDriver`的文件，这是 JDK SPI 需要读取的配置文件，具体内容如下：

```css
com.yibo.spi.OracleDriver
```

8、新建**client-demo**工程，引入**database-driver**、**mysql-driver**、**oracle-driver**依赖

```xml
<dependencies>
    <dependency>
        <artifactId>database-driver</artifactId>
        <groupId>com.yibo</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <artifactId>mysql-driver</artifactId>
        <groupId>com.yibo</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <artifactId>oracle-driver</artifactId>
        <groupId>com.yibo</groupId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

9、创建测试类，加载**DataBaseDriver**接口，调用connect方法

```kotlin
public class App {

    public static void main( String[] args ) {
        ServiceLoader<DataBaseDriver> serviceLoader = ServiceLoader.load(DataBaseDriver.class);
        System.out.println( "Java SPI" );
        for (DataBaseDriver driver : serviceLoader) {
            System.out.println(driver.connect("localhost"));
        }
    }
}

#输出
Java SPI
begin build Mysql connect：localhost
begin build Oracle connect：localhost
```

**JDK SPI 在 JDBC 中的应用**

JDK 中只定义了一个 `java.sql.Driver` 接口，具体的实现是由不同数据库厂商来提供的。

在 `mysql-connector-java-*.jar` 包中的 META-INF/services 目录下，有一个 `java.sql.Driver` 文件中只有一行内容，如下所示：

```css
com.mysql.cj.jdbc.Driver 
```

在使用 `mysql-connector-java-*.jar `包连接 MySQL 数据库的时候，我们会用到如下语句创建数据库连接：

```bash
String url = "jdbc:xxx://xxx:xxx/xxx"; 
Connection conn = DriverManager.getConnection(url, username, pwd); 
```

