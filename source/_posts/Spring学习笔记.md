---
title: Spring学习笔记I
date: 2018-03-13 10:47:30
tags:
    - Java
    - Spring
---



#### 自动装配bean

* 创建接口

* 实现接口，通过```@Component```注解将实现标注为组件类

  > bean的ID默认为类名将第一个字母小写，比如
  >
  > ``` java
  > @Component
  > public class TestComponent implement TestInterface {
  >     ...
  > }
  > ```
  >
  > bean的ID就是testComponent
  >
  > 也可以自己指定bean ID，```@Component("testComponent2")```

* 通过```@ComponentScan```注解扫描组件

  > 默认扫描相同的包
  >
  > 如果想扫描不同的包，可以通过```@ComponentScan("otherPackage")```实现
  >
  > 想扫描多个包，可以使用```@ComponentScan(basePackages={"otherPackage1", "otherPackage2"})```，但这样不是类型安全的，不利于重构，通过```@ComponentScan(basePackageClasses={otherBaseClass.class, otherBaseClass.class})```更加方便重构

* 通过```@Autowired```自动装配bean

  > 实际就是将bean实例化，根据被注解对象尝试找到需要实例化的bean并传入被注解对象；如果没找到需要实例化的bean则会抛出一个异常，通过```@Autowired(required=false)```可以避免异常的抛出；如果找到多个可实例化的bean，也会抛出异常
  >
  > ```@Autowired```可以用在构造器、方法或者变量上


#### 通过Java代码手动装配bean

* 手动装配时，不需要和自动装配一样，使用```@Component @ComponentScan @Autowired```，而是通过```@Bean```注解，告诉Spring这个方法会返回一个对象，需要将这个对象注册到上下文中

  > 同样的，bean ID 默认为类名，第一个字母小写
  >
  > 可以通过```@Bean(name="beanName")```命名

* 如果一个bean对另外一个有依赖，会自动调用被依赖的bean的实例，**默认情况下，Spring的bean都是单例的**

  > ```java
  > @Bean
  > public TestClass testClass () {
  >     return new TestClass(testClass2());
  > }
  > // 这样会自动使用testClass2这个bean
  > ```

* 更好的方法

  > ```java
  > @Bean
  > public TestClass testClass(TestClass2 testClass2) {
  >     return new TestClass(testClass);
  > }
  > ```
  >
  > 这种方法对依赖的bean的创建方式没有限制，这样的话，被依赖的bean可以通过手动装配，也可以自动装配产生



  _可以看到，自动装配的步骤比较清晰，定义 -> 扫描 -> 装配，而手动装配时，由于没有自动扫描，只有一个```@Bean```注解，完成bean的定义和装配_


#### 解决自动装配时找到多个满足条件的bean的问题

* 通过在bean上增加```@Primary```来标示首选bean，但是```@Primary```是可以同时加在多个bean上的，这样并不能完全解决问题

* 在自动装配```@Autowired```时，通过```@Qualifier("beanId")```来指定具体的bean，这样做的缺点是，一旦重构导致bean ID发生改变，这个Qualifier就会失效

* 通过创建自定义的限定符，可以让Qualifier注解不再依赖bean ID，做法是在bean声明上同样增加```@Qualifier("name")```注解

  > ```java
  > @Component
  > @Qualifier("qual01")
  > public class Test implements TestInterface {...}
  > 
  > @Autowired
  > @Qualifier("qual01")
  > public void setTest(Test test) {
  >     this.test = test
  > }
  > ```

* 这样做也会有新的问题，Qualifier指定的名字是可以相同的，如果两个bean都有相同的Qualifier名字，那么还是会产生问题，解决这个问题的办法是多增加几个Qualifier

  > 由于Java不支持同一个条目上用多个没有```@Repeatable```注解的相同类型的注解，而```@Qualifier```刚好没有添加```@Repeatable```注解，所以这里需要自己创建自定义注解



#### bean的作用域

* Spring可以给予四种作用域创建bean

  * 单例（Singleton），整个应用中只创建一个bean实例（**默认**)

  * 原型（Prototype），每次注入或者通过Spring获取上下文的时候都会创建一个bean实例

  * 会话（Session），为每个会话创建一个bean实例

  * 请求（Request），为每个请求创建一个bean实例

    > 后两种都是用在Web应用中

* 使用方法

  > 将```@Scope```注解和```@Component```或```@Bean```一起使用，申明bean的作用域
  >
  > ```java
  > @Component
  > @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
  > public class Test { ... }
  > // 申明prototype作用域，也可以使用@Scope("prototype")
  > ```



#### 将外部属性注入Spring

* ```@PropertySource```可以实现将外部值注入到Spring上下文

  > ```java
  > @Configuration
  > @PropertySource("classpath:/com/test/app.properties")
  > public class TestConfig {
  >     @Autowired
  >     Environment env;
  >     
  >     @Bean
  >     public Test test() {
  >         return new Test(
  >             env.getProperty("test.title"),
  >             env.getProperty("test.title"))
  >     }
  > }
  > ```

* 通过```getProterty()```可以获取```env```中的属性，这个方法可以通过重载设置默认值和自动转换为新类型，不设置默认值且这个属性不存在的话，获取到的是```null```

  > ```getRequiredProperty()```获取值则表明这个属性必须被定义，否则会抛出异常

* Spring也支持通过占位符实现外部属性注入，用法为```${value}```，同时需要配置一个```PropertySourcesPlaceholderConfigurer``` bean
