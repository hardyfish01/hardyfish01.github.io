---
title: Bean注册
categories: 
- 常用框架
- Spring
---

**XML文件配置的方式**

Spring的xml配置文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="user" class="com.zhou.bean.User">
		<property name="age" value="110"></property>
		<property name="name" value="月伴飞鱼"></property>
	</bean>
</beans>
```

测试类测试：

```java
public class MainTest {
    public static void main(String[]args){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User user = (User) applicationContext.getBean("User");
        System.out.println(user);
    }
}
```

**注解的方式**

配置类：

```java
@Configuration
public class MainConfig {

    //式默认用的是方法名来作为bean的id
    @Bean
    public Person person() {
        return new Person("lisi",20);
    }
  
    @Bean(value = "person")//通过这个value属性可以指定bean在IOC容器的id
    public Person person01() {
        return new Person("lisi",20);
    }
}
```

测试类测试：

```java
public class MainTest {
    public static void main(String[]args){
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = applicationContext.getBean(Person.class);
        System.out.println(person);
      
        //我们可以来获取bean的定义信息
        String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
        for (String name : namesForType) {
          System.out.println(name);
        }
    }
}
```

**@ComponentScan:自动扫描组件**

```java
    <!-- 包扫描、标注了@Controller、@Service、@Repository，@Component -->
    <context:component-scan base-package="com.zhou"/>
```

在xml配置文件里面可以写包扫描，也可以在配置类里面写包扫描：

```java
@Configuration
@ComponentScan(value = "com.zhou")
public class MainConfig {

    @Bean(value = "person")//通过这个value属性可以指定bean在IOC容器的id
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

我们在 `@ComponentScan` 这个注解上，也可以指定要排除哪些包或者是只包含哪些包来进行管理。

```java
@Configuration
@ComponentScan(value = "com.ldc",excludeFilters = {
        //FilterType.ANNOTATION：按照注解的方式来进行排除
        //classes = {Controller.class,Service.class}表示的是标有这些注解的类给排除掉
        @Filter(type = FilterType.ANNOTATION,classes = {Controller.class,Service.class})
})
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

也可以来配置**includeFilters**：指定在扫描的时候，只需要包含哪些组件

在用xml文件配置的方式来进行配置的时候，还要禁用掉默认的配置规则，只包含哪些组件的配置才能生效

```xml
<context:component-scan base-package=“com.zhou” use-default-filters=“false”/>
```

```java
@Configuration
@ComponentScan(value = "com.zhou",includeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
},useDefaultFilters = false)
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

可以用 `@ComponentScans`来定义多个扫描规则：里面是`@ComponentScan`规则的数组

```java
@Configuration
@ComponentScans(value = {
    @ComponentScan(value = "com.zhou",includeFilters = {
            @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
    },useDefaultFilters = false),
    @ComponentScan(value = "com.zhou")
})
public class MainConfig {

    @Bean(value = "person")
    public Person person01() {
        return new Person("lisi",20);
    }

}
```

**@Scope设置组件作用域**

```java
@Configuration
public class MainConfig {
    //singleton:单实例的
    //prototype:多实例的
    //request:同一次请求创建一个实例
    //session:同一个session创建的一个实例
    @Scope("prototype")
    //默认是单实例的
    @Bean("person")
    public Person person() {
        return new Person();
    }
}
```

**@Lazy懒加载**

懒加载：是专门针对于单实例的Bean的：

- 单实例的bean：默认是在容器启动的时候创建对象；
- 懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化

```java
@Configuration
public class MainConfig {

    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}
```

**@Import给容器中快速导入一个组件**

```java
@Configuration
//快速导入组件，id默认是组件的全类名
@Import(Person.class)
public class MainConfig {

}
```

**使用ImportSelector**

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    //返回值就是要导入到容器中的组件的全类名
    //AnnotationMetadata ：当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //方法不要返回null值
        return new String[]{"com.zhou.bean.Blue","com.ldc.zhou.Yellow"};
    }
}
```

```java
@Configuration
@Import(MyImportSelector.class)
public class MainConfig {

}
```

**使用ImportBeanDefinitionRegistrar**

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     * AnnotationMetadata 当前类的注解信息
     * BeanDefinitionRegistry BeanDefinition注册类
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //判断IOC容器里面是否含有这两个组件
        boolean definition = registry.containsBeanDefinition("com.zhou.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("com.zhou.bean.Blue");
        //如果有的话，我就把RainBow的bean的实例给注册到IOC容器中
        if (definition && definition2) {
            //指定bean的定义信息，参数里面指定要注册的bean的类型：RainBow.class
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个bean，并且指定bean名
            registry.registerBeanDefinition("rainBow", rootBeanDefinition );
        }
    }
}
```

```java
@Configuration
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig {

}
```

**使用FactoryBean注册组件**

```javascript
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {

    //返回一个Color对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean...getBean...");
        return new Color();
    }

    //返回的类型
    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    //控制是否为单例
    // true：表示的就是一个单实例，在容器中保存一份
    // false:多实例，每次获取都会创建一个新的bean
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```java
@Configuration
public class MainConfig {

    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }
}
```

**@Conditional按照条件注册Bean**

```java
public class MySQLCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return false;
    }
}
```

```java
@Configuration
public class MainConfig {

    @Conditional({MySQLCondition.class})
    @Bean("aa")
    public Person person() {
        return new Person("aa",48);
    }

}
```

