# 什么是MyBatis

MyBatis 是一个持久层框架，简化了 SQL 操作。它**不完全是 ORM（对象关系映射）**，但可以将 SQL 语句与 Java 对象绑定，方便开发者直接编写和管理 SQL，同时支持动态 SQL 和灵活的查询映射。

## MyBatis架构

![img](MyBatis面试题.assets/mybatis-y-arch-1.png)

### **接口层**

接口层负责和数据库交互，有两种方式。

**使用MyBatis提供的API**

这是传统的传递Statement Id 和查询参数给 SqlSession 对象，使用 SqlSession对象完成和数据库的交互；MyBatis 提供了非常方便和简单的API，供用户实现对数据库的增删改查数据操作，以及对数据库连接信息和MyBatis 自身配置信息的维护操作。**这种方式固然很简单和实用，但是它不符合面向对象语言的概念和面向接口编程的编程习惯。**

**使用Mapper接口**

MyBatis 将配置文件中的每一个`<mapper>` 节点抽象为一个 Mapper 接口，而这个接口中声明的方法和跟`<mapper>` 节点中的`<select|update|delete|insert>` 节点项对应，即`<select|update|delete|insert>` 节点的id值为Mapper 接口中的方法名称，parameterType 值表示Mapper 对应方法的入参类型，而resultMap 值则对应了Mapper 接口表示的返回值类型或者返回结果集的元素类型。

根据MyBatis的配置规范配置好后，通过SqlSession.getMapper(XXXMapper.class)方法，MyBatis 会根据相应的接口声明的方法信息，通过**动态代理机制生成一个Mapper实例**，我们使用Mapper接口的某一个方法时，MyBatis会根据这个方法的方法名和参数类型，确定Statement Id，底层还是通过SqlSession.select("statementId",parameterObject);或者SqlSession.update("statementId",parameterObject); 等等来实现对数据库的操作， MyBatis 引用Mapper 接口这种调用方式，纯粹是**为了满足面向接口编程的需要**。（其实还有一个原因是在于，面向接口的编程，使得用户在接口上可以使用注解来配置SQL语句，这样就可以脱离XML配置文件，实现“0配置”）。

### **数据处理层**

- 通过传入参数构建动态SQL
- SQL语句的执行以及封装查询结果集成`List<E>`

**参数映射和动态SQL语句生成**

动态语句生成可以说是MyBatis框架非常优雅的一个设计，MyBatis 通过传入的参数值，使用 Ognl 来动态地构造SQL语句，使得MyBatis 有很强的灵活性和扩展性。

参数映射指的是对于java 数据类型和jdbc数据类型之间的转换：这里有包括两个过程：查询阶段，我们要将java类型的数据，转换成jdbc类型的数据，通过 preparedStatement.setXXX() 来设值；另一个就是对resultset查询结果集的jdbcType 数据转换成java 数据类型。

**SQL语句的执行以及封装查询结果集成List**

动态SQL语句生成之后，MyBatis 将执行SQL语句，并将可能返回的结果集转换成`List<E>` 列表。MyBatis 在对结果集的处理中，支持结果集关系一对多和多对一的转换，并且有两种支持方式，一种为嵌套查询语句的查询，还有一种是嵌套结果集的查询。

### **框架支撑层**

- 事务管理机制
- 连接池管理机制
- 缓存机制
- SQL语句配置方式

### **引导层**

引导层是配置和启动MyBatis配置信息的方式。MyBatis提供两种方式来引导MyBatis：基于XML配置文件的方式和基于Java API的方式。

# MyBatis优缺点



# MyBatis缓存



# MyBatis是如何进行分页的？分页的原理是什么？

## 逻辑分页与物理分页

# 简述MyBatis的插件运行原理，以及如何编写一个插件



# MyBatis动态SQL是做什么的？都有哪些动态SQL？简述一下原理？



# #{}和${}有什么区别



# 为什么说MyBatis是半自动ORM映射工具？与全自动的区别在哪？



# MyBatis是否支持延迟加载？如果支持，它的实现原理是？