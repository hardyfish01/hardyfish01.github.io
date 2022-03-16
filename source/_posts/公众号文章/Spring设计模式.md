---
title: Spring设计模式
categories: 
- 公众号文章
---

# 前言

前几天，一位**读者面阿里**被问到一个问题：Spring框架用到了哪些设计模式？，答的不是很好，于是打算写篇文章讲讲这个！

* 文章首发在公众号（月伴飞鱼），之后同步到个人网站：https://xiaoflyfish.cn/
* 面经：[社招一年半面经分享(含阿里美团头条京东滴滴)](https://mp.weixin.qq.com/s/SOErvCCrmPaAVUphSO2Wqw)

![](https://img-blog.csdnimg.cn/ed97dd67ef6f4df4adde3af262888c6f.png)

微信搜索：**月伴飞鱼**，交个朋友，进面试交流群

* 公众号后台回复666，可以获得免费电子书籍

**觉得不错，希望点赞，在看，转发支持一下，谢谢**

# 代理模式

所谓代理，是指它与被代理对象实现了相同的接口，客户端必须通过代理才能与被代理的目标类进行交互，而代理一般在交互的过程中（交互前后），进行某些特定的处理，比如在调用这个方法前做前置处理，调用这个方法后做后置处理。

代理又分为静态代理和动态代理两种方式，Spring的AOP采用的是动态代理的方式

Spring通过动态代理对类进行方法级别的切面增强，动态生成目标对象的代理类，并在代理类的方法中设置拦截器，通过执行拦截器中的逻辑增强了代理方法的功能，从而实现AOP。

**关于动态代理可以看我之前的文章，写的很详细**：[动态代理总结，你要知道的都在这里，无废话！](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485541&idx=1&sn=371e92596b8e7ba47019b15767b18873&scene=21#wechat_redirect)

# 策略模式

我们前面讲到，Spring AOP是通过动态代理来实现的。

具体到代码实现，Spring支持两种动态代理实现方式，一种是JDK提供的动态代理实现方式，另一种是Cglib提供的动态代理实现方式。

Spring会在运行时动态地选择不同的动态代理实现方式。**这个应用场景实际上就是策略模式的典型应用场景。**

我们只需要定义一个策略接口，让不同的策略类都实现这一个策略接口。对应到Spring源码，AopProxy是策略接口，JdkDynamicAopProxy、CglibAopProxy是两个实现了AopProxy接口的策略类。

**其中，AopProxy接口的定义如下所示：**

![](https://img-blog.csdnimg.cn/d0266b32cacf4d8a9d4b47811b25730e.png)

在策略模式中，策略的创建一般通过工厂方法来实现。对应到Spring源码，AopProxyFactory是一个工厂类接口，DefaultAopProxyFactory是一个默认的工厂类，用来创建AopProxy对象。

源码如下所示：

![](https://img-blog.csdnimg.cn/9eb06c1cf175456b9c2f56f7d78ff8a8.png)

策略模式的典型应用场景，一般是通过环境变量、状态值、计算结果等动态地决定使用哪个策略。

对应到Spring源码中，我们可以参看刚刚给出的DefaultAopProxyFactory类中的createAopProxy()函数的代码实现。

**其中，第10行代码是动态选择哪种策略的判断条件。**

# 装饰器模式

我们知道，缓存一般都是配合数据库来使用的。如果写缓存成功，但数据库事务回滚了，那缓存中就会有脏数据。

为了解决这个问题，我们需要将缓存的写操作和数据库的写操作，放到同一个事务中，要么都成功，要么都失败。

**实现这样一个功能，Spring使用到了装饰器模式。**

TransactionAwareCacheDecorator增加了对事务的支持，在事务提交、回滚的时候分别对Cache的数据进行处理。

TransactionAwareCacheDecorator实现Cache接口，并且将所有的操作都委托给targetCache来实现，对其中的写操作添加了事务功能。这是典型的装饰器模式的应用场景和代码实现。

![](https://img-blog.csdnimg.cn/0512c704ef2a48948122aeaa55935038.png)

# 单例模式

单例模式是指一个类在整个系统运行过程中，只允许产生一个实例

在Spring中，Bean可以被定义为两种模式：Prototype（多例）和Singleton（单例），Spring Bean默认是单例模式。

**那Spring是如何实现单例模式的呢？**

答案是通过单例注册表的方式，具体来说就是使用了HashMap。简化代码如下：

```java
public class DefaultSingletonBeanRegistry {
    
    //使用了线程安全容器ConcurrentHashMap，保存各种单实例对象
    private final Map singletonObjects = new ConcurrentHashMap;

    protected Object getSingleton(String beanName) {
    //先到HashMap中拿Object
    Object singletonObject = singletonObjects.get(beanName);
    
    //如果没拿到通过反射创建一个对象实例，并添加到HashMap中
    if (singletonObject == null) {
      singletonObjects.put(beanName,
                           Class.forName(beanName).newInstance());
   }
   
   //返回对象实例
   return singletonObjects.get(beanName);
  }
}
```

上面的代码逻辑比较清晰，先到HashMap去拿单实例对象，没拿到就创建一个添加到HashMap。

# 简单工厂模式

**有这样一个场景：**

当A对象需要调用B对象的方法时，我们需要在A中new一个B的实例，它的缺点是一旦需求发生变化，比如需要使用C类来代替B时，就要改写A类的方法。

假如应用中有100个类以的方式耦合了B，那改起来就费劲了。

**使用简单工厂模式：**

简单工厂模式又叫静态工厂方法，其实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

其中Spring中的BeanFactory就是简单工厂模式的体现，BeanFactory是Spring IOC容器中的一个核心接口，它的定义如下：

![](https://img-blog.csdnimg.cn/b7134f2874f64daab6e91cc7405049f1.png)

我们可以通过它的具体实现类（比如ClassPathXmlApplicationContext）来获取Bean：

```java
BeanFactory bf = new ClassPathXmlApplicationContext("spring.xml");
FlyFish flyFishBean = (FlyFish) bf.getBean("flyfishBean");
```

从上面代码可以看到，使用者不需要自己来new对象，而是通过工厂类的方法getBean来获取对象实例，这是典型的简单工厂模式，只不过Spring是用反射机制来创建Bean的。

# 工厂方法模式

在简单工厂中，由工厂类进行所有的逻辑判断、实例创建；如果不想在工厂类中进行判断，可以为不同的产品提供不同的工厂，不同的工厂生产不同的产品，每一个工厂都只对应一个相应的对象，这就是工厂方法模式。

Spring中的FactoryBean就是这种思想的体现，FactoryBean可以理解为工厂Bean，先来看看它的定义：

![](https://img-blog.csdnimg.cn/6cb8a14078d34b4188b4465ab3033128.png)

我们定义一个类FlyFishFactoryBean来实现FactoryBean接口，主要是在getObject方法里new一个FlyFish对象。这样我们通过getBean(id) 获得的是该工厂所产生的FlyFish的实例，而不是FlyFishFactoryBean本身的实例，像下面这样：

```java
BeanFactory bf = new ClassPathXmlApplicationContext("spring.xml");
FlyFish flyFishBean = (FlyFish) bf.getBean("flyfishBean");
```

# 观察者模式

Spring中实现的观察者模式包含三部分：Event事件（相当于消息）、Listener监听者（相当于观察者）、Publisher发送者（相当于被观察者）

我们通过一个例子来看下Spring提供的观察者模式是怎么使用的

```java
// Event事件
public class DemoEvent extends ApplicationEvent {
  private String message;

  public DemoEvent(Object source, String message) {
    super(source);
  }

  public String getMessage() {
    return this.message;
  }
}

// Listener监听者
@Component
public class DemoListener implements ApplicationListener {
  @Override
  public void onApplicationEvent(DemoEvent demoEvent) {
    String message = demoEvent.getMessage();
    System.out.println(message);
  }
}

// Publisher发送者
@Component
public class DemoPublisher {
  @Autowired
  private ApplicationContext applicationContext;

  public void publishEvent(DemoEvent demoEvent) {
    this.applicationContext.publishEvent(demoEvent);
  }
}
```

从代码中，我们可以看出，主要包含三部分工作：

- 定义一个继承ApplicationEvent的事件（DemoEvent）；
- 定义一个实现了ApplicationListener的监听器（DemoListener）；
- 定义一个发送者（DemoPublisher），发送者调用ApplicationContext来发送事件消息。

**在Spring的实现中，观察者注册到了哪里呢？又是如何注册的呢？**

Spring把观察者注册到了ApplicationContext对象中。

实际上，具体到源码来说，ApplicationContext只是一个接口，具体的代码实现包含在它的实现类AbstractApplicationContext中。我把跟观察者模式相关的代码，如下。你只需要关注它是如何发送事件和注册监听者就好。

![](https://img-blog.csdnimg.cn/5f2a44383f0846888d7cb4cc42581aa1.png)

从上面的代码中，我们发现，真正的消息发送，实际上是通过ApplicationEventMulticaster这个类来完成的。

下面这个类的源码我只摘抄了最关键的一部分，也就是multicastEvent()这个消息发送函数，它通过线程池，支持异步非阻塞、同步阻塞这两种类型的观察者模式。

![](https://img-blog.csdnimg.cn/352ce797c65942c38659ea7cdeefc72a.png)

借助Spring提供的观察者模式的骨架代码，如果我们要在Spring下实现某个事件的发送和监听，只需要做很少的工作，定义事件、定义监听器、往ApplicationContext中发送事件就可以了，剩下的工作都由Spring框架来完成。

实际上，这也体现了Spring框架的扩展性，也就是在不需要修改任何代码的情况下，扩展新的事件和监听。

# 模板模式

我们经常在面试中被问到的一个问题：

> 请你说下Spring Bean的创建过程包含哪些主要的步骤。

这其中就涉及模板模式。它也体现了Spring的扩展性。利用模板模式，Spring能让用户定制Bean的创建过程。

下面是Spring Bean的整个生命周期，一张图，清晰明了：

更多详细内容，可以看看之前的文章：[Spring奇技淫巧之扩展点的应用](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485001&idx=1&sn=2b84dd07d216fde4cea4b0c31e16a90f&scene=21#wechat_redirect)

![](https://img-blog.csdnimg.cn/df041737e435461a80ee1a74358d3fbf.png)

如果你仔细看过源码会发现，实际上，这里的模板模式的实现，并不是标准的抽象类的实现方式，而是有点类似Callback回调的实现方式，也就是将要执行的函数封装成对象（比如，初始化方法封装成InitializingBean对象），传递给模板（BeanFactory）来执行。

**观察者模式和模板模式，这两种模式能够帮助我们创建扩展点，让框架的使用者在不修改源码的情况下，基于扩展点定制化框架功能。**

# 适配器模式

在Spring MVC中，定义一个Controller最常用的方式是，通过@Controller注解来标记某个类是Controller类，通过@RequesMapping注解来标记函数对应的URL

不过，我们还可以通过让类实现Controller接口或者Servlet接口，来定义一个Controller。

**针对这三种定义方式，我写了三段示例代码，如下所示：**

```java
// 方法一：通过@Controller、@RequestMapping来定义
@Controller
public class DemoController {
    @RequestMapping("/FlyFish")
    public ModelAndView getEmployeeName() {
        ModelAndView model = new ModelAndView("FlyFish");        
        model.addObject("message", "FlyFish");       
        return model; 
    }  
}

// 方法二：实现Controller接口 + xml配置文件:配置DemoController与URL的对应关系
public class DemoController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        ModelAndView model = new ModelAndView("FlyFish");
        model.addObject("message", "FlyFish");
        return model;
    }
}

// 方法三：实现Servlet接口 + xml配置文件:配置DemoController类与URL的对应关系
public class DemoServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    this.doPost(req, resp);
  }
  
  @Override
  protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    resp.getWriter().write("Hello World.");
  }
}
```

在应用启动的时候，Spring容器会加载这些Controller类，并且解析出URL对应的处理函数，封装成Handler对象，存储到HandlerMapping对象中。当有请求到来的时候，DispatcherServlet从HanderMapping中，查找请求URL对应的Handler，然后调用执行Handler对应的函数代码，最后将执行结果返回给客户端。

但是，不同方式定义的Controller，其函数的定义（函数名、入参、返回值等）是不统一的。

DispatcherServlet调用的是service()方法，DispatcherServlet需要根据不同类型的Controller，调用不同的函数。

Spring利用适配器模式，我们将不同方式定义的Controller类中的函数，适配为统一的函数定义。

**我们再具体看下Spring的代码实现。**

Spring定义了统一的接口HandlerAdapter，并且对每种Controller定义了对应的适配器类。

这些适配器类包括：AnnotationMethodHandlerAdapter、SimpleControllerHandlerAdapter、SimpleServletHandlerAdapter等。

![](https://img-blog.csdnimg.cn/6e53e7f9631c4257872b2d6b53c36185.png)

在DispatcherServlet类中，我们就不需要区分对待不同的Controller对象了，统一调用HandlerAdapter的handle()函数就可以了

# 最后

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

微信搜索：月伴飞鱼，交个朋友

公众号后台回复666，可以获得免费电子书籍