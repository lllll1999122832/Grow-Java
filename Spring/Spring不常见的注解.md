### @EnableAsync

配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口。（类上）

### @Async

在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

### @EnableScheduling

在配置类上使用，开启计划任务的支持。（类上）

### @Scheduled

来申明这是一个任务，包括`cron,fixDelay,fixRate`等类型。（方法上，需先开启计划任务的支持）

###@Scope

@Scope注解可以用来定义@Component标注的类的作用范围以及@Bean所标记的类的作用范围。@Scope所限定的作用范围有：singleton、prototype、request、session、globalSession或者其他的自定义范围。这里以prototype为例子进行讲解。

### @Cacheable

作用：用于缓存方法的返回值。

在Spring框架中，如果一个方法的返回结果是固定的，而且这个方法的执行比较耗时，我们可以使用@Cacheable注解将这个方法的返回结果缓存起来，下次执行这个方法时直接从缓存中获取结果，避免重复执行。

### @RequestHeader

把Request请求header部分的值绑定到方法的参数上

### @CookieValue

把Request header中关于cookie的值绑定到方法的参数上