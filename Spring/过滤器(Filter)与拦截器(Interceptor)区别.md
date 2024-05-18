## ；1 过滤器（Filter）

Servlet 中的过滤器 Filter 实现了 javax.servlet.Filter 接口的服务器端程序，主要用途是设置字符集（CharacterEncodingFilter）、控制权限、控制转向、用户是否已经登陆、有没有权限访问该页面等。

其工作原理是，只要你在 web.xml 文件配置好要拦截的客户端请求，它都会帮你拦截到请求。此时，其实你可以对请求或响应（request、response）统一设置编码；它随 web 应用启动而启动，只初始化一次，以后就可以拦截相关请求。只有当你的 web 应用停止或重新部署的时候才销毁。

- Filter 可以认为是 Servlet 的一种”加强版”。它主要用于对用户请求进行预处理，也可以对
  HttpServletResponse 进行后处理，是个典型的处理链；
- Filter 可以对用户请求生成响应，和 Servlet 相同；
- 处理流程：用户请求 -> Filter 预处理 -> Servlet 处理请求生成响应 -> Filter 对响应进行后处理。

### 1.1 Filter 的用处

- 在 HttpServletRequest 到达 Servlet 之前，拦截客户的 HttpServletRequest；
- 根据需要检查 HttpServletRequest，也可以修改 HttpServletRequest 头和数据；
- 在 HttpServletResponse 到达客户端之前，拦截 HttpServletResponse；
- 根据需要检查 HttpServletResponse，也可以修改 HttpServletResponse头和数据。

Filter 有如下几个种类。

### 1.2 Filter 的种类

- 用户授权的 Filter：Filter 负责检查用户请求，根据请求过滤用户非法请求；
- 日志 Filter：详细记录某些特殊的用户请求；
- 负责解码的 Filter：包括对非标准编码的请求解码；
- 能改变 XML 内容的 XSLT Filter 等。

Filter 可以负责拦截多个请求或响应，一个请求或响应也可以被多个 Filter 拦截。

### 1.3 创建 Filter 的步骤

1. 创建 Filter 处理类，并实现 javax.servlet.Filter 接口；
2. web.xml 文件中配置 Filter（或者使用 @WebFilter 注解）。

javax.servlet.Filter 接口中定义了三个方法：

- void init(FilterConfig config) 用于完成 Filter 的初始化；
- void destory() 用于 Filter 销毁前，完成某些资源的回收；
- void [doFilter](https://so.csdn.net/so/search?q=doFilter&spm=1001.2101.3001.7020)(ServletRequest request,ServletResponse response,FilterChain chain) 实现过滤功能。

doFilter 方法就是对每个请求及响应增加的额外处理。该方法可以实现对用户请求进行预处理（ServletRequest request），也可实现对服务器响应进行后处理(ServletResponse response）。它们的分界线为，是否调用了 chain.doFilter()。执行该方法之前，即对用户请求进行预处理；执行该方法之后，即对服务器响应进行后处理。

## 2 [拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)（Interceptor）

拦截器是在面向切面编程中应用的，就是 service 或一个方法前/后调用一个方法。是基础 Java 的反射机制。拦截不是在 web.xml，而是在 AOP（Aspect-Oriented Programming）中用于某个方法或字段被访问之前，进行拦截。然后在之前或之后加入某些操作，甚至在抛出异常的时候做业务逻辑的操作。拦击器是 AOP 的一种实现策略。

### 2.1 拦截器的实现方式

SpringMVC 中的 Interceptor 拦截请求是通过 HandlerInterceptor 来实现的。在 SpringMVC 中定义 Interceptor 主要有两种方式：

- 实现 Spring 的 HandlerInterceptor 接口或者继承了实现 HandlerInterceptor 接口的类（比如HandlerInterceptorAdapter）；
- 实现 Spring 的 WebRequestInterceptor 接口，或者继承了实现WebRequestInterceptor接口的类。

Interceptor 中的方法：

**preHandle (HttpServletRequest request, HttpServletResponse response, Object handle) 方法**

顾名思义，该方法将在请求处理之前进行调用。

SpringMVC 中的 Interceptor 是链式调用。在一个应用中或者说是在一个请求中可以同时存在多个 Interceptor，每个 Interceptor 的调用会依据它的声明顺序依次执行，而且最先执行的都是 Interceptor 中的 preHandle 方法。

所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值 Boolean 类型的，当它返回为 false 时，表示请求结束，后续的 Interceptor 和Controller 都不会再执行；当返回值为 true 时就会继续调用下一个 Interceptor 的 preHandle 方法。如果已经是最后一个 Interceptor 的时候就会是调用当前请求的Controller 方法。

**postHandle (HttpServletRequest request, HttpServletResponse response, Object handle, ModelAndView modelAndView) 方法**

由 preHandle 方法的解释我们知道，这个方法包括后面要说到的 afterCompletion 方法都只能是在当前所属的 Interceptor 的 preHandle 方法的返回值为 true 时才能被调用。

postHandle 方法，顾名思义就是在当前请求进行处理之后，也就是 Controller 方法调用之后执行，但是它会在 DispatcherServlet 进行视图返回渲染之前被调用。所以，我们可以在这个方法中对 Controller 处理之后的 ModelAndView 对象进行操作。

postHandle 方法被调用的方向跟 preHandle 是相反的。也就是说，先声明的 Interceptor 的 postHandle 方法反而会后执行。这和 Struts2 里面的 Interceptor 的执行过程有点类似。Struts2 里面的 Interceptor 的执行过程也是链式的，只是在 Struts2 里面需要手动调用 ActionInvocation 的 invoke 方法来触发对下一个 Interceptor 或者是 Action 的调用，然后每一个 Interceptor 中在 invoke 方法调用之前的内容都是按照声明顺序执行的，而 invoke 方法之后的内容就是反向的。

**afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex) 方法**

该方法也是需要当前对应的 Interceptor 的p reHandle 方法的返回值为 true 时才会执行。顾名思义，该方法将在整个请求结束之后，也就是在 DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。

```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.log4j.Logger;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
public class ExecuteTimeInterceptor extends HandlerInterceptorAdapter{
    private static final Logger logger = Logger.getLogger(ExecuteTimeInterceptor.class);
    //before the actual handler will be executed
    public boolean preHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler)
        throws Exception {
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        return true;
    }

    //after the handler is executed
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
        throws Exception {
        long startTime = (Long)request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        //统计耗时
        long executeTime = endTime - startTime;
        //modified the exisitng modelAndView
        modelAndView.addObject("executeTime",executeTime);

        //log it
        if(logger.isDebugEnabled()){
           logger.debug("[" + handler + "] executeTime : " + executeTime + "ms");
        }
    }
}

```

#### 2.1.1 非 SpringBoot 项目实现方式

使用 mvc:interceptors 标签来声明需要加入到 SpringMVC 拦截器链中的拦截器。

```xml
<mvc:interceptors>  
<!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
<bean class="com.company.app.web.interceptor.AllInterceptor"/>  
    <mvc:interceptor>  
         <mvc:mapping path="/**"/>  
         <mvc:exclude-mapping path="/parent/**"/>  
         <bean class="com.company.authorization.interceptor.SecurityInterceptor" />  
    </mvc:interceptor>  
    <mvc:interceptor>  
         <mvc:mapping path="/parent/**"/>  
         <bean class="com.company.authorization.interceptor.SecuritySystemInterceptor" />  
    </mvc:interceptor>  
</mvc:interceptors>

```

可以利用 mvc:interceptors 标签声明一系列的拦截器，然后它们就可以形成一个拦截器链。拦截器的执行顺序是按声明的先后顺序执行的，先声明的拦截器中的 preHandle 方法会先执行，然而它的 postHandle 方法和 afterCompletion 方法却会后执行。

在 mvc:interceptors 标签下声明 interceptor 主要有两种方式：

- 直接定义一个 Interceptor 实现类的 bean 对象。使用这种方式声明的 Interceptor
  拦截器将会对所有的请求进行拦截；
- 使用 mvc:interceptor 标签进行声明。使用这种方式进行声明的 Interceptor 可以通过 mvc:mapping
  子标签来定义需要进行拦截的请求路径。

经过上述两步之后，定义的拦截器就会发生作用对特定的请求进行拦截了。

#### 2.1.2 Spring Boot 项目

配置拦截器

```java
@Configuration
//@Configurationpublic
public class WebAppConfigurer implements WebMvcConfigurer {

    /**
     * 配置拦截器
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 多个拦截器组成一个拦截器链
        registry.addInterceptor(new ExecuteTimeInterceptor()).addPathPatterns("/**");

        //API限流拦截
        registry.addInterceptor(accessLimitAjaxInterceptor()).addPathPatterns("/**").excludePathPatterns("/static/**","/login.html");
        registry.addInterceptor(accessInterceptor()).addPathPatterns("/**").excludePathPatterns("/static/**","/login.html");
    }
}

```

### 2.2 拦截器（interceptor）使用

1. 请求到达 DispatcherServlet；
2. DispatcherServlet 发送至 Interceptor，执行 preHandler；
3. 请求到达 Controller；
4. 请求结束后，执行 postHandler。

## 3 过滤器（Filter）与 拦截器（Interceptor）的区别

Spring 的 Interceptor（拦截器）与 Servlet 的 Filter 有相似之处，比如二者都是 AOP 编程思想的体现，都能实现权限检查、日志记录等。

不同之处总结如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/afaa30ce5d99420fb76629d5e64a15f4.png)

### 3.1 执行顺序

**用户请求 -> 过滤前 -> 拦截前 -> Action处理 -> 拦截后 -> 过滤后 -> 响应**
![在这里插入图片描述](https://img-blog.csdnimg.cn/963dc79e416946c1b7e14e5a80b5d1e2.png)

## 4 过滤器（Filter）与 拦截器（Interceptor）常见用途

- Authentication Filters
- Logging and Auditing Filtersx
- Image conversion Filters
- Data compression Filters
- Encryption Filters
- Tokenizing Filters
- Filters that trigger resource access events
- XSL/T filters
- Mime-type chain Filter

Request Filters 可以实现：

- 执行安全检查
- 格式化请求头和主体
- 审查或者记录日志
- 根据请求内容授权或者限制用户访问
- 根据请求频率限制用户访问

Response Filters 可以实现:

- 压缩响应内容：比如让下载的内容更小
- 追加或者修改响应
- 创建或者整体修改响应
- 根据地方不同修改响应内容

## 5 总结

### 5.1 过滤器

所谓过滤器顾名思义是用来过滤的。在 Java Web中，传入的 request、response 提前过滤掉一些信息，或者提前设置一些参数；然后再传入 Servlet 或者 Struts 的 action 进行业务逻辑。比如，过滤掉非法 URL（不是 login.do 的地址请求，如果用户没有登陆都过滤掉），或者在传入 Servlet 或者 Struts 的 action 前统一设置字符集，或者去除掉一些非法字符（聊天室经常用到的，一些骂人的话）。

Filter 流程是线性的， URL 传来之后，检查之后，可保持原来的流程继续向下执行，被下一个 filter、servlet 接收等。

### 5.2 Java 的拦截器

主要是用在插件上，扩展件上比如 Hibernate Spring Struts2等 有点类似面向切片的技术，在用之前先要在配置文件即 XML 文件里进行对应的声明。

**拦截器（Interceptor）是基于 Java 的反射机制，而过滤器（Filter）是基于函数回调**。从灵活性上说拦截器功能更强大些，Filter 能做的事情，拦截器都能做，而且可以在请求前、请求后执行，比较灵活。Filter 主要是针对 URL 地址做一个编码的事情、过滤掉没用的参数、安全校验（比如登录不登录之类），太细的话，还是建议用拦截器。