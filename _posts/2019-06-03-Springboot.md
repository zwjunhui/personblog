---
layout: post
title: "Spring Boot2"
data: 2019-06-03 15:45:15 +3500
categories: jekyll update
---
####  SpringBoot配置详解
`SpringBoot` 虽然干掉了 XML 但未做到 **零配置**，它体现出了一种 **约定优于配置，也称作按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。** 一般情况下默认的配置足够满足日常开发所需，但在特殊的情况下，我们往往需要用到**自定义属性配置、自定义文件配置、多环境配置、外部命令引导**等一系列功能。
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
//该依赖可以不添加，但是在 IDEA 和 STS 中不会有属性提示，没有提示的配置就跟你用记事本写代码一样苦逼，出个问题弄哭你去），该依赖只会在编译时调用，所以不用担心会对生产造成影响
```
##### **自定义配置属性**
配置文件中书写自定义信息 application.properties
```java
my1.age=22
my1.name=tom
```
在文件中映射出配置文件中的信息
```java
@Component
//如果配置文件不为application.properties
//需要加上@PropertySource("classpath:my2.properties")
@ConfigurationProperties(prefix = "my1")
public class entity {

    private int age;
    private String name;
	// 省略 get set
}
```
**Spring4.x 以后，推荐使用构造函数的形式注入属性…**
```java
@RestController
public class PropertiesController {

    private final Entity entity;

    @Autowired
    public PropertiesController(Entity entity) {
        this.entity = entity;
    }
}
```
在真实的应用中，常常会有多个环境（**如：开发，测试，生产等**），不同的环境数据库连接都不一样，这个时候就需要用到`spring.profile.active` 的强大功能了，
它的格式为 `application-{profile}.properties`，这里的 `application` 为前缀不能改，`{profile}` 是我们自己定义的。!


在 `application.properties` 配置文件中写入 `spring.profiles.active=dev`

##### 使用jdbcTemplate访问数据库
```javascript
<!-- Spring JDBC 的依赖包，使用 spring-boot-starter-jdbc 或 spring-boot-starter-data-jpa 将会自动获得HikariCP依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!-- MYSQL包 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!-- 分页插件文档地址：https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.5</version>
</dependency>
<!-- 默认就内嵌了Tomcat 容器，如需要更换容器也极其简单-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--swagger-spring-boot-starter 的依赖-->
<dependency>
    <groupId>com.battcn</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.4.5-RELEASE</version>
</dependency>
```
```javascript
#SpringBoot默认会自动配置DataSource，它将优先采用HikariCP连接池，如果没有该依赖的情况则选取#tomcat-jdbc，如果前两者都不可用最后选取Commons DBCP2。

spring.datasource.url=jdbc:mysql://localhost:3306/chapter4?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&useSSL=false
spring.datasource.password=root
spring.datasource.username=root
#spring.datasource.type
#更多细微的配置可以通过下列前缀进行调整
#spring.datasource.hikari
#spring.datasource.tomcat
#spring.datasource.dbcp2
# JPA配置
spring.jpa.hibernate.ddl-auto=update
# 输出日志
spring.jpa.show-sql=true
# 数据库类型
spring.jpa.database=mysql

#Mysql的配置
# 注意注意
mybatis.mapper-locations=classpath:mapper/*.xml        
#这种方式需要自己在resources目录下创建mapper目录然后存放xml
mybatis.type-aliases-package=com.battcn.entity
# 驼峰命名规范 如：数据库字段是  order_id 那么 实体字段就要写成 orderId
mybatis.configuration.map-underscore-to-camel-case=true

# 如果想看到mybatis日志需要做如下配置
logging.level.com.battcn=DEBUG
########## 通用Mapper ##########
# 主键自增回写方法,默认值MYSQL,详细说明请看文档
mapper.identity=MYSQL
mapper.mappers=tk.mybatis.mapper.common.BaseMapper
# 设置 insert 和 update 中，是否判断字符串类型!=''
mapper.not-empty=true
# 枚举按简单类型处理
mapper.enum-as-simple-type=true  枚举按简单类型处理，如果有枚举字段则需要加上该配置才会做映射
########## 分页插件 ##########
pagehelper.helper-dialect=mysql
pagehelper.params=count=countSql
pagehelper.reasonable=false  分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
pagehelper.support-methods-arguments=true  支持通过 Mapper 接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页。
```