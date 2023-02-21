# 启动时执行指定方法

## 一、实现方式

### 1. 实现 ServletContextListener 的 contextInitialized 方法

```Java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

@Slf4j
@Component
public class ServletContextListenerDemo implements ServletContextListener {
    /**
     * web应用程序初始化过程正在启动的通知。在初始化web应用程序中的任何过滤器或servlet之前执行
     * @param sce
     */
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        log.info("启动执行 ServletContextListener 的 contextInitialized 方法");
    }
}
```

### 2. 静态代码块

```Java
@Slf4j
@Component
public class TestDemo {
    static {
        log.info("启动执行静态代码块");
    }
    public TestDemo() {
        log.info("构造方法");
    }
    @PostConstruct
    public void initial() {
        log.info("执行 PostConstruct 注解的方法");
    }
}
```

### 3. 注解 @PostConstruct 【同上】

### 4. 实现 ServletContextAware 的 setServletContext 方法

```Java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.context.ServletContextAware;

import javax.servlet.ServletContext;

@Slf4j
@Component
public class ServletContextAwareDemo implements ServletContextAware {
    /**
     * 在填充普通bean属性之后，但在初始化回调（如InitializingBean的afterPropertiesSet或自定义init方法）之前调用。
     * 在ApplicationContextAware的setApplicationContext之后调用。
     * @param servletContext
     */
    @Override
    public void setServletContext(ServletContext servletContext) {
        log.info("启动执行 setServletContext 方法");
    }
}
```

### 5. 注解 @EventListener

```Java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class EventListenerDemo {

    @EventListener
    public void init(ContextRefreshedEvent event) {
        log.info("启动 EventListener 事件交给 spring 管理");
    }
}
```

### 6. 实现 ApplicationRunner 接口的 run 方法

```Java
@Slf4j
@Component
public class ApplicationRunnerDemo implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("启动 ApplicationRunner 的 run 方法");
        Set<String> optionNames = args.getOptionNames();
        for (String op: optionNames
             ) {
            log.info("传入的参数名" + op);
        }

        String[] sourceArgs = args.getSourceArgs();
        for (String arg:sourceArgs
             ) {
            log.info("传入的原始参数。" + arg);
        }
    }
}
```

### 7. 实现 CommandLineRunner 接口的 run 方法

```Java
@Slf4j
@Component
public class CommandLineRunnerDemo implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("执行 CommandLineRunner 的 run 方法， 传入参数 ", args);
    }
}
```

## 二、执行顺序

```log
22:03:25.688  INFO 21132 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
22:03:25.716  INFO 21132 --- [main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
22:03:25.716  INFO 21132 --- [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.19]
22:03:25.808  INFO 21132 --- [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
22:03:25.809  INFO 21132 --- [main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1200 ms
22:03:25.841  INFO 21132 --- [main] c.b.n.i.ServletContextListenerDemo       : 启动执行 ServletContextListener 的 contextInitialized 方法
22:03:25.850  INFO 21132 --- [main] c.b.n.i.ServletContextAwareDemo          : 启动执行 setServletContext 方法
22:03:25.851  INFO 21132 --- [main] com.bookman.notes.initmethod.TestDemo    : 启动执行静态代码块
22:03:25.851  INFO 21132 --- [main] com.bookman.notes.initmethod.TestDemo    : 构造方法
22:03:25.852  INFO 21132 --- [main] com.bookman.notes.initmethod.TestDemo    : 执行 PostConstruct 注解的方法
22:03:25.983  INFO 21132 --- [main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
22:03:26.121  INFO 21132 --- [main] c.b.notes.initmethod.EventListenerDemo   : 启动 EventListener 事件交给 spring 管理
22:03:26.150  INFO 21132 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
22:03:26.155  INFO 21132 --- [main] com.bookman.notes.NotesApplication       : Started NotesApplication in 1.953 seconds (JVM running for 2.378)
22:03:26.156  INFO 21132 --- [main] c.b.n.initmethod.ApplicationRunnerDemo   : 启动 ApplicationRunner 的 run 方法
22:03:26.157  INFO 21132 --- [main] c.b.n.initmethod.ApplicationRunnerDemo   : 传入的参数名foo
22:03:26.157  INFO 21132 --- [main] c.b.n.initmethod.ApplicationRunnerDemo   : 传入的原始参数。--foo=debug
22:03:26.157  INFO 21132 --- [main] c.b.n.initmethod.CommandLineRunnerDemo   : 执行 CommandLineRunner 的 run 方法， 传入参数  
```
