# 为什么需要Spring？

## 传统JavaWeb开发存在的问题与解决方案

### 问题一：层与层之间紧密耦合在一起，接口与具体实现之间紧密耦合在了一起（归根究底还是耦合程度太高）

解决方案：程序代码中不要手动new对象，使用第三方根据要求为程序提供需要的Bean对象。**此即IoC。**

### 问题二：通用的事务功能耦合在业务代码中，例如：通用的日志功能耦合在业务代码中

解决思路：程序代码中不要手动new对象，第三方根据要求为程序需要的Bean对象的代理对象，代理对象内部动态结合业务和通用功能。**此即AOP。**



# IoC

## 何为IoC？

IoC：Inversion of Control，译为“控制反转”。强调的是将原来在程序中**创建Bean的权力反转给第三方。**

## 第三方是谁？

**BeanFactory**，通过读取配置文件中Bean的基本信息，BeanFactory采用**工厂设计模式**根据配置文件来生产Bean实例。

## 有了BeanFactory，为什么还需要依赖注入(DI)？

在使用了BeanFactory以后，我们确实可以通过BeanFactory来获取Bean，但是当A Bean依赖于B Bean时，我们依旧需要在程序中显式地将B Bean注入至A Bean中，**但此时A B两个Bean同时都存在于BeanFactory中**，我们可以**在BeanFactory中设置将B Bean注入至A bean中**，也就不需要再在代码中显示声明了，此为依赖注入。

## IoC与DI之间有什么关系？

- IoC强调将Bean的创建权交给第三方
- DI强调某个Bean的完整创建依赖于其他Bean或普通参数的注入

## BeanFactory和ApplicationContext的关系：

- BeanFactory是Spring早期接口，ApplicationContext是后期更高级的接口
- ApplicationContext在BeanFactory的基础上对功能进行了扩展，例如：监听功能、国际化功能等
- BeanFactory的API更加偏向底层，ApplicationContext的API大多数是对这些底层API的封装
- ApplicationContext继承自BeanFactory，同时ApplicationContext中维护着对BeanFactory的引用
- **Bean的初始化时机不同**，原始BeanFactory是在首次调用getBean时才进行Bean的创建，而ApplicationContext则是配置文件加载，容器一创建就将Bean都实例化并初始化好。

## 基于XML的Spring应用

| XML配置方式                               | 功能描述                                                     |
| ----------------------------------------- | ------------------------------------------------------------ |
| <bean id="" class="">                     | Bean的id和全限定名配置                                       |
| <bean name="">                            | 通过name设置Bean的别名，通过别名也能直接获取Bean实例         |
| <bean scope="">                           | Bean的作用范围，BeanFactory作为容器时取值singleton和prototype |
| <bean lazy-init="">                       | Bean的实例化时机，是否延迟加载。BeanFactory作为容器时无效    |
| <bean init-method="">                     | Bean实例化后自动执行的初始化方法，method指定方法名           |
| <bean destory-method="">                  | Bean实例销毁前的方法，method指定方法名                       |
| <bean autowire="byType">                  | 设置自动注入模式，常用的有按照类型byType，按照名字byName     |
| <bean factory-bean="" factory-method=""/> | 指定哪个工厂Bean的哪个方法完成Bean的创建                     |

### Bean Scope

- singleton：单例，默认值，Spring容器创建的时候，就会进行Bean的实例化，并存储到容器内部的单例池中，每次getBean时都是从单例池中获取相同的Bean实例。
- prototype：原型，Spring容器初始化时不会创建Bean实例，当调用getBean时才会实例化Bean，每次getBean都会创建一个新的Bean实例。

### lazy-init

- 当lazy-init设置为true的时候将会**延迟加载**，Spring容器创建时，不会立即创建Bean实例，等待用到此实例时再创建并存储到单例池中，后续直接在单例池中获取该Bean，**本质上依旧是单例的**。

### init-method

- 执行指定的初始化方法完成一些初始化的操作

### destory-method

- 执行指定的销毁方法完成一些销毁操作

### Bean的实例化配置

Spring Bean的实例化方式主要有如下两种：

- **构造方法实例化**：底层通过构造方法对Bean进行实例化

  Spring中配置的bean几乎都是无参构造，不再赘述，仅讨论有参构造实例化Bean：

  ```java
  //有参构造方法
  public UserDaoImpl(String name) {
    
  }
  ```

  有参构造在实例化Bean时，需要参数的注入，通过`<constructor-arg>`标签，嵌入在`<bean>`标签内部提供构造参数，如下：

  ```xml
  <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
    <constructor-arg name="name" value="haohao"/>
  </bean>
  ```

  

- **工厂方法实例化**：底层通过调用自定义的工厂方法对Bean进行实例化

  **静态工厂方法**实例化Bean，其实就是定义一个工厂类，提供一个静态方法用于生产Bean实例，再将该工厂类及其静态方法配置给Spring即可。**此方法不需要实例化工厂类本身**。

  ```xml
  <bean id="exampleBean" class="com.example.Factory" factory-method="createInstance"/>
  ```
  
  **非静态工厂方法**实例化Bean，需要先创建工厂类的实例（即工厂Bean），然后通过工厂实力调用非静态方法创建Bean。

  ```xml
  <bean id="factory" class="com.example.Factory"/>
  <bean id="exampleBean" factory-bean="factory" factory-method="createInstance"/>
  ```
  

#### 为什么需要静态工厂方法和非静态工厂方法两种Bean实例化模式？

- 静态工厂方法：

  - 工厂类无需维护状态，所有逻辑通过静态方法实现。
  - 典型场景：工具类工厂（如`Calendar.getInstance`）

  ```java
  public class StaticFactory {
      // 静态方法直接返回Bean实例
      public static User createUser() {
          return new User("Admin", 30);
      }
  }
  ```

  ```xml
  // XML配置
  <bean id="user" class="com.example.StaticFactory" factory-method="createUser"/>
  ```

- 非静态工厂方法：

  - 工厂类需要维护状态或者依赖其他Bean
  - 适合需要根据动态条件（如配置参数）生成不同Bean的情况

  ```java
  public class InstanceFactory {
      private String role;
  
      public void setRole(String role) {
          this.role = role;
      }
  
      // 非静态方法依赖工厂状态
      public User createUser() {
          return new User(role, 30);
      }
  }
  ```

  ```xml
  // XML配置
  <bean id="instanceFactory" class="com.example.InstanceFactory">
      <property name="role" value="Guest"/>
  </bean>
  
  <bean id="user" factory-bean="instanceFactory" factory-method="createUser"/>
  ```

##### 生命周期管理

- 静态工厂方法
  - 工厂类本身不会被Spring容器管理（不参与依赖注入）。
  - 静态方法的调用与Spring Bean生命周期无关。
- 非静态工厂方法：
  - 工厂Bean由Spring容器管理，可参与依赖注入和生命周期回调（如`@PostConstruct`）
  - 工厂实例的初始化优先于目标Bean的创建

### Bean的依赖注入方式

- 通过Bean的set方法注入
  ``` xml
  <property name="userDao" ref="userDao"/>
  <property name="userDao" value="haohao"/>
  ```

- 通过构造Bean的方法进行注入

  ```xml
  <constructor-arg name="userDao" ref="userDao"/>
  <constructor-arg name="userDao" value="haohao"/>
  ```

#### 三种依赖注入的数据类型

- 普通数据类型：String / int / boolean等，通过`value`属性指定
- 引用数据类型：UserDaoImpl / DataSource等，通过`ref`属性指定
- 集合数据类型：List / Map / Properties等



## 基于注解的Spring应用

### @Component

可以通过`@Component`注解的value属性指定当前Bean实例的beanName，也可以省略不写，**不写的情况下为当前类名首字母小写**。

| xml配置                  | 注解           | 描述                                                         |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| <bean scope="">          | @Scope         | 在类上或使用了@Bean标注的方法上，标注Bean的作用范围，取值为signlethon或prototype |
| <bean lazy-init="">      | @Lazy          | 在类上或使用了@Bean标注的方法上，标注Bean是否延迟加载，取值为true和false |
| <bean init-method="">    | @PostConstruct | 在方法上使用，标注Bean的实例化后执行的方法                   |
| <bean destroy-method=""> | @PreConstruct  | 在方法上使用，标注Bean的销毁前执行方法                       |

由于JavaEE开发是分层的，为了每层Bean标识的注解语义更加明确，@Component又衍生出如下三个注解：

| @Component衍生注解 | 描述                |
| ------------------ | ------------------- |
| @Repository        | 在Dao层类上使用     |
| @Service           | 在Service层类上使用 |
| @Controller        | 在Web层类上使用     |

Bean依赖注入的注解，主要是使用注解的方式替代XML的`<property>`标签完成属性的注入操作。Spring主要提供如下注解，用于在Bean内部进行属性注入的：

| 属性注入注解 | 描述                                                 |
| ------------ | ---------------------------------------------------- |
| @Value       | 使用在字段或方法上，用于注入普通数据                 |
| @Autowired   | 使用在字段或方法上，用于根据类型(byType)注入引用数据 |
| @Qualifier   | 使用在字段或方法上，结合`@Autowired`，根据名称注入   |
| @Resource    | 使用在字段或方法上，根据类型或者名称进行注入         |

### byType和byName有什么区别？

`byName`：按名称自动装配

- 匹配规则：Spring容器会查找与属性名相同的Bean ID（或名称），完成注入。
- 特点：
  - 依赖属性的名称必须与目标Bean的ID完全一致
  - 不关心类型是否匹配，仅通过名称查找，如果类型不匹配，则抛出异常

示例：

```java
@Component("userService") // Bean 的 ID 是 "userService"
public class UserServiceImpl implements UserService {}

public class UserController {
    @Autowired
    private UserService userService; // 属性名为 "userService"，与 Bean ID 匹配
}
```

`byType`：按类型自动装配

- 匹配规则：Spring容器会查找与属性类型兼容的Bean完成注入。
- 特点：
  - 依赖属性的类型必须与容器中某个Bean的类型兼容（可以是实现类或者其父类/接口）
  - 如果容器中存在多个相同类型的Bean，会抛出`NoUniqueBeanDefinitionException`，此时需要配合`@Qualifier`指定具体的Bean名称

示例：

```java
@Component // 默认 Bean 名称是 "userServiceImpl"
public class UserServiceImpl implements UserService {}

public class UserController {
    @Autowired // 按类型匹配（UserService 接口）
    private UserService userService; 
}
```

### @Resource和@Autowired的区别

1. **来源不同**：

   - **@Autowired**：由 Spring 框架提供，属于 `org.springframework.beans.factory.annotation` 包。
   - **@Resource**：由 Java 标准（JSR-250）定义，属于 `javax.annotation` 包。

2. **默认注入方式**：

   - **@Autowired**：
     - **按类型注入（byType）**。
     - 若存在多个同类型的 Bean，需配合 `@Qualifier` 指定名称。
   - **@Resource**：
     - **按名称注入（byName）**。
     - 若未指定 `name`，则优先按字段名/方法名匹配 Bean 名称，失败后回退到按类型匹配。

3. **指定 Bean 名称的方式**：

   - **@Autowired**：

     - 需结合 `@Qualifier("beanName")` 显式指定名称。

     ```java
     @Autowired
     @Qualifier("userServiceImplA")
     private UserService userService;
     ```

   - **@Resource**：

     - 直接通过 `name` 属性指定 Bean 名称。

     ```java
     @Resource(name = "userServiceImplA")
     private UserService userService;
     ```

4. **适用范围**：

   - **@Autowired**：
     - 支持构造器、方法、字段、参数注入。
     - 常用于 Spring 生态项目。
   - **@Resource**：
     - 通常用于字段和方法注入，不支持构造器注入。
     - 作为 Java 标准注解，兼容性更好（如 Jakarta EE 环境）。

5. **依赖处理**：

   - **@Autowired**：
     - 默认必须注入（`required=true`），若找不到匹配的 Bean 会报错。
     - 可设置为非必需：`@Autowired(required = false)`。
   - **@Resource**：
     - 默认必须注入，若找不到 Bean 会抛出异常，无法设置非必需。

6. **多 Bean 冲突处理**：

   - **@Autowired**：

     - 同一类型存在多个 Bean 时，必须通过 `@Qualifier` 解决歧义。

     ```java
     @Autowired
     @Qualifier("mysqlDataSource")
     private DataSource dataSource;
     ```

   - **@Resource**：

     - 优先按名称匹配，无需额外注解即可避免歧义（若字段名与 Bean 名称一致）。

     ```java
     @Resource // 字段名 dataSource 需与 Bean 名称一致
     private DataSource dataSource;
     ```

7. **依赖包要求**：

   - **@Autowired**：需引入 Spring 相关依赖（如 `spring-boot-starter`）。
   - **@Resource**：需引入 `javax.annotation-api` 依赖（Java 11 后需手动添加）。

**使用场景对比**

| **场景**               | **推荐注解** | **说明**                                                     |
| :--------------------- | :----------- | :----------------------------------------------------------- |
| 需要与 Spring 解耦     | @Resource    | 使用 Java 标准注解，便于迁移到其他框架。                     |
| 需要构造器注入         | @Autowired   | `@Resource` 不支持构造器注入。                               |
| 存在多个同类型 Bean    | @Resource    | 通过名称直接指定更简洁；或结合 `@Qualifier` 使用 `@Autowired`。 |
| 需要非必需依赖         | @Autowired   | 支持 `required=false`，允许依赖不存在。                      |
| 字段名与 Bean 名称一致 | @Resource    | 无需显式指定名称，代码更简洁。                               |

**示例代码**

1. **使用 @Autowired**：

   ```java
   @Service
   public class UserService {
       @Autowired
       @Qualifier("jdbcUserRepository")
       private UserRepository userRepository;
   }
   ```

2. **使用 @Resource**：

   ```java
   @Service
   public class UserService {
       @Resource(name = "jdbcUserRepository")
       private UserRepository userRepository;
   }
   ```

**总结**

- **优先使用 @Autowired**：在纯 Spring 项目中，结合 `@Qualifier` 处理复杂依赖。
- **选择 @Resource**：需兼容 Java 标准或简化名称匹配时。

### @Bean注解的作用

`@Bean` 注解是 Spring 框架中用于显式声明和配置 Bean 的核心注解，通常用在 **配置类的方法** 上。它的核心作用是告诉 Spring：**该方法返回的对象需要被注册为 Spring 容器中的一个 Bean**，并由容器管理其生命周期和依赖注入。

核心作用：

- 显式定义第三方库或复杂对象的Bean
  当需要将无法修改源码的类（比如第三方库中的类）或**需要复杂初始化逻辑的对象**注册为Bean时，`@Bean`是唯一的选择

  ```java
  @Configuration
  public class AppConfig {
      @Bean
      public DataSource dataSource() {
          HikariDataSource dataSource = new HikariDataSource();
          dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
          dataSource.setUsername("root");
          dataSource.setPassword("123456");
          return dataSource;
      }
  }
  ```

- 自定义Bean的初始化逻辑
  `@Bean`方法中可以编写任意代码，**灵活控制对象的创建和初始化**。

  ```java
  @Bean
  public MyService myService() {
      if (isProduction()) {
          return new ProductionService();
      } else {
          return new DevelopmentService();
      }
  }
  ```

### Bean配置类的注解开发

`@Configuration`注解标识的类为配置类，替代原有xml配置文件，该注解第一个作用是标识该类是一个配置类，第二个作用是具备`@Component`作用。

```java
@Configuration
public class ApplicationContextConfig{}
```

`@ComponentScan`组件扫描配置，替代原有xml文件中的`<context:component-scan base-package=""/>`。

```java
@Configuration
@ComponentScan({"com.example.service","com.example.dao"})
public class ApplicationContextConfig{}
```

base-package的配置方式：

- 指定一个或多个包名：扫描指定包及其子包下使用注解的类
- 不配置包名：扫描当前`@ComponentScan`注解配置类所在包及其子包下的类

`@PropertySource`注解用于加载外部properties资源配置，代替原有的`<context:property-placeholder location=""/>`配置

```java
@Configuration
@ComponentScan
@PropertySource("classpath:jdbc.properties","classpath:xxx.properties")
public class ApplicationContextConfig{}
```

`@Import`用于加载其他配置类，替代原有xml中的`<import resource="classpath:beans.xml"/>`配置

```java
@Configuration
@ComponentScan
@PropertySource("classpath:jdbc.properties","classpath:xxx.properties")
@Import(OtherConfig.class)
public class ApplicationContextConfig{}
```

`@Primary`注解用于标注相同类型的Bean优先被使用权，@Primary是Spring3.0引入的，与@Component和@Bean一起使用，标注该Bean的优先级更高，则在通过类型获取Bean或通过@Autowired根据类型进行注入时，会选用优先级更高的。@Primary注解可以用于解决多个实现类之间的选择问题。当一个接口有多个实现类时，可以使用@Primary注解来指定其中一个实现类作为首选的候选对象。例如：

```java
@Primary  
@Component  
public class PrimaryServiceImpl implements Service {  
    // 实现细节  
}
```

在上面的示例中，@Primary注解指示Spring容器在初始化时优先选择PrimaryServiceImpl实例作为Service接口的实现对象。

`@Profile`注解的作用等同于xml配置时的profile属性，是进行环境切换用的：`<beans profile="test">`。注解@Profile标注在类或方法上，标注当前产生的Bean从属于哪个环境，只有激活了当前环境，被标注的Bean才能被注册到Spring容器里，不指定环境的Bean，任何环境下都能注册到Spring容器里。



# AOP

AOP是横向的对不同事物的抽象，属性与属性、方法与方法、对象与对象都可以组成一个切面，而用这种思维去设计编程的方式叫做面向切面编程。AOP的核心思想是将某些通用功能（如日志、事务管理、异常处理等）从业务逻辑代码中剥离出来，形成独立的切面模块。这样可以在不修改原始代码的情况下为程序添加额外的功能，从而实现代码的解耦和复用。

**AOP思想的实现方案：**动态代理技术，在运行期间，对目标对象的方法进行增强，代理对象同名方法内可以执行原有逻辑的同时嵌入执行其他增强逻辑或其他对象的方法。

## AOP的相关概念

| 概念     | 单词      | 解释                                       |
| -------- | --------- | ------------------------------------------ |
| 目标对象 | Target    | 被增强的方法所在的对象                     |
| 代理对象 | Proxy     | 被增强后的对象，也就是客户端实际调用的对象 |
| 连接点   | Joinpoint | 目标对象中可以被增强的方法                 |
| 切入点   | Pointcut  | 目标对象中已经被增强的方法                 |
| 增强     | Advice    | 增强部分的代码逻辑                         |
| 切面     | Aspect    | 增强和切入点的组合                         |
| 织入     | Weaving   | 将通知和切入点动态组合的过程               |



# Spring事务
