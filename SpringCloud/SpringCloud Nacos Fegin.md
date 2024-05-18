# SpringCloud之Nacos配置管理、Feign、Gateway服务网关

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[陶然同学](https://taoran.blog.csdn.net/)![img](https://csdnimg.cn/release/blogv2/dist/pc/img/newUpTime2.png)已于 2022-06-27 17:29:40 修改

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes2.png)阅读量3.5k![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect2.png) 收藏 33

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/newHeart2023Active.png)点赞数23

文章标签： [nacos](https://so.csdn.net/so/search/s.do?q=nacos&t=all&o=vip&s=&l=&f=&viparticle=) [gateway](https://so.csdn.net/so/search/s.do?q=gateway&t=all&o=vip&s=&l=&f=&viparticle=) [feign](https://so.csdn.net/so/search/s.do?q=feign&t=all&o=vip&s=&l=&f=&viparticle=) [springcloud](https://so.csdn.net/so/search/s.do?q=springcloud&t=all&o=vip&s=&l=&f=&viparticle=) [微服务](https://so.csdn.net/so/search/s.do?q=微服务&t=all&o=vip&s=&l=&f=&viparticle=)

版权

> - **💂 个人主页:** [陶然同学](https://blog.csdn.net/weixin_45481821?spm=1000.2115.3001.5343)
> - **🤟 版权:** 本文由【陶然同学】原创、在CSDN首发、需要转载请联系博主
> - 💬 如果文章对你有帮助、**欢迎关注、点赞、收藏(一键三连)和订阅专栏哦**
> - 💅 **想寻找共同成长的小伙伴，请点击【\**\*\*\*\*\*\*\*\*\*\*\*\*[Java全栈开发社区](https://bbs.csdn.net/forums/TaoRan?category=0)\*\*\*\*\*\*\*\*\*\*\*\*\**】**

------

## 1.[Nacos](https://so.csdn.net/so/search?q=Nacos&spm=1001.2101.3001.7020)配置管理

Nacos除了可以做[注册中心](https://so.csdn.net/so/search?q=注册中心&spm=1001.2101.3001.7020)，同样可以做配置管理来使用。

### 1.1统一配置管理

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。我们需要一种统一配置管理方案，可以集中管理所有实例的配置。

![img](https://img-blog.csdnimg.cn/c280a5fced7c4465a87d7c3339d7588c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

Nacos一方面可以将配置集中管理，另一方可以在配置变更时，及时通知微服务，实现配置的[热更新](https://so.csdn.net/so/search?q=热更新&spm=1001.2101.3001.7020)。  

#### 1.1.1在nacos中添加配置文件

如何在nacos中管理配置呢？

![img](https://img-blog.csdnimg.cn/7971bc3f3fe34d8bb18e34bb4f644bee.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

然后在弹出的表单中，填写配置信息：

![img](https://img-blog.csdnimg.cn/b162da72caad44b18802ec293a5aae4f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

注意：项目的核心配置，需要热更新的配置才有放到nacos管理的必要。基本不会变更的一些配置还是保存在微服务本地比较好。  

#### 1.1.2从微服务拉取配置

微服务要拉取nacos中管理的配置，并且与本地的application.yml配置合并，才能完成项目启动。

但如果尚未读取application.yml，又如何得知nacos地址呢？

因此spring引入了一种新的配置文件：bootstrap.yaml文件，会在application.yml之前被读取，流程如下

![img](https://img-blog.csdnimg.cn/cb067dbdc46c417e83a715a166e3d894.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

1）引入nacos-config依赖

首先，在user-service服务中，引入nacos-config的客户端依赖：

```xml
<!--nacos配置管理依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2）添加bootstrap.yaml

然后，在user-service中添加一个bootstrap.yaml文件，内容如下：

```yaml
spring:
  application:
    name: userservice # 服务名称
  profiles:
    active: dev #开发环境，这里是dev 
  cloud:
    nacos:
      server-addr: localhost:8848 # Nacos地址
      config:
        file-extension: yaml # 文件后缀名
```

这里会根据spring.cloud.nacos.server-addr获取nacos地址，再根据

`${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`作为文件id，来读取配置。

本例中，就是去读取`userservice-dev.yaml`：

![img](https://img-blog.csdnimg.cn/db75c7b2aea44451820aaeff2c0a256f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

3）读取nacos配置

在user-service中的UserController中添加业务逻辑，读取pattern.dateformat配置：

![img](https://img-blog.csdnimg.cn/6c9f296f227c4d3994439eaf34c7fe96.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

完整代码：

```java
 
import cn.itcast.user.pojo.User;
import cn.itcast.user.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;
 
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
 
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
 
    @Autowired
    private UserService userService;
 
    @Value("${pattern.dateformat}")
    private String dateformat;
    
    @GetMapping("now")
    public String now(){
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateformat));
    }
    // ...略
}
```

 在页面访问，可以看到效果：

![img](https://img-blog.csdnimg.cn/63db652523fc4b4a911c757c7b8d67a1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_18,color_FFFFFF,t_70,g_se,x_16)

### 1.2配置热更新

我们最终的目的，是修改nacos中的配置后，微服务中无需重启即可让配置生效，也就是**配置热更新**。

实现配置热更新，可以使用两种方式：

#### 1.2.1方式一

在@Value注入的变量所在类上添加注解@RefreshScope：

![img](https://img-blog.csdnimg.cn/ab8365149b4b46ccb45f38497112f30d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 1.2.2方式二

使用@ConfigurationProperties注解代替@Value注解。

在user-service服务中，添加一个类，读取patterrn.dateformat属性：

```java
package cn.itcast.user.config;
 
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
 
@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
```

在UserController中使用这个类代替@Value：

![img](https://img-blog.csdnimg.cn/a4ca0ec976db44709d8681d0187be7ab.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

完整代码：

```java
package cn.itcast.user.web;
 
import cn.itcast.user.config.PatternProperties;
import cn.itcast.user.pojo.User;
import cn.itcast.user.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
 
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
 
    @Autowired
    private UserService userService;
 
    @Autowired
    private PatternProperties patternProperties;
 
    @GetMapping("now")
    public String now(){
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(patternProperties.getDateformat()));
    }
 
    // 略
}
```

### 1.3配置共享

其实微服务启动时，会去nacos读取多个配置文件，例如：

- `[spring.application.name]-[spring.profiles.active].yaml`，例如：userservice-dev.yaml
- `[spring.application.name].yaml`，例如：userservice.yaml

而`[spring.application.name].yaml`不包含环境，因此可以被多个环境共享。



下面我们通过案例来测试配置共享

**1.3.1添加一个环境共享配置**

我们在nacos中添加一个userservice.yaml文件：

![img](https://img-blog.csdnimg.cn/fc5633cd7b7c412e80fbde7fe787fb8d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 1.3.2在user-service中读取共享配置

在user-service服务中，修改PatternProperties类，读取新添加的属性：

![img](https://img-blog.csdnimg.cn/8354078fa06d4bb0a50162461ba2d12b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

在user-service服务中，修改UserController，添加一个方法：

![img](https://img-blog.csdnimg.cn/1d214195fa144e88aa3b77e2f0629fb5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 1.3.3运行两个UserApplication 使用不同profile

修改UserApplication2这个启动项，改变其profile值：

![img](https://img-blog.csdnimg.cn/dc8a9526db5643dd99645b4fcae53702.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

![img](https://img-blog.csdnimg.cn/e29c0f83a898446dbd765117833b358d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

这样，UserApplication(8081)使用的profile是dev，UserApplication2(8082)使用的profile是test。

启动UserApplication和UserApplication2

访问http://localhost:8081/user/prop，结果：

![img](https://img-blog.csdnimg.cn/9a35c5f5a6b1426bb439655e04055e95.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

访问http://localhost:8082/user/prop，结果：

![img](https://img-blog.csdnimg.cn/b1eefa8f1c4a4fdda0e94b6e157e367a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看出来，不管是dev，还是test环境，都读取到了envSharedValue这个属性的值。  

#### 1.3.4配置共享的优先级

当nacos、服务本地同时出现相同属性时，优先级有高低之分：

![img](https://img-blog.csdnimg.cn/a8ed1b5993304f7d90a1e5245026ce5b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1.4搭建Nacos集群

Nacos生产环境下一定要部署为集群状态，部署方式参考：

[SpringCloudAlibaba之Nacos集群搭建(详细)_陶然同学。的博客-CSDN博客💂 个人主页:陶然同学🤟 版权:本文由【陶然同学】原创、在CSDN首发、需要转载请联系博主💬 如果文章对你有帮助、欢迎关注、点赞、收藏(一键三连)和订阅专栏哦💅想寻找共同成长的小伙伴，请点击【Java全栈开发社区】1.集群结构图官方给出的Nacos集群图：其中包含3个nacos节点，然后一个负载均衡器代理3个Nacos。这里负载均衡器可以使用nginx。我们计划的集群结构：三个nacos节点的地址：节点ipportnacos1...![img](https://g.csdnimg.cn/static/logo/favicon32.ico)https://blog.csdn.net/weixin_45481821/article/details/123829626?spm=1001.2014.3001.5501](https://blog.csdn.net/weixin_45481821/article/details/123829626?spm=1001.2014.3001.5501)

## 2.Feign远程调用

先来看我们以前利用RestTemplate发起远程调用的代码：

![img](https://img-blog.csdnimg.cn/2512e9a57902499dbe3a82c2074d5d3c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

存在下面的问题：

•代码可读性差，编程体验不统一

•参数复杂URL难以维护



Feign是一个声明式的http客户端，官方地址：https://github.com/OpenFeign/feign

其作用就是帮助我们优雅的实现http请求的发送，解决上面提到的问题。

![img](https://img-blog.csdnimg.cn/3ea1657cdb1e4826b3b041574deb70c2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

### 2.1Feign替代RestTemplate

Fegin的使用步骤如下：

#### 2.1.1引入依赖

我们在order-service服务的pom文件中引入feign的依赖：

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2.1.2添加注解

在order-service的启动类添加注解开启Feign的功能：

![img](https://img-blog.csdnimg.cn/381b6f005ee546508c9d0c26b5046906.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 2.1.3编写Feign的客户端

在order-service中新建一个接口，内容如下：

```java
package cn.itcast.order.client;
 
import cn.itcast.order.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
 
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

这个客户端主要是基于SpringMVC的注解来声明远程调用的信息，比如：

- 服务名称：userservice
- 请求方式：GET
- 请求路径：/user/{id}
- 请求参数：Long id
- 返回值类型：User

这样，Feign就可以帮助我们发送http请求，无需自己使用RestTemplate来发送了。

**2.1.4测试**

修改order-service中的OrderService类中的queryOrderById方法，使用Feign客户端代替RestTemplate：

![img](https://img-blog.csdnimg.cn/d12452ca0d7140a68fc8d7d2fa331a65.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

是不是看起来优雅多了。  

#### 2.1.5总结

使用Feign的步骤：

① 引入依赖

② 添加@EnableFeignClients注解

③ 编写FeignClient接口

④ 使用FeignClient中定义的方法代替RestTemplate

**2.2自定义配置**

Feign可以支持很多的自定义配置，如下表所示：

| 类型                   | 作用             | 说明                                                   |
| :--------------------- | :--------------- | :----------------------------------------------------- |
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign. Contract        | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign. Retryer         | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的@Bean覆盖默认Bean即可。

**2.2.1配置文件方式**

基于配置文件修改feign的日志级别可以针对单个服务：

```YML
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

也可以针对所有服务：

```YML
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

而日志的级别分为四种：

- NONE：不记录任何日志信息，这是默认值。
- BASIC：仅记录请求的方法，URL以及响应状态码和执行时间
- HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息
- FULL：记录所有请求和响应的明细，包括头信息、请求体、元数据。

#### 2.2.2Java代码方式

也可以基于Java代码来修改日志级别，先声明一个类，然后声明一个Logger.Level的对象：

```java
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```

如果要**全局生效**，将其放到启动类的@EnableFeignClients这个注解中：

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) 
```

如果是**局部生效**，则把它放到对应的@FeignClient这个注解中：

```java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) 
```

### 2.3Feign使用优化

Feign底层发起http请求，依赖于其它的框架。其底层客户端实现包括：

•URLConnection：默认实现，不支持连接池

•Apache HttpClient ：支持连接池

•OKHttp：支持连接池



因此提高Feign的性能主要手段就是使用**连接池**代替默认的URLConnection。



这里我们用Apache的HttpClient来演示。

1）引入依赖

在order-service的pom文件中引入Apache的HttpClient依赖：

```XML
<!--httpClient的依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

2）配置连接池

在order-service的application.yml中添加配置：

```YMl
feign:
  client:
    config:
      default: # default全局的配置
        loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息
  httpclient:
    enabled: true # 开启feign对HttpClient的支持
    max-connections: 200 # 最大的连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
```

接下来，在FeignClientFactoryBean中的loadBalance方法中打断点：

![img](https://img-blog.csdnimg.cn/01436c305cef43579981b75d766433e1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

Debug方式启动order-service服务，可以看到这里的client，底层就是Apache HttpClient：

![img](https://img-blog.csdnimg.cn/30ee73ec5e9044a6b21e6533c080e8c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

总结，Feign的优化：

1.日志级别尽量用basic

2.使用HttpClient或OKHttp代替URLConnection

① 引入feign-httpClient依赖

② 配置文件开启httpClient功能，设置连接池参数

### 2.4最佳实践

所谓最近实践，就是使用过程中总结的经验，最好的一种使用方式。

自习观察可以发现，Feign的客户端与服务提供者的controller代码非常相似：

feign客户端：

![img](https://img-blog.csdnimg.cn/17bb3ddc83064da3adc833c0a5bd216f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

UserController：

![img](https://img-blog.csdnimg.cn/0bbc39d53e294d48aaed1f51b5b233f8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

有没有一种办法简化这种重复的代码编写呢？  

#### 2.4.1继承方式

一样的代码可以通过继承来共享：

1）定义一个API接口，利用定义方法，并基于SpringMVC注解做声明。

2）Feign客户端和Controller都集成改接口

![img](https://img-blog.csdnimg.cn/d9d7788ba6f643df89faf9cdc012148d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

优点：

- 简单
- 实现了代码共享

缺点：

- 服务提供方、服务消费方紧耦合
- 参数列表中的注解映射并不会继承，因此Controller中必须再次声明方法、参数列表、注解

#### 2.4.2抽取方式

将Feign的Client抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。

例如，将UserClient、User、Feign的默认配置都抽取到一个feign-api包中，所有微服务引用该依赖包，即可直接使用。

![img](https://img-blog.csdnimg.cn/aafdc937e6544d049f506a64e401e626.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 2.4.3实现基于抽取的最佳实践

1）抽取

首先创建一个module，命名为feign-api：

![img](https://img-blog.csdnimg.cn/459bafd6c75548deb42d88cd7d05b2d5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

项目结构：

![img](https://img-blog.csdnimg.cn/6e67ff927523472f95698d20cb6a8952.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_11,color_FFFFFF,t_70,g_se,x_16)

在feign-api中然后引入feign的starter依赖

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

然后，order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中

![img](https://img-blog.csdnimg.cn/60f4eaa7d9c447faa3f650c2ff0c88c5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_14,color_FFFFFF,t_70,g_se,x_16)

2）在order-service中使用feign-api

首先，删除order-service中的UserClient、User、DefaultFeignConfiguration等类或接口。



在order-service的pom文件中中引入feign-api的依赖：

```XML
<dependency>
    <groupId>cn.itcast.demo</groupId>
    <artifactId>feign-api</artifactId>
    <version>1.0</version>
</dependency>
```

修改order-service中的所有与上述三个组件有关的导包部分，改成导入feign-api中的包

3）重启测试

重启后，发现服务报错了：

![img](https://img-blog.csdnimg.cn/248b67ff21574395be475843e73a2d8d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

这是因为UserClient现在在cn.itcast.feign.clients包下，

而order-service的@EnableFeignClients注解是在cn.itcast.order包下，不在同一个包，无法扫描到UserClient。

4）解决扫描包问题

方式一：

指定Feign应该扫描的包：

```java
@EnableFeignClients(basePackages = "cn.itcast.feign.clients")
```

方式二：

指定需要加载的Client接口：

```java
@EnableFeignClients(clients = {UserClient.class})
```

## 3.Gateway服务网关

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等响应式编程和事件流技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

### 3.1为什么需要网关

Gateway网关是我们服务的守门神，所有微服务的统一入口。

网关的**核心功能特性**：

- 请求路由
- 权限控制
- 限流

架构图：

![img](https://img-blog.csdnimg.cn/d76931a6f4e042d3b607fc9b4016626b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

**权限控制**：网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**路由和负载均衡**：一切请求都必须先经过gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**限流**：当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大。



在SpringCloud中网关的实现包括两种：

- gateway
- zuul

Zuul是基于Servlet的实现，属于阻塞式编程。而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程的实现，具备更好的性能。

### 3.2gateway快速入门

下面，我们就演示下网关的基本路由功能。基本步骤如下：

1. 创建SpringBoot工程gateway，引入网关依赖
2. 编写启动类
3. 编写基础配置和路由规则
4. 启动网关服务进行测试

#### 3.2.1创建gateway服务 引入依赖

创建服务：

![img](https://img-blog.csdnimg.cn/8f735dec327a42a183da81f50df8f457.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

引入依赖：

```XML
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 3.2.1编写启动类

```java
package cn.itcast.gateway;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
 
@SpringBootApplication
public class GatewayApplication {
 
	public static void main(String[] args) {
		SpringApplication.run(GatewayApplication.class, args);
	}
}
```

#### 3.2.3编写基础配置和路由规则

创建application.yml文件，内容如下：

```yaml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
```

我们将符合`Path` 规则的一切请求，都代理到 `uri`参数指定的地址。

本例中，我们将 `/user/**`开头的请求，代理到`lb://userservice`，lb是负载均衡，根据服务名拉取服务列表，实现负载均衡。

#### 3.2.4重启测试

重启网关，访问http://localhost:10010/user/1时，符合`/user/**`规则，请求转发到uri：http://userservice/user/1，得到了结果：

![img](https://img-blog.csdnimg.cn/ebf518323f99489e8f54e5255bbe8870.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_14,color_FFFFFF,t_70,g_se,x_16)

#### 3.2.5网关路由的流程图

整个访问的流程如下：

![img](https://img-blog.csdnimg.cn/2d4bcd15a9b94a88b2d4891f426ed495.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

总结：

网关搭建步骤：

1. 创建项目，引入nacos服务发现和gateway依赖
2. 配置application.yml，包括服务基本信息、nacos地址、路由

路由配置包括：

1. 路由id：路由的唯一标示
2. 路由目标（uri）：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. 路由断言（predicates）：判断路由的规则，
4. 路由过滤器（filters）：对请求或响应做处理



接下来，就重点来学习路由断言和路由过滤器的详细知识

**3.3断言工厂**



我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件

例如Path=/user/**是按照路径匹配，这个规则是由

`org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory`类来

处理的，像这样的断言工厂在SpringCloudGateway还有十几个:

| **名称**   | **说明**                       | **示例**                                                     |
| :--------- | :----------------------------- | :----------------------------------------------------------- |
| After      | 是某个时间点后的请求           | - After=2037-01-20T17:42:47.789-07:00[America/Denver]        |
| Before     | 是某个时间点之前的请求         | - Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]        |
| Between    | 是某两个时间点之前的请求       | - Between=2037-01-20T17:42:47.789-07:00[America/Denver], 2037-01-21T17:42:47.789-07:00[America/Denver] |
| Cookie     | 请求必须包含某些cookie         | - Cookie=chocolate, ch.p                                     |
| Header     | 请求必须包含某些header         | - Header=X-Request-Id, \d+                                   |
| Host       | 请求必须是访问某个host（域名） | - Host=**.somehost.org,**.anotherhost.org                    |
| Method     | 请求方式必须是指定方式         | - Method=GET,POST                                            |
| Path       | 请求路径必须符合指定规则       | - Path=/red/{segment},/blue/**                               |
| Query      | 请求参数必须包含指定参数       | - Query=name, Jack或者- Query=name                           |
| RemoteAddr | 请求者的ip必须是指定范围       | - RemoteAddr=192.168.1.1/24                                  |
| Weight     | 权重处理                       |                                                              |



我们只需要掌握Path这种路由工程就可以了。

**3.4过滤器工厂**

GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理：

![img](https://img-blog.csdnimg.cn/128d91765ad24e3ca4c6910b8481a4d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 3.4.1路由过滤器的种类

Spring提供了31种不同的路由过滤器工厂。例如：

| **名称**             | **说明**                     |
| :------------------- | :--------------------------- |
| AddRequestHeader     | 给当前请求添加一个请求头     |
| RemoveRequestHeader  | 移除请求中的一个请求头       |
| AddResponseHeader    | 给响应结果中添加一个响应头   |
| RemoveResponseHeader | 从响应结果中移除有一个响应头 |
| RequestRateLimiter   | 限制请求的流量               |

**3.4.2请求头过滤器**



下面我们以AddRequestHeader 为例来讲解。

> **需求**：给所有进入userservice的请求添加一个请求头：Truth=itcast is freaking awesome!



只需要修改gateway服务的application.yml文件，添加路由过滤即可：

```YML
spring:
  cloud:
    gateway:
      routes:
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/** 
        filters: # 过滤器
        - AddRequestHeader=Truth, Itcast is freaking awesome! # 添加请求头
```

当前过滤器写在userservice路由下，因此仅仅对访问userservice的请求有效。

#### 3.4.3默认过滤器

如果要对所有的路由都生效，则可以将过滤器工厂写到default下。格式如下：

```YML
spring:
  cloud:
    gateway:
      routes:
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/**
      default-filters: # 默认过滤项
      - AddRequestHeader=Truth, Itcast is freaking awesome! 
```

#### 3.4.4总结

过滤器的作用是什么？

① 对路由的请求或响应做加工处理，比如添加请求头

② 配置在路由下的过滤器只对当前路由的请求生效

defaultFilters的作用是什么？

① 对所有路由都生效的过滤器

**3.5全局过滤器**

上一节学习的过滤器，网关提供了31种，但每一种过滤器的作用都是固定的。如果我们希望拦截请求，做自己的业务逻辑则没办法实现。

#### 3.5.1全局过滤器作用

全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。区别在于GatewayFilter通过配置定义，处理逻辑是固定的；而GlobalFilter的逻辑需要自己写代码实现。

定义方式是实现GlobalFilter接口。

```java
public interface GlobalFilter {
    /**
     *  处理当前请求，有必要的话通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
     *
     * @param exchange 请求上下文，里面可以获取Request、Response等信息
     * @param chain 用来把请求委托给下一个过滤器 
     * @return {@code Mono<Void>} 返回标示当前过滤器业务结束
     */
    Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```

在filter中编写自定义逻辑，可以实现下列功能：

- 登录状态判断
- 权限校验
- 请求限流等

#### 3.5.2自定义全局过滤器

需求：定义全局过滤器，拦截请求，判断请求的参数是否满足下面条件：

- 参数中是否有authorization，
- authorization参数值是否为admin

如果同时满足则放行，否则拦截



实现：

在gateway中定义一个过滤器：

```cobol
package cn.itcast.gateway.filters;
 
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;
 
@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1.获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
        // 2.获取authorization参数
        String auth = params.getFirst("authorization");
        // 3.校验
        if ("admin".equals(auth)) {
            // 放行
            return chain.filter(exchange);
        }
        // 4.拦截
        // 4.1.禁止访问，设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
        // 4.2.结束处理
        return exchange.getResponse().setComplete();
    }
}
```

#### 3.5.3过滤器执行顺序

请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter

请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链（集合）中，排序后依次执行每个过滤器：

![img](https://img-blog.csdnimg.cn/21a210b81ff94d65889493c168f586da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

排序的规则是什么呢？

- 每一个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前**。
- GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
- 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。



详细内容，可以查看源码：

`org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()`方法是先加载defaultFilters，然后再加载某个route的filters，然后合并。



`org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()`方法会加载全局过滤器，与前面的过滤器合并后根据order排序，组织过滤器链

**3.6跨域问题**

#### 3.6.1什么是跨域问题

跨域：域名不一致就是跨域，主要包括：

- 域名不同： [www.taobao.com](https://blog.csdn.net/weixin_45481821/article/details/www.taobao.com) 和 [www.taobao.org](https://blog.csdn.net/weixin_45481821/article/details/www.taobao.org) 和 [www.jd.com](https://blog.csdn.net/weixin_45481821/article/details/www.jd.com) 和 miaosha.jd.com
- 域名相同，端口不同：localhost:8080和localhost8081

跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题



解决方案：CORS，这个以前应该学习过，这里不再赘述了。不知道的小伙伴可以查看[跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html)

#### 3.6.2模拟跨域问题

找到课前资料的页面文件：

![img](https://img-blog.csdnimg.cn/7c7723803a2f4403a08221109648adae.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_18,color_FFFFFF,t_70,g_se,x_16)

放入tomcat或者nginx这样的web服务器中，启动并访问。

可以在浏览器控制台看到下面的错误：

![img](https://img-blog.csdnimg.cn/af7321485c044274a58e2087cb614315.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Zm254S25ZCM5a2m44CC,size_20,color_FFFFFF,t_70,g_se,x_16)

 从localhost:8090访问localhost:10010，端口不同，显然是跨域的请求。  

#### 3.6.3解决跨域问题

在gateway服务的application.yml文件中，添加下面的配置：

```XML
spring:
  cloud:
    gateway:
      # 。。。
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求 
              - "http://localhost:8090"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```