# 1. Spring bean注解

* @Component, @Service, @Controller, @Repository是spring注解，注解后可以被spring框架所扫描并注入到spring容器来进行管理

* @Component是通用注解，其他三个注解是这个注解的拓展，并且具有了特定的功能

* @Repository注解在持久层中，具有将数据库操作抛出的原生异常翻译转化为spring的持久层异常的功能。

* @Controller层是spring-mvc的注解，具有将请求进行转发，重定向的功能。

* @Service层是业务逻辑层注解，这个注解只是标注该类处于业务逻辑层。

# 2. Spring通知支持顺序

![image-20230823174927922](../../AppData/Roaming/Typora/typora-user-images/image-20230823174927922.png)

不同版本的通知执行顺序不一致

---

# 3.  Spring MVC属于Spring的一个模块

Spring MVC 是 Spring 中的一个很重要的模块，主要赋予 Spring 快速构建 MVC 架构的 Web 程序的能力。MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

---

# 4. Spring资源路径

spring.web.resources.static-locations：根据官网的描述和实际效果，可以理解为实际静态文件地址，也就是静态文件URL后，匹配的实际静态文件。Springboot默认为：

* classpath:/META-INF/resources/
* classpath:/resources/
* classpath:/static/
* classpath:/public/

---

# 5. IOC 和 工厂模式的区别

IOC（Inversion of Control）和工厂模式是两个不同的概念，它们在软件设计和开发中有不同的用途和目的。

1. IOC（控制反转）：
   - IOC是一种设计原则，它强调将控制权从应用程序的代码中转移到外部容器或框架。这意味着应用程序的组件不再负责管理它们的依赖关系，而是将这些依赖关系交给容器或框架进行管理。
   - IOC的目的是实现松耦合，提高代码的可维护性和可测试性。它可以通过依赖注入（DI）来实现，其中组件的依赖关系由外部容器注入，而不是在组件内部硬编码。
   - IOC的实现方式包括依赖注入容器（如Spring Framework）和服务定位器等。
2. 工厂模式：
   - 工厂模式是一种创建型设计模式，用于创建对象的实例。它将对象的实例化和创建过程封装在一个工厂类中，客户端代码通过工厂类来获取所需的对象，而不需要直接调用构造函数。
   - 工厂模式有多种变体，包括简单工厂、工厂方法和抽象工厂等。这些变体允许根据需求创建不同类型的对象。
   - 工厂模式的主要目的是将对象的创建与客户端代码分离，以提供更大的灵活性和可维护性。

区别：

- IOC是一种设计原则，关注的是控制权的转移和依赖关系的管理，以降低组件之间的耦合度，提高可维护性。
- 工厂模式是一种设计模式，关注的是对象的创建和实例化，目的是将创建过程封装起来，提供更灵活的对象创建方式。
- IOC通常用于解决依赖管理和松耦合的问题，而工厂模式用于对象的创建和选择。

虽然它们有不同的用途，但在实际应用中，IOC容器（如Spring的应用上下文）通常使用工厂模式来管理和创建对象，以便更好地实现IOC原则。这意味着工厂模式可以用于IOC的实现。

---



# 6. JWT 的组成

JWT，全称JSON Web Token，是一种开放标准（RFC 7519），它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。具体来说，JWT由三部分组成：头部（Header）、有效载荷（Payload）和签名（Signature）。

头部部分包含两个主要的元素：alg和typ。alg代表签名算法，默认是HS256；typ则代表token的类型，一般JWT默认为JWT。

有效载荷，通常被称为payload，是存放有效信息的地方。这部分包含了关于用户或其他实体的重要信息。

最后一部分是签名，用于验证消息的完整性和来源。



---

# 7. Spring三级缓存

Spring中的一级缓存名为singletonObjects，二级缓存名为earlySingletonObjects，三级缓存名为singletonFactories，除了一级缓存是[ConcurrentHashMap](https://so.csdn.net/so/search?q=ConcurrentHashMap&spm=1001.2101.3001.7020)之外，二级缓存和三级缓存都是HashMap。它们的定义是在DefaultSingletonBeanRegistry类中。

Bean在这三个缓存之间的流转顺序为（存在循环引用）：

1. 通过反射创建Bean实例。是单例Bean，并且IoC容器允许Bean之间循环引用，保存到三级缓存中。
2. 当发生了循环引用时，从三级缓存中取出Bean对应的ObjectFactory实例，调用其getObject方法，来获取早期曝光Bean，从三级缓存中移除，保存到二级缓存中。
3. Bean初始化完成，生命周期的相关方法执行完毕，保存到一级缓存中，从二级缓存以及三级缓存中移除。

Bean在这三个缓存之间的流转顺序为（没有循环引用）：

1. 通过反射创建Bean实例。是单例Bean，并且IoC容器允许Bean之间循环引用，保存到三级缓存中。
2. Bean初始化完成，生命周期的相关方法执行完毕，保存到一级缓存中，从二级缓存以及三级缓存中移除。

#### 总结

通过以上分析，我们可以得知Bean在一级缓存、二级缓存、三级缓存中的流转顺序为：三级缓存->二级缓存->一级缓存。但是并不是所有Bean都会经历这个过程，例如对于原型Bean(Prototype)，IoC容器不会将其保存到任何一个缓存中的，另外即便是单例Bean(Singleton)，如果没有循环引用关系，也不会被保存到二级缓存中的。

---

# 8. SpringBoot中内嵌服务器

在Spring Boot中，你可以内嵌多种服务器来运行你的应用程序，而不需要部署到外部的Servlet容器中。以下是Spring Boot支持的一些内嵌服务器：

1. **Tomcat**: Tomcat是Apache的一个开源Servlet容器，也是Spring Boot默认的内嵌服务器。当你使用`spring-boot-starter-web`时，默认情况下就会使用Tomcat。
2. **Jetty**: Jetty是一个高性能、可扩展的Servlet容器和Java组件/框架。你可以通过添加`spring-boot-starter-jetty`依赖来在Spring Boot项目中使用Jetty。
3. **Undertow**: Undertow是另一个高性能的Servlet容器，它支持Servlet 3.1、JSP 2.3和EL 3.0规范。通过添加`spring-boot-starter-undertow`依赖，你可以在Spring Boot项目中使用Undertow。

---

