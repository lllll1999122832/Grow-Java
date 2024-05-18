![在这里插入图片描述](https://img-blog.csdnimg.cn/b54d85c79590423c91b6dff66987bb9a.png)
来自面试官发自内审深处的灵魂拷问：“说一下springboot的启动流程”；
一脸懵逼的面试者：“它简化了spring的配置，主要是因为有[自动装配](https://so.csdn.net/so/search?q=自动装配&spm=1001.2101.3001.7020)的功能，并且可以直接启动，因为它内嵌了tomcat容器”；
面试官：“嗯， 没错，这是 它的一些概念，你还没回答我的问题，它是怎么启动的，启懂时都经过了哪些东西？”；
一脸懵逼的面试者：“额~~~不知道额····，我用的很熟练，但是不知道它里面做了哪些事情！”；
面试官：“了解内部原理是为了帮助我们做扩展，同时也是验证了一个人的学习能力，如果你想让自己的职业道路更上一层楼，这些底层的东西你是必须要会的，行吧，你回去等消息吧！”
面试者：↓
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d9b69c654d844bcbe63fe0dfbc30033.png)

## SpringBoot是什么

springboot是依赖于spring的，比起spring，除了拥有spring的全部功能以外，springboot无需繁琐的xml配置，这取决于它自身强大的自动装配功能；并且自身已嵌入Tomcat、[Jetty](https://so.csdn.net/so/search?q=Jetty&spm=1001.2101.3001.7020)等web容器，集成了springmvc，使得springboot可以直接运行，不需要额外的容器，提供了一些大型项目中常见的非功能性特性，如嵌入式服务器、安全、指标，健康检测、外部配置等，

其实spring大家都知道，boot是启动的意思。所以，spring boot其实就是一个启动[spring项目](https://so.csdn.net/so/search?q=spring项目&spm=1001.2101.3001.7020)的一个工具而已，总而言之，springboot 是一个服务于框架的框架；也可以说springboot是一个工具，这个工具简化了spring的配置；



## Spring Boot的核心功能

1、 可独立运行的Spring项目：Spring Boot可以以jar包的形式独立运行。

2、 内嵌的Servlet容器：Spring Boot可以选择内嵌Tomcat、Jetty或者Undertow，无须以war包形式部署项目。

3、 简化的Maven配置：Spring提供推荐的基础 POM 文件来简化Maven 配置。

4、 自动配置Spring：Spring Boot会根据项目依赖来自动配置Spring 框架，极大地减少项目要使用的配置。

5、 提供生产就绪型功能：提供可以直接在生产环境中使用的功能，如性能指标、应用信息和应用健康检查。

6、 无代码生成和xml配置：Spring Boot不生成代码。完全不需要任何xml配置即可实现Spring的所有配置。



## SpringBoot启动过程

springboot的启动经过了一些一系列的处理，我们先看看整体过程的流程图
![在这里插入图片描述](https://img-blog.csdnimg.cn/f48383125dc04a35bfd05a9d01b950cb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
你别说，步骤还挺多，但是不要紧，为了帮助大家理解，接下来将上面的编号一个个地讲解，以通俗易懂的方式告诉大家，里面都做了什么事情，废话不多说，开整

### 1、运行 SpringApplication.run() 方法

可以肯定的是，所有的标准的springboot的应用程序都是从run方法开始的

```java
package com.spring;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
 
@SpringBootApplication
public class App  {
 
    public static void main(String[] args) {
        // 启动springboot
        ConfigurableApplicationContext run = SpringApplication.run(App.class, args);
    }
 
}
123456789101112131415
```

进入run方法后，会 new 一个SpringApplication 对象，创建这个对象的构造函数做了一些准备工作，编号第2~5步就是构造函数里面所做的事情

```java
	/**
	 * Static helper that can be used to run a {@link SpringApplication} from the
	 * specified sources using default settings and user supplied arguments.
	 * @param primarySources the primary sources to load
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return the running {@link ApplicationContext}
	 */
	public static ConfigurableApplicationContext run(Class<?>[] primarySources,
			String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
1234567891011
```

另外在说明一下，springboot启动有三种方式，其他的启动方式可参照我的另一个帖子： [SpringBoot的三种启动方式](https://blog.csdn.net/qq_27184497/article/details/117534334?spm=1001.2014.3001.5501)

### 2、确定应用程序类型

在SpringApplication的构造方法内，首先会通过 WebApplicationType.deduceFromClasspath()； 方法判断当前应用程序的容器，默认使用的是Servlet 容器，除了servlet之外，还有NONE 和 REACTIVE （响应式编程）；
![在这里插入图片描述](https://img-blog.csdnimg.cn/0000a60bbd4045bdbf803e2c37f914de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3、加载所有的初始化器

这里加载的初始化器是springboot自带初始化器，从从 META-INF/spring.factories 配置文件中加载的，那么这个文件在哪呢？自带有2个，分别在源码的jar包的 spring-boot-autoconfigure 项目 和 spring-boot 项目里面各有一个
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f5c03e48f874a67bdcb0ca4c23625b5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
spring.factories文件里面，看到开头是 org.springframework.context.ApplicationContextInitializer 接口就是初始化器了 ，
![在这里插入图片描述](https://img-blog.csdnimg.cn/44152f6d1d0249078487b62f0888f86f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
当然，我们也可以自己实现一个自定义的初始化器：实现 ApplicationContextInitializer接口既可

MyApplicationContextInitializer.java

```java
package com.spring.application;
 
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
/**
 * 自定义的初始化器
 */
public class MyApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        System.out.println("我是初始化的 MyApplicationContextInitializer...");
    }
}
12345678910111213
```

在resources目录下添加 META-INF/spring.factories 配置文件，内容如下，将自定义的初始化器注册进去；

```bash
org.springframework.context.ApplicationContextInitializer=\
com.spring.application.MyApplicationContextInitializer
12
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a605b6f8ed354da299e665af8e082051.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_19,color_FFFFFF,t_70,g_se,x_16)
启动springboot后，就可以看到控制台打印的内容了，在这里我们可以很直观的看到它的执行顺序，是在打印banner的后面执行的；
![在这里插入图片描述](https://img-blog.csdnimg.cn/e761a2185b144f49a48907c25efdc2b7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 4、加载所有的监听器

加载监听器也是从 META-INF/spring.factories 配置文件中加载的，与初始化不同的是，监听器加载的是实现了 ApplicationListener 接口的类
![在这里插入图片描述](https://img-blog.csdnimg.cn/338e3f23f6844695a0dbef316489a792.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
自定义监听器也跟初始化器一样，依葫芦画瓢就可以了，这里不在举例；

### 5、设置程序运行的主类

deduceMainApplicationClass(); 这个方法仅仅是找到main方法所在的类，为后面的扫包作准备，deduce是推断的意思，所以准确地说，这个方法作用是推断出主方法所在的类；
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed36534332354847a2617a6cce2ea902.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 6、开启计时器

程序运行到这里，就已经进入了run方法的主体了，第一步调用的run方法是静态方法，那个时候还没实例化SpringApplication对象，现在调用的run方法是非静态的，是需要实例化后才可以调用的，进来后首先会开启计时器，这个计时器有什么作用呢？顾名思义就使用来计时的嘛，计算springboot启动花了多长时间；关键代码如下：

```java
// 实例化计时器
StopWatch stopWatch = new StopWatch(); 
// 开始计时
stopWatch.start();
1234
```

run方法代码段截图
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5ccc29e064b4b3c8067883894a79ef3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 7、将java.awt.headless设置为true

这里将java.awt.headless设置为true，表示运行在服务器端，在没有显示器器和鼠标键盘的模式下照样可以工作，模拟输入输出设备功能。

做了这样的操作后,SpringBoot想干什么呢?其实是想设置该应用程序,即使没有检测到显示器,也允许其启动.对于服务器来说,是不需要显示器的,所以要这样设置.

方法主体如下：

```java
	private void configureHeadlessProperty() {
		System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, System.getProperty(
				SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
	}
1234
```

通过方法可以看到，setProperty()方法里面又有个getProperty()；这不多此一举吗？其实getProperty()方法里面有2个参数， 第一个key值，第二个是默认值，意思是通过key值查找属性值，如果属性值为空，则返回默认值 true；保证了一定有值的情况；

### 8、获取并启用监听器

这一步 通过监听器来实现初始化的的基本操作，这一步做了2件事情

1. 创建所有 Spring 运行监听器并发布应用启动事件
2. 启用监听器
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/1fd7683ae8a148d2ab1ffec80fabf4c7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 9、设置应用程序参数

将执行run方法时传入的参数封装成一个对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b56165551664a2ab08b25b85fd14c9f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
仅仅是将参数封装成对象，没啥好说的，对象的构造函数如下

```java
	public DefaultApplicationArguments(String[] args) {
		Assert.notNull(args, "Args must not be null");
		this.source = new Source(args);
		this.args = args;
	}
12345
```

那么问题来了，这个参数是从哪来的呢？其实就是main方法里面执行静态run方法传入的参数，
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c2f3a39ad2a479aa503f4537126d03e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 10、准备环境变量

准备环境变量，包含系统属性和用户配置的属性，执行的代码块在 prepareEnvironment 方法内![在这里插入图片描述](https://img-blog.csdnimg.cn/e3b024d70928495898e01c9e21f6120e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
打了断点之后可以看到，它将maven和系统的环境变量都加载进来了
![在这里插入图片描述](https://img-blog.csdnimg.cn/31fd799a10864f4aa3bbb52f6a94c508.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 11、忽略bean信息

这个方法configureIgnoreBeanInfo() 这个方法是将 spring.beaninfo.ignore 的默认值值设为true，意思是跳过beanInfo的搜索，其设置默认值的原理和第7步一样；

```java
    private void configureIgnoreBeanInfo(ConfigurableEnvironment environment) {
		if (System.getProperty(
				CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME) == null) {
			Boolean ignore = environment.getProperty("spring.beaninfo.ignore",
					Boolean.class, Boolean.TRUE);
			System.setProperty(CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME,
					ignore.toString());
		}
	}
123456789
```

当然也可以在配置文件中添加以下配置来设为false

```bash
spring.beaninfo.ignore=false
1
```

目前还不知道这个配置的具体作用，待作者查明后在补上

### 12、打印 banner 信息

显而易见，这个流程就是用来打印控制台那个很大的spring的banner的，就是下面这个东东
![在这里插入图片描述](https://img-blog.csdnimg.cn/11c0be5df65c46d9a7b68f244a8b6ff4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
那他在哪里打印的呢？他在 SpringBootBanner.java 里面打印的，这个类实现了Banner 接口，

而且banner信息是直接在代码里面写死的；
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f502c2a13944df2bb9dc8568c7bf1f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
有些公司喜欢自定义banner信息，如果想要改成自己喜欢的图标该怎么办呢，其实很简单,只需要在resources目录下添加一个 banner.txt 的文件即可，txt文件内容如下

```java
                 _           _
                (_)         | |
 _   _  _____  ___ _ __   __| | ___  _ __   __ _
| | | |/ _ \ \/ / | '_ \ / _` |/ _ \| '_ \ / _` |
| |_| |  __/>  <| | | | | (_| | (_) | | | | (_| |
 \__, |\___/_/\_\_|_| |_|\__,_|\___/|_| |_|\__, |
  __/ |                                     __/ |
 |___/                                     |___/
:: yexindong::
123456789
```

一定要添加到resources目录下，别加错了
![在这里插入图片描述](https://img-blog.csdnimg.cn/31e1481aa5d3489a83420b40fb116d7a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
只需要加一个文件即可，其他什么都不用做，然后直接启动springboot，就可以看到效果了
![在这里插入图片描述](https://img-blog.csdnimg.cn/87378bcc85944dc189d51acfc1bc3b93.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 13、创建应用程序的上下文

实例化应用程序的上下文， 调用 createApplicationContext() 方法，这里就是用反射创建对象，没什么好说的；
![在这里插入图片描述](https://img-blog.csdnimg.cn/cfdfa55d6b0a4f96a3de191088cc9d23.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 14、实例化异常报告器

异常报告器是用来捕捉全局异常使用的，当springboot应用程序在发生异常时，异常报告器会将其捕捉并做相应处理，在spring.factories 文件里配置了默认的异常报告器，

![在这里插入图片描述](https://img-blog.csdnimg.cn/955ab3a57918432e990a0c80967e38d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
需要注意的是，这个异常报告器只会捕获启动过程抛出的异常，如果是在启动完成后，在用户请求时报错，异常报告器不会捕获请求中出现的异常，
![在这里插入图片描述](https://img-blog.csdnimg.cn/eaba60b98fd9495c9d2d961b55d2f6a6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
了解原理了，接下来我们自己配置一个异常报告器来玩玩；

MyExceptionReporter.java 继承 SpringBootExceptionReporter 接口

```java
package com.spring.application;
 
import org.springframework.boot.SpringBootExceptionReporter;
import org.springframework.context.ConfigurableApplicationContext;
 
public class MyExceptionReporter implements SpringBootExceptionReporter {
 
 
    private ConfigurableApplicationContext context;
    // 必须要有一个有参的构造函数，否则启动会报错
    MyExceptionReporter(ConfigurableApplicationContext context) {
        this.context = context;
    }
 
    @Override
    public boolean reportException(Throwable failure) {
        System.out.println("进入异常报告器");
        failure.printStackTrace();
        // 返回false会打印详细springboot错误信息，返回true则只打印异常信息 
        return false;
    }
}
12345678910111213141516171819202122
```

在 spring.factories 文件中注册异常报告器

```bash
# Error Reporters 异常报告器
org.springframework.boot.SpringBootExceptionReporter=\
com.spring.application.MyExceptionReporter
123
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/6dbacc6030ac4cfb90cfa7e369d19d16.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
接着我们在application.yml 中 把端口号设置为一个很大的值，这样肯定会报错，

```bash
server:
  port: 80828888
12
```

启动后，控制台打印如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2692c1d9aa44d1d83118324259f877e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 15、准备上下文环境

这里准备的上下文环境是为了下一步刷新做准备的，里面还做了一些额外的事情；
![在这里插入图片描述](https://img-blog.csdnimg.cn/90b4e5a408f04288a60760d9eb19e6b7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 15.1、实例化单例的beanName生成器

在 postProcessApplicationContext(context); 方法里面。使用单例模式创建 了BeanNameGenerator 对象，其实就是beanName生成器，用来生成bean对象的名称

#### 15.2、执行初始化方法

初始化方法有哪些呢？还记得第3步里面加载的初始化器嘛？其实是执行第3步加载出来的所有初始化器，实现了ApplicationContextInitializer 接口的类

#### 15.3、将启动参数注册到容器中

这里将启动参数以单例的模式注册到容器中，是为了以后方便拿来使用，参数的beanName 为 ：springApplicationArguments

### 16、刷新上下文

刷新上下文已经是spring的范畴了，自动装配和启动 tomcat就是在这个方法里面完成的，还有其他的spring自带的机制在这里就不一一细说了，
![在这里插入图片描述](https://img-blog.csdnimg.cn/101c7adbfc514ae4b91dc4e0454f4cfe.png)

### 17、刷新上下文后置处理

afterRefresh 方法是启动后的一些处理，留给用户扩展使用，目前这个方法里面是空的，

```java
/**
	 * Called after the context has been refreshed.
	 * @param context the application context
	 * @param args the application arguments
	 */
	protected void afterRefresh(ConfigurableApplicationContext context,
			ApplicationArguments args) {
	}
12345678
```

### 18、结束计时器

到这一步，springboot其实就已经完成了，计时器会打印启动springboot的时长
![在这里插入图片描述](https://img-blog.csdnimg.cn/f56f7bdfad0d4b379eebb1c576f8c0db.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)
在控制台看到启动还是挺快的，不到2秒就启动完成了；
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a55f69c6b2c49999c678cc1de54bbaf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 19、发布上下文准备就绪事件

告诉应用程序，我已经准备好了，可以开始工作了
![在这里插入图片描述](https://img-blog.csdnimg.cn/d823d184b4c947b4b1745324f1798b27.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 20、执行自定义的run方法

这是一个扩展功能，callRunners(context, applicationArguments) 可以在启动完成后执行自定义的run方法；有2中方式可以实现：

1. 实现 ApplicationRunner 接口
2. 实现 CommandLineRunner 接口

接下来我们验证一把，为了方便代码可读性，我把这2种方式都放在同一个类里面

```java
package com.spring.init;
 
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
 
/**
 * 自定义run方法的2种方式
 */
@Component
public class MyRunner implements ApplicationRunner, CommandLineRunner {
 
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(" 我是自定义的run方法1，实现 ApplicationRunner 接口既可运行"        );
    }
 
    @Override
    public void run(String... args) throws Exception {
        System.out.println(" 我是自定义的run方法2，实现 CommandLineRunner 接口既可运行"        );
    }
}
1234567891011121314151617181920212223
```

启动springboot后就可以看到控制台打印的信息了
![在这里插入图片描述](https://img-blog.csdnimg.cn/01fc9e7bc4884943b9d07025f579e868.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARmx55Li2WA==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 完

其实了解springboot启动原理对开发人员还是有好处的，至少你知道哪些东西是可以扩展的，以及怎么扩展，它的内部原理是怎么做的，我相信了解这些思路之后，让你自己写一个springboot出来也是可以的； 但是这里只是列出了启动过程，并不涉及到全部，源码是很负杂，记得一个大牛说过：“我们看源码的时候，只能通过联想或猜测作者是怎么想的，并且小心验证，就像我们小时候学古诗一样，也只能去猜测古人的想法，拿道德经来说，每个人读完后都有不同的看法，这就需要见仁见智了”；