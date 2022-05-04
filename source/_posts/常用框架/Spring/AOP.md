---
title: AOP
categories: 
- 常用框架
- Spring
---

AOP的核心是将横切关注点从业务逻辑中抽离出来，只需要关注业务主体逻辑即可，无需关注通用逻辑（日志、埋点、事务等通用的逻辑）。

**AOP术语**

1. 连接点（JoinPoint）

   程序执行的某个特定位置，例如：类初始化前、类初始化后、类的某个方法执行前、类的某个方法正常执行后、类的某个方法抛出异常后等。一个类中或者一段代码中具有边界位置特性的点，这些点就叫连接点。总而言之，连接点指程序运行时允许插入切面的一个点，可以是一个函数、一个包路径、一个类、或者抛出的异常。

2. 切点（PointCut）

   每个类中都有多个连接点，比如一个类中有两个方法，那么这两个方法都是连接点。连接点是程序中客观存在的事物，切点用于确定特定的连接点。连接点相当于数据库中的记录，切点相当于数据库查询的查询条件。一个切点可以匹配多个连接点。

3. 增强（Advice）

   增强是织入到目标类连接点上的一段代码，定义了在切点周围所要做的操作。

4. 目标对象（Target）

   增强逻辑的织入目标类。如果不使用AOP，在目标业务类的业务方法需要自己实现所有的逻辑，在使用AOP后，我们只需要关注非横切逻辑的核心业务逻辑，而像日志监控、埋点等横切逻辑只需要使用AOP织入到连接点上即可。

5. 引入（Introduction）

   引入是一种特殊的增强，他可以为类添加一些属性或者方法。即使一个类原本没有实现某个接口，通过AOP的引入功能，可以动态的为业务类添加接口的实现逻辑，使业务类成为这个接口的实现类

6. 织入（Weaving）

   织入是将增强添加到目标类具体连接点上的过程

7. 代理（Proxy）

   一个类被代理类织入增强之后产生一个结果类，其融合了原类和增强逻辑的代理类

8. 切面（Aspect）

   切面由横切点的处理逻辑和连接点的定义构成。简而言之，切面定义了在什么时间、什么地点、做什么事。

**Spring AOP使用实例**

需求背景：为controller层方法添加请求日志监控，通过使用Spring-AOP，编写LogAspect来实现。

```java
@Component
@Aspect
@Slf4j
public class LogAspect {

    /**
     * 定义切点
     *
     */
    @Pointcut("execution(* edu.lyuconl.controller..*.*(..))")
    public void controllerAspect() {}


    /**
     * 前置增强：目标方法执行之前执行
     *
     * @param jp 连接点
     */
    @Before("execution(* edu.lyuconl.controller.*.*(..))") // 所有controller包下面的所有方法的所有参数
    public void beforeMethod(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        log.info("[前置增强] the method [" + methodName + "], begin with " + JSON.toJSONString(jp.getArgs()));
    }

    /**
     * 后置增强：目标方法执行完毕之后执行，不管是否发生异常
     *
     * @param jp 连接点
     */
    @After("execution(* edu.lyuconl.controller.*.*(..))") // 所有controller包下面的所有方法的所有参数
    public void afterMethod(JoinPoint jp) {
        String methodName = jp.getSignature().getName();
        log.info("[后置增强] the method [" + methodName + "] finished");
    }


    /**
     * 返回增强，目标方法正常执行完毕时执行
     *
     * @param jp 连接点
     * @param result 目标方法的返回值
     */
    @AfterReturning(value = "execution(* edu.lyuconl.controller.*.*(..))", returning = "result")
    public void afterReturningMethod(JoinPoint jp, Object result) {
        String methodName = jp.getSignature().getName();
        log.info("[返回增强] the method [" + methodName + "] end with [" + result + "]");
    }

    /**
     * 异常增强，目标方法发生异常时执行
     *
     * @param jp 连接点
     * @param e 目标方法发生异常的类型
     */
    @AfterThrowing(value = "execution(* edu.lyuconl.controller.*.*(..))", throwing = "e")
    public void afterThrowingMethod(JoinPoint jp, Exception e) {
        String methodName = jp.getSignature().getName();
        log.error("[异常增强] the method [" + methodName + "], occurs exception: ", e);
    }

    /**
     *
     * 环绕增强，目标方法执行前后分别执行一些代码，发生异常的时候执行另外一些代码
     *
     * @param jp 连接点
     * @return 增强后的返回值
     */
    @Around("controllerAspect()")
    public Object aroundMethod(ProceedingJoinPoint jp) {
        String methodName = jp.getSignature().getName();
        Object result = null;
        try {
            Object[] args = jp.getArgs();
            log.info("[环绕增强]-->[前置增强], the method [" + methodName + "], begin with " + Arrays.toString(args));
            result = jp.proceed();
            log.info("[环绕增强]-->[返回增强], the method [" + methodName + "], end with [" + result + "]");
        } catch (Throwable e) {
            e.printStackTrace();
            log.error("[环绕增强]-->[异常增强], the method [" + methodName + "], occurs exception: ", e);
        } finally {
            log.info("[环绕增强]-->[异常增强]-->[finally], the method [" + methodName + "], finally!");
        }
        log.info("[环绕增强]-->[后置增强], ------- end! --------");
        return result;
    }
}
```

**通过注解指定切点**

```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AspectTest {
}
```

```java
@Component
@Aspect
@Slf4j
public class LogAspect {

    /**
     * 定义切点
     *
     */
    @Pointcut("@annotation(edu.lyuconl.annotation.AspectTest)")
    public void controllerAspect() {}

    /**
     *
     * 环绕增强，目标方法执行前后分别执行一些代码，发生异常的时候执行另外一些代码
     *
     * @param jp 连接点
     * @return 增强后的返回值
     */
    @Around("controllerAspect()")
    public Object aroundMethod(ProceedingJoinPoint jp) {
        String methodName = jp.getSignature().getName();
        Object result = null;
        try {
            Object[] args = jp.getArgs();
            log.info("[环绕增强]-->[前置增强], the method [" + methodName + "], begin with " + Arrays.toString(args));
            result = jp.proceed();
            log.info("[环绕增强]-->[返回增强], the method [" + methodName + "], end with [" + result + "]");
        } catch (Throwable e) {
            e.printStackTrace();
            log.error("[环绕增强]-->[异常增强], the method [" + methodName + "], occurs exception: ", e);
        } finally {
            log.info("[环绕增强]-->[异常增强]-->[finally], the method [" + methodName + "], finally!");
        }
        log.info("[环绕增强]-->[后置增强], ------- end! --------");
        return result;
    }
}
```

