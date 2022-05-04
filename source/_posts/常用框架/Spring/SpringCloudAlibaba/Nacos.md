---
title: SpringCloudAlibaba
categories:
- 常用框架
- Spring
- Nacos
---

**服务调用问题**

为应对服务的压力，采用多实例集群部署已成为简捷易用的解决方案。

仅仅多实例部署后，直接面临一个问题，调用方如何知晓调用哪个实例，当实例运行失败后，如何转移到别的实例上去处理请求？

> 如果采用了负载均衡，但往往是静态的，在服务不可用时，如果动态的更新负载均衡列表，保证调用者的正常调用呢？

面对以上两种情况，服务注册中心的需求迫在眉捷，将所有的服务统一的、动态的管理起来。

**服务注册中心**

服务注册中心作分布式服务框架的核心模块，要实现的功能是服务的注册、订阅，与之相应的功能是注销、通知这四个功能。

所有的服务都与注册中心发生连接，由注册中心统一配置管理，不再由实例自身直接调用。

服务管理过程大致过程如下：

* 服务提供者启动时，将服务提供者的信息主动提交到服务注册中心进行服务注册。

* 服务调用者启动时，将服务提供者信息从注册中心下载到调用者本地，调用者从本地的服务提供者列表中，基于某种负载均衡策略选择一台服务实例发起远程调用，这是一个点到点调用的方式。

* 服务注册中心能够感知服务提供者某个实例下线，同时将该实例服务提供者信息从注册中心清除，并通知服务调用者集群中的每一个实例，告知服务调用者不再调用本实例，以免调用失败。

**Nacos 应用**

官网地址：https://nacos.io/en-us/，由阿里开源，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台，已经作为Spring Cloud Alibaba 的一个子项目，更好与 Spring Cloud 融合在一起。

# 单机版

**安装 Nacos**

下载后解压[nacos-server-2.0.4.tar.gz ](https://github.com/alibaba/nacos/releases/download/2.0.4/nacos-server-2.0.4.tar.gz)，直接使用对应命令启动。

> tar -xvf nacos-server-$version.tar.gz 
>
> cd nacos/bin 
>
> sh startup.sh -m standalone (standalone代表着单机模式运行，非集群模式)

启动日志末显示 "staring" 表示启动成功，打开http://127.0.0.1:8848/nacos，输入默认的用户名 nacos、密码 nacos。

关闭服务：

> sh shutdown.sh

**服务中应用 Nacos**

父项目 parking-project 的 pom.xml中增加如下配置：

```xml
    <properties>
        <spring-cloud.version>Greenwich.SR4</spring-cloud.version>
        <spring-cloud-alibaba.version>2.1.0.RELEASE</spring-cloud-alibaba.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

在子模块服务中引入 nacos jar 包，在子模块的 pom.xml 文件中增加如下配置

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>
```

模块启动类中加入 @EnableDiscoveryClient 注解，这与使用 Eureka 时，采用注解配置是一致的，此注解基于 spring-cloud-commons ，是一种通用解决方案。

在对应的项目配置文件 application.properties 中增加配置项：

```xml
#必须填写application.name，否则服务无法注册到nacos
spring.application.name=card-service/member-service
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

启动项目，通过nacos控制台检查服务是否注册到nacos。