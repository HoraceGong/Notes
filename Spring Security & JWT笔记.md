---
title: Spring Security & JWT笔记
date: 2024-09-29 13:53:01
tags: [Java, Java Web, Spring, Spring Security, Micro-service, note]
---



# 0.简介

Spring Security是Spring家族中的一个安全管理框架，相比于Shiro，它提供了更丰富的功能。一般来说中大型项目都是使用Spring Security来做安全框架。

Web应用中最重要的安全管理内容：**认证**和**授权**。

**认证：验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户**

**授权：经过认证后判断当前用户是否有权限进行某个操作**

<!-- more -->



# 1.快速入门

第一步：创建项目并引入Springboot

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```



第二步，创建Springboot启动类

```java
@SpringBootApplication
public class SpringSecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityApplication.class, args);
    }

}
```



第三步，新建controller，并启动程序测试是否成功运行

```java
@RestController
@RequestMapping("/hello")
public class index {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
    
}
```

<img src="https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-03%2014.44.50.png" alt="运行成功" style="zoom:50%;" />

运行成功！



第四步，引入Spring Security

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**注意：此时不要写版本号<version>，否则pom文件会报错**

此时再次访问接口，自动转跳到Spring Security的自带的登录页面

<img src="https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-03%2015.07.11.png" alt="登录页面" style="zoom:50%;" />

默认用户名为user，默认密码在控制台输出。点击sign in后，再次进入原本的页面。

此时Spring Security已经可以运行！



# 2.认证

## 2.1 登录校验流程

<img src="https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-03%2015.45.26.png" alt="校验流程" style="zoom:50%;" />

## 2.2 Spring Security原理

### 2.2.1 Spring Security完整流程

Spring Security本身就是一个过滤器链。

<img src="https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-03%2020.02.56.png" alt="过滤器链" style="zoom:50%;" />

- **UsernamePasswordAuthenticationFilter：**处理在登录页面填写了用户名和密码后的登录请求。入门案例的认证工作主要由它负责。
- **ExceptionTranslationFilter：**处理过滤器链充抛出的任何AccessDeniedException和AuthenticationException。
- **FilterSecurityInterceptor：**负责权限校验。



### 2.2.2 认证流程详解

<img src="https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-04%2015.30.04.png" alt="认证流程" style="zoom:50%;" />

概念速查：

- Authentication接口：它的实现类表示当前访问系统的用户，封装了用户的相关信息。
- AuthenticationManagement接口：定义了认证Authentication的方法。
- UserDetailsSerivce接口：加载用户特定数据的核心接口，里面定义了一个根据用户名查询用户信息的方法。
- UserDetails接口：提供核心用户信息。通过UserDetailsService根据用户名获取处理的用户信息，封装成UserDetails对象返回，然后将这些信息封装到Authentication对象中。

但是此时出现了一些问题：

- 实际项目中是前后端分离项目，需要查询数据库，并不是从内存中查询，所以InMemoryUserDetailsService需要自己重写，改为去数据库查询用户信息。
- 对于前后端分离项目，验证通过后需要响应回去一个token，无法在Spring Security的filter种返回token，需要自己写一个filter。



![前后端分离项目验证流程](https://picturebedforhorace.oss-cn-beijing.aliyuncs.com/%E6%88%AA%E5%B1%8F2024-10-04%2015.47.42.png)



## 2.3 解决问题

### 2.3.1 思路分析

登录：

- 自定义登录接口，调用ProviderManager的方法进行认证，如果认证通过，则生成jwt，并将数据信息存入redis中。
- 自定义UserDetailsService，在这个实现类中去查询用户信息数据库。

校验：

- 定义jwt认证过滤器，在过滤器中获取token、解析token并获取其中的userID，从redis中获取用户信息，并存入SecurityContextHolder。
