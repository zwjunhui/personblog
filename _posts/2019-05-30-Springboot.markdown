---
layout: post
title: "Spring Boot2.0"
data: 2019-05-30 15:45:15 +3500
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
