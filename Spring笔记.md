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

