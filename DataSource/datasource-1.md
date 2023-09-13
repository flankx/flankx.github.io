# 多数据源组件

dynamic-datasource-spring-boot-starter 🔥是一个基于springboot的快速集成多数据源的启动器。

其支持**Jdk 1.7+, SpringBoot 1.5.x 和 2.x.x**。

## 特性

1.  支持**数据源分组**，适用于多种场景 纯粹多库 读写分离 一主多从 混合模式。
2.  支持数据库敏感配置信息**加密**ENC()。
3.  支持每个数据库独立初始化表结构schema和数据库database。
4.  支持**自定义注解**，需继承DS(3.2.0+)。
5.  提供对Druid，Mybatis-Plus，P6sy，Jndi的快速集成。
6.  简化Druid和HikariCp配置，提供**全局参数配置**。配置一次，全局通用。
7.  提供**自定义数据源来源**方案。
8.  提供项目启动后**动态增加移除数据源**方案。
9.  提供Mybatis环境下的**纯读写分离**方案。
10.  提供使用**spel动态参数**解析数据源方案。内置spel，session，header，支持自定义。
11.  支持**多层数据源嵌套切换**。（ServiceA >>> ServiceB >>> ServiceC）。
12.  提供对shiro，sharding-jdbc,quartz等第三方库集成的方案,注意事项和示例。
13.  提供**基于seata的分布式事务方案。**附：不支持原生spring事务。
14.  提供**本地多数据源事务方案。**附：不支持原生spring事务。

## 约定

1.  本框架只做**切换数据源**这件核心的事情。
2.  配置文件所有以下划线`_`分割的数据源**首部**即为组的名称，相同组名称的数据源会放在一个组下。
3.  切换数据源可以是组名，也可以是具体数据源名称。组名则切换时采用负载均衡算法切换。
4.  默认的数据源名称为**master**，你可以通过`spring.datasource.dynamic.primary`修改。
5.  方法上的注解优先于类上注解。

## 使用方法

1.  引入dynamic-datasource-spring-boot-starter。[![](https://img.shields.io/maven-central/v/com.baomidou/dynamic-datasource-spring-boot-starter.svg)](http://mvnrepository.com/artifact/com.baomidou/dynamic-datasource-spring-boot-starter)

~~~xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>

~~~

2.  配置数据源。

~~~yaml
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #设置严格模式,默认false不启动. 启动后在未匹配到指定数据源时候会抛出异常,不启动则使用默认数据源.
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0开始支持SPI可省略此配置
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx) # 内置加密,使用请查看详细文档
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
          schema: db/schema.sql # 配置则生效,自动初始化表结构
          data: db/data.sql # 配置则生效,自动初始化数据
          continue-on-error: true # 默认true,初始化失败是否继续
          separator: ";" # sql默认分号分隔符
          
       #......省略
       #以上会配置一个默认库master，一个组slave下有两个子库slave_1,slave_2

~~~

~~~
# 多主多从                      纯粹多库（记得设置primary）                   混合配置
spring:                               spring:                               spring:
  datasource:                           datasource:                           datasource:
    dynamic:                              dynamic:                              dynamic:
      datasource:                           datasource:                           datasource:
        master_1:                             mysql:                                master:
        master_2:                             oracle:                               slave_1:
        slave_1:                              sqlserver:                            slave_2:
        slave_2:                              postgresql:                           oracle_1:
        slave_3:                              h2:                                   oracle_2:

~~~

3.  使用**@DS**切换数据源。

**@DS**可以注解在方法上或类上，**同时存在就近原则 方法上注解 优先于 类上注解**。

| 注解          | 结果                                     |
| ------------- | ---------------------------------------- |
| 没有@DS       | 默认数据源                               |
| @DS("dsName") | dsName可以为组名也可以为具体某个库的名称 |

~~~java
@Service
@DS("slave")
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public List selectAll() {
    return  jdbcTemplate.queryForList("select * from user");
  }
  
  @Override
  @DS("slave_1")
  public List selectByCondition() {
    return  jdbcTemplate.queryForList("select * from user where age >10");
  }
}
~~~


## 官方文档
文档直达：https://www.kancloud.cn/tracy5546/dynamic-datasource