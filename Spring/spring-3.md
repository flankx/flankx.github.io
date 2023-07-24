---
title: SpringBoot自动配置实现原理

tags: [SpringBoot]
authors: Flankx
description:  自动配置

---

# SpringBoot自动配置实现原理

`SpringBoot` 定义了一套接口规范，这套规范规定：`SpringBoot` 在启动时会扫描外部引用 jar 包中的 `META-INF/spring.factories` 文件，将文件中配置的类型信息加载到 Spring 容器（此处涉及到 JVM 类加载机制与 `Spring` 的容器知识），并执行类中定义的各种操作。对于外部 jar 来说，只需要按照 `SpringBoot` 定义的标准，就能将自己的功能装置进 `SpringBoot`

+ `@SpringBootApplication` 看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合
+ `@EnableAutoConfiguration`：启用 `SpringBoot` 的自动配置机制
+ `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
+ `@ComponentScan`： 扫描被 `@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除TypeExcludeFilter和AutoConfigurationExcludeFilter。

总结：`SpringBoot` 通过 `@EnableAutoConfiguration` 开启自动装配，通过 `SpringFactoriesLoader` 最终加载 `META-INF/spring.factories` 中的自动配置类实现自动装配，自动配置类其实就是通过`@Conditional` 按需加载的配置类，想要其生效必须引入   `spring-boot-starter-xxx` 包实现起步依赖
