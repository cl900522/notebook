Spring启动过程
=============

## Spring Boot启动图

![Spring Boot 启动流程](/images/2019/04/spring-boot-start.png)

## 流程说明

* org.springframework.boot.SpringApplication#run 自定义的应用类通过静态方法启动Spring Boot
  
* org.springframework.boot.SpringApplication#SpringApplication构造SpringApplication，通过查找META-INF/spring.factories文件，查找**org.springframework.context.ApplicationContextInitializer**和**org.springframework.context.ApplicationListener**声明的类，并初始化实例对象。

* org.springframework.boot.SpringApplication#run 启动计时功能，s通过META-INF/spring.factories查找**org.springframework.boot.SpringApplicationRunListener**和**org.springframework.boot.SpringBootExceptionReporter**声明类，并初始化实例对象

* 初始化args入参，通过SpringApplicationRunListeners触发环境准备事件，org.springframework.boot.SpringApplication#createApplicationContext创建spring的ApplicationContext
  * **org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext**
  * **org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext**
  * **org.springframework.context.annotation.AnnotationConfigApplicationContext**

* org.springframework.boot.SpringApplication#prepareContext 准备Spring的ApplicationContext并通过org.springframework.boot.SpringApplicationRunListener触发事件监听处理

* org.springframework.boot.SpringApplication#refreshContext，调用ApplicationContext的refresh方法

* org.springframework.boot.SpringApplication#afterRefresh 什么也没做

* org.springframework.boot.SpringApplication#callRunners，找到**org.springframework.boot.ApplicationRunner**和**org.springframework.boot.CommandLineRunner**在spring容器中的实例，入参为arg调用他们的run方法

* 启动错误通过org.springframework.boot.SpringBootExceptionReporter的实例进行处理