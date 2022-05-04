---
title: SpringBootStarter
categories: 
- 常用框架
- Spring
- SpringBoot
---

Spring Boot 对比 Spring MVC 最大的优点就是使用简单，约定大于配置。

* 这要归功于组成了 Spring Boot 的各种各样的 starters，有官方提供的，也有第三方开源出来。

用 Spring Boot 的功能组件（例如 spring-boot-starter-actuator、 spring-boot-starter-data-redis 等）的步骤非常简单。

**有以下三步就可以完成组件功能的使用：**

在 pom 文件中引入对应的包，例如：

```xml
<dependency> 
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-actuator</artifactId> 
</dependency>
```

在应用配置文件中加入相应的配置，配置都是组件约定好的，需要查看官方文档或者相关说明。

* 有些比较复杂的组件，对应的参数和规则也相应的较多，有点可能多大几十上百了。

以上两步都正常的情况下，我们就可以使用组件提供的相关接口来开发业务功能了。

**spring boot starter 具体 是什么呢？**

> 它首先是一个包，一个集合，它把需要用的其他功能组件囊括进来，放到自己的 pom 文件中。、
>
> 然后它是一个连接，把它引入的组件和我们的项目做一个连接，并且在中间帮我们省去复杂的配置，力图做到使用最简单。

**实现一个 spring boot starter** 

实现一个 starter 有四个要素：

1. starter 命名 ;
2. 自动配置类，用来初始化相关的 bean ;
3. 指明自动配置类的配置文件 spring.factories ;
4. 自定义属性实体类，声明 starter 的应用配置属性 ;

> 给 starter 起个名

也就是我们使用它的时候在 pom 中引用的 artifactId。

命名有规则的，官方规定：

* 官方的 starter 的命名格式为 spring-boot-starter-{name} ，例如上面提到的 spring-boot-starter-actuator。

* 非官方的 starter 的命名格式为 {name}-spring-boot-starter，我们把自定的 starter 命名为 kite-spring-boot-starter，命名在 pom 文件里。

```xml
<groupId>kite.springcloud</groupId>
<artifactId>kite-spring-boot-starter</artifactId>
<packaging>jar</packaging>
<version>1.0-SNAPSHOT</version>
```

> 引入自动配置包及其它相关依赖包

实现 starter 主要依赖自动配置注解，所以要在 pom 中引入自动配置相关的两个 jar 包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```

> 创建 spring.factories 文件

在 resource/META-INF 目录下创建名称为 spring.factories 的文件?

> 当 Spring Boot 启动的时候，会在 classpath 下寻找所有名称为 spring.factories 的文件，然后运行里面的配置指定的自动加载类，将指定类(一个或多个)中的相关 bean 初始化。

例如本例中的配置信息是这样的：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  kite.springcloud.boot.starter.example.KiteAutoConfigure
```

等号前面是固定的写法，后面就是我们自定义的自动配置类了，如果有多个的话，用英文逗号分隔开。

> 编写自动配置类

自动配置类是用来初始化 starter 中的相关 bean 的。可以说是实现 starter 最核心的功能。

```java
@Configuration
@ConditionalOnClass(KiteService.class)
@EnableConfigurationProperties(KiteProperties.class)
@Slf4j
public class KiteAutoConfigure {

    @Autowired
    private KiteProperties kiteProperties;

    @Bean
    @ConditionalOnMissingBean(KiteService.class)
    @ConditionalOnProperty(prefix = "kite.example",value = "enabled", havingValue = "true")
    KiteService kiteService(){
        return new KiteService(kiteProperties);
    }
}
```

@Configuration 表示这是个自动配置类，我们平时做项目时也会用到，一般是用作读取配置文件的时候。

@ConditionalOnClass(KiteService.class) ：

* 只有在 classpath 中找到 KiteService 类的情况下，才会解析此自动配置类，否则不解析。

@EnableConfigurationProperties(KiteProperties.class)：

* 启用配置类。

@Bean：实例化一个 bean 。

@ConditionalOnMissingBean(KiteService.class)：

* 与 @Bean 配合使用，只有在当前上下文中不存在某个 bean 的情况下才会执行所注解的代码块，也就是当前上下文还没有 KiteService 的 bean 实例的情况下，才会执行 kiteService() 方法，从而实例化一个 bean 实例出来。

@ConditionalOnProperty：

* 当应用配置文件中有相关的配置才会执行其所注解的代码块。

**这个类的整体含义就是:** 

当 classpath 中存在 KiteService 类时解析此配置类，什么情况下才会在 classpath 中存在呢，就是项目引用了相关的 jar 包。并且在上下文中没有 KiteService 的 bean 实例的情况下，new 一个实例出来，并且将应用配置中的相关配置值传入。

> 实现属性配置类

```java
@Data
@ConfigurationProperties("kite.example")
public class KiteProperties {

    private String host;

    private int port;
}
```

配置类很简单，只有两个属性，一个 host ，一个 port 。配置参数以 kite.example 作为前缀。

> 实现相关功能类

也就是前面一直提到的 KiteService。

```java
@Slf4j
public class KiteService {

    private String host;

    private int port;

    public KiteService(KiteProperties kiteProperties){
        this.host = kiteProperties.getHost();
        this.port = kiteProperties.getPort();
    }

    public void print(){
        log.info(this.host + ":" +this.port);
    }
}
```

> 打包

通过 maven 命令将这个 starter 安装到本地 maven 仓库

```bash
mvn install
```

也可以通过 `mvn package deploy` 发布到你的私服或者发布到中央仓库。

上面已经完成了 starter 的开发，并安装到了本地仓库，然后就是在我们的项目中使用它了。

**开始使用**

> 创建项目，在 pom 中引用

```xml
<dependency>
    <groupId>kite.springcloud</groupId>
    <artifactId>kite-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

> 应用配置项

创建 application.yml ，配置如下:

```yaml
server:
  port: 3801
kite:
  example:
    enabled: true  # 开启才生效
    host: 127.0.0.1
    port: 3801
```

> 调用 KiteService 的服务方法

```java
@RestController
@RequestMapping(value = "use")
public class UseController {

    @Autowired
    private KiteService kiteService;

    @GetMapping(value = "print")
    public void print(){
        kiteService.print();
    }
}
```

> 启动服务，并访问接口

访问 /use/print 接口，会发现在日志中打印出了配置信息

```text
2019-05-24 16:45:04.234  INFO 36687 --- [nio-3801-exec-1] k.s.boot.starter.example.KiteService     : 127.0.0.1:3801
```