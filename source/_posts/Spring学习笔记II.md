```
---
title: Spring学习笔记II
date: 2018-03-20 13:20:30
tags:
    - Java
    - Spring
---
```



#### 面向切面编程(AOP)

* Spring支持5种类型的通知，**_Before, After, After-returning, After-throwing, Around_**，提供4种类型的AOP支持，**_基于代理的Spring AOP， POJO切面，@AspectJ注解驱动切面，注入式AspectJ切面_**


* Spring通过AOP容器在运行时为目标动态地创建一个代理对象织入(weaving)切面，即在运行时才创建代理对象，且只支持方法连接点，如果需要方法拦截之外的连接点功能，则需要使用Aspect


* Spring 文档中对AOP相关的概念做了介绍

  > - *Aspect*: a modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented using regular classes (the [schema-based approach](https://docs.spring.io/spring/docs/5.0.9.RELEASE/spring-framework-reference/core.html#aop-schema)) or regular classes annotated with the `@Aspect` annotation (the [`@AspectJ` style](https://docs.spring.io/spring/docs/5.0.9.RELEASE/spring-framework-reference/core.html#aop-ataspectj)).
  > - *Join point*: a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point *always* represents a method execution.
  > - *Advice*: action taken by an aspect at a particular join point. Different types of advice include "around", "before" and "after" advice. (Advice types are discussed below.) Many AOP frameworks, including Spring, model an advice as an *interceptor*, maintaining a chain of interceptors *around* the join point.
  > - *Pointcut*: a predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
  > - *Introduction*: declaring additional methods or fields on behalf of a type. Spring AOP allows you to introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an `IsModified` interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
  > - *Target object*: object being advised by one or more aspects. Also referred to as the *advised* object. Since Spring AOP is implemented using runtime proxies, this object will always be a *proxied* object.
  > - *AOP proxy*: an object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy will be a JDK dynamic proxy or a CGLIB proxy.
  > - *Weaving*: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.
  >
  > Types of advice:
  >
  > - *Before advice*: Advice that executes before a join point, but which does not have the ability to prevent execution flow proceeding to the join point (unless it throws an exception).
  > - *After returning advice*: Advice to be executed after a join point completes normally: for example, if a method returns without throwing an exception.
  > - *After throwing advice*: Advice to be executed if a method exits by throwing an exception.
  > - *After (finally) advice*: Advice to be executed regardless of the means by which a join point exits (normal or exceptional return).
  > - *Around advice*: Advice that surrounds a join point such as a method invocation. This is the most powerful kind of advice. Around advice can perform custom behavior before and after the method invocation. It is also responsible for choosing whether to proceed to the join point or to shortcut the advised method execution by returning its own return value or throwing an exception.

  _AOP的术语较多，需要多理解_


* 编写切点(Pointcut)

  > 切点表达式，设置切点方法执行时触发通知(advise)，`execution(* com.Test.exec(...))`，其中execution代表方法执行时触发，第一个*表示返回任意类型， 中间的com.Test表示方法所属的类，exec表示方法, ...表示使用任意参数
  >
  > 切点表达式里还可以通过&&、||和!来组合命令, `execution(* com.Test.exec(...) && within(com.Test1.*))`表示com.Test包内的任何方法被调用时执行表达式
  >
  > 还可以使用`execution(* com.Test.exec(...) && bean('test2'))`表示切面的通知会被编织到ID为test2的bean中


* 定义切面(Aspect)

  > ```java
  > @Aspect
  > public class AspectDemo {
  >     
  >     @Pointcut(*execution(** com.Text.exec(...))*)
  >     public void pointDemo() {}
  >     
  >     @Before(* pointDemo() *)
  >     public void execBefore() { ... }
  > }
  > ```
  >
  > 利用`@Pointcut`注解设置切点表达式，如上面的代码，设置了方法pointDemo为切点表达式，这样就可以在别的地方重用，pointDemo是一个空函数，只做标识用，供`@Pointcut`注解依附

  **_整个切面表达式的语法含义比较明确，就是before|after某个函数之前，执行这个被注解的通知_**

* 自动代理

  > 使用`@EnableAspectJAutoProxy`注解可以实现自动代理，自动使用`@Aspect`注解的bean创建一个代理
  >
  > ```java
  > @EnableAspectJAutoProxy
  > @ComponentScan
  > @Configuration
  > public class AppConfig {
  >     
  >     @Bean
  >     public AspectDemo aspectDemo() {
  >         return new AspectDemo();
  >     }
  > }
  > // 使用手动注入的方式创建了切面的bean，同时通过@EnableAspectJAutoProxy注解启动自动代理
  > // 如果不使用自动代理的注解，那么这个切面会被当作普通的bean处理，不会被视为切面
  > ```



* 带参数的切面表达式

  > 如果需要在表达式中带参数，则表达式要写成类似`@Pointcut(* execution(**  com.Test.exec(int)) && args(inputNumber))`，exec函数中写接受参数的类型，args中写参数名
  >
  > 实际上，args(inputNumber)的意思是exec()的参数也会传递到通知方法(advise)中，这样就可以做到切点到通知方法的参数传递
  >
  > ```java
  > @Aspect
  > public class AspectDemo {
  >     @Pointcut(*execution(* com.Test.exec(int)) && args(inputNumber) *)
  >     public void pointDemo() {}
  >     
  >     @Before(* poingDemo(inputNumber) *)
  >     public void execBefore(int inputNumber) {
  >         ...
  >     }
  > }
  > ```



* 引入新功能 (Introductions)

  > 通过`@DeclareParents`注解，将新的接口引入到bean中，语法是`@DeclareParents(value="com.Test.*", defaultImpl=newInterfaceImpl.class)`
  >
  > 可以看出来，这个注解的意思就是为value增加一个新的接口，这里引入的是newInterface
  >
  > ```java
  > // 文档中的例子
  > @Aspect
  > public class UsageTracking {
  > 
  >     @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
  >     public static UsageTracked mixin;
  > 
  >     @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
  >     public void recordUsage(UsageTracked usageTracked) {
  >         usageTracked.incrementUseCount();
  >     }
  > 
  > }
  > ```
