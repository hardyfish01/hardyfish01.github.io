---
title: OOP规约
categories: 
- 阿里Java开发手册
- 编程规约
---

1.避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用类名来访问即可。

2.相同参数类型，相同业务含义，才可以使用 Java 的可变参数，避免使用 Object。 

> 说明:可变参数必须放置在参数列表的最后。(提倡同学们尽量不用可变参数编程)

3.外部正在调用或者二方库依赖的接口，不允许修改方法签名，避免对接口调用方产生 影响。

> 接口过时必须加@Deprecated 注解，并清晰地说明采用的新接口或者新服务是什么。

4.Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals。 

```java
正例:"test".equals(object);
反例:object.equals("test");
```

> 说明:推荐使用 java.util.Objects#equals(JDK7 引入的工具类)。

5.所有整型包装类对象之间值的比较，全部使用equals方法比较。

6.任何货币金额，均以最小货币单位且整型类型来进行存储。

**7.浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断。**

反例：

```java
float a = 1.0F - 0.9F;
float b = 0.9F - 0.8F;
if (a == b) {
// 预期进入此代码块，执行其它业务逻辑
// 但事实上 a==b 的结果为 false
}
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)) {
// 预期进入此代码块，执行其它业务逻辑
// 但事实上 equals 的结果为 false
}
```

正例：

```java
//(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
float a = 1.0F - 0.9F;
float b = 0.9F - 0.8F;
float diff = 1e-6F;
if (Math.abs(a - b) < diff) {
	System.out.println("true");
}
//(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
if (x.equals(y)) {
  System.out.println("true");
}
```

8.定义数据对象 DO 类时，属性类型要与数据库字段类型相匹配。

> 正例:数据库字段的 bigint 必须与类属性的 Long 类型相对应。
>
> 反例:某个案例的数据库表 id 字段定义类型 bigint unsigned，实际类对象属性为 Integer，随着 id 越来 越大，超过 Integer 的表示范围而溢出成为负数。

9.禁止使用构造方法 `BigDecimal(double)`的方式把 double 值转化为 BigDecimal 对象。 

> 说明:BigDecimal(double)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。
>
> 正例:优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了

```java
 BigDecimal recommend1 = new BigDecimal("0.1");
 BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```

10.所有的 POJO 类属性必须使用包装数据类型。

> 正例:数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险。 

11.RPC 方法的返回值和参数必须使用包装数据类型。 

> 包装数据类型 的 null 值，能够表示额外的信息，如:远程调用失败，异常退出。

12.所有的局部变量使用基本数据类型。

13.定义 DO/DTO/VO 等 POJO 类时，不要设定任何属性默认值。

14.序列化类新增属性时，请不要修改 serialVersionUID 字段，避免反序列失败;

如果 完全不兼容升级，避免反序列化混乱，那么请修改 serialVersionUID 值。

> 说明:注意 serialVersionUID 不一致会抛出序列化运行时异常。

15.构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在 init 方法中。

16.POJO 类必须写 toString 方法。

> 说明:在方法执行抛出异常时，可以直接调用 POJO 的 toString()方法打印其属性值，便于排查问题。

17.POJO类中禁止存在对应属性xxx的isXxx()和getXxx()方法。

> 框架在调用属性xxx的提取方法时，并不能确定哪个方法一定是被优先调用到。

18.使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有IndexOutOfBoundsException 的风险。

```java
String str = "a,b,c,,";
String[] ary = str.split(",");
// 预期大于 3，结果是 3 
System.out.println(ary.length);
```

19.当一个类有多个构造方法，或者多个同名方法，这些方法应该按顺序放置在一起，便 于阅读。

20.类内方法定义的顺序依次是：公有方法或保护方法 > 私有方法 > getter / setter 方法。

21.在getter/setter 方法中，不要增加业务逻辑，增加排查问题的难度。

22.循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。