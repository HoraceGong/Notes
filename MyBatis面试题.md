# 什么是MyBatis

MyBatis 是一个持久层框架，简化了 SQL 操作。它**不完全是 ORM（对象关系映射）**，但可以将 SQL 语句与 Java 对象绑定，方便开发者直接编写和管理 SQL，同时支持动态 SQL 和灵活的查询映射。

## MyBatis架构

来源：https://pdai.tech/md/framework/orm-mybatis/mybatis-y-arch.html

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

**优点**：

1. **灵活性和控制力**
   - 开发者直接编写SQL，可精细优化复杂查询（如多表关联、动态条件、分页等），适合对性能要求高的场景。
   - 支持动态SQL（`<if>`, `<foreach>` 等标签），灵活应对多条件查询。
   - 适用于遗留数据库或需要与复杂存储过程交互的场景。
2. **轻量级与低侵入性**
   - 无需强制遵循对象-关系映射（ORM）规范，适合对数据库表结构有特殊设计的项目。
   - 与Spring等框架集成简单，配置简洁。
3. **透明化SQL管理**
   - SQL与Java代码解耦，集中管理在XML或注解中，便于维护和DBA协作。
4. **性能优化可控**
   - 避免全自动ORM框架可能生成的冗余SQL（如N+1查询问题），直接控制执行计划。

**缺点**：

1. **手动编写SQL**
   - 开发效率较低，需为每个操作编写SQL，增加代码量。
   - SQL维护成本高，尤其在表结构频繁变更时。
2. **半自动化特性**
   - 需要手动处理对象-关系映射（ResultMap配置），复杂映射场景下配置较繁琐。
   - 不支持自动化缓存机制（需手动集成如Redis等）。
3. **学习曲线**
   - 需熟悉XML配置或注解语法，以及动态SQL标签的使用。

## MyBatis 与其他ORM框架对比

| **框架**            | **类型**    | **核心优势**                                             | **劣势**                                         | **适用场景**                                        |
| :------------------ | :---------- | :------------------------------------------------------- | :----------------------------------------------- | :-------------------------------------------------- |
| **MyBatis**         | 半自动化ORM | SQL完全可控，灵活性高，适合复杂查询和性能优化。          | 需手动编写SQL，开发效率较低。                    | 对SQL有精细化控制需求的业务（如金融、大数据分析）。 |
| **Hibernate**       | 全自动化ORM | 开发效率高，自动化CRUD和缓存，支持HQL面向对象查询。      | 复杂查询优化困难，生成的SQL可能不够高效。        | 快速开发、对象关系映射简单的业务系统。              |
| **Spring Data JPA** | JPA规范实现 | 极简Repository接口，支持派生查询，与Spring生态无缝集成。 | 复杂查询需依赖@Query或Criteria API，灵活性受限。 | 基于Spring的快速开发项目，简单CRUD操作为主。        |

# `#{}`和`${}`有什么区别

1. 处理机制

- `#{}`：采用**预编译(PreparedStatement)**机制，参数会被替换为`?`占位符，通过JDBC的预编译功能对参数进行安全处理，**防止SQL注入**。
- `${}`：采用**字符串替换**机制，直接将参数值拼接到SQL语句中，生成完整的SQL。

2. 安全性

- `#{}`：天然防止SQL注入，适用于传递**参数值**。
- `${}`：存在SQL注入风险，适用于动态拼接**SQL关键字、表名、列名**等非参数值场景。

3. 使用场景

- **`#{}`** 适用场景：

  - WHERE条件中的值：`WHERE id = #{id}`

  - INSERT/UPDATE的字段值：`VALUES(#{name}, #{age})`

  - 模糊查询（需结合函数处理）：

    ```sql
    WHERE name LIKE CONCAT('%', #{keyword}, '%')
    ```

- **`${}`** 适用场景：

  - 动态表名/列名：`SELECT * FROM ${tableName}`
  - 动态排序：`ORDER BY ${column} ${order}`
  - 动态SQL片段（如动态的`GROUP BY`或`HAVING`）

最佳实践

- 优先使用`#{}`，仅在必须动态拼接SQL关键字或结构时使用`${}`，并严格校验参数值。

## 为什么使用`#{}`能防止SQL注入

在MyBatis中使用 `#{}` 能防止SQL注入，主要归功于其底层依赖的 **预编译（PreparedStatement）机制** 和 **参数化查询** 的安全特性。

### 预编译机制

`#{}` 在底层会通过JDBC的 `PreparedStatement` 实现参数替换。具体过程如下：

1. **SQL预解析**：
   数据库会先将SQL语句的骨架（如 `SELECT * FROM user WHERE id = ?`）进行语法分析、编译和优化，生成执行计划。此时，参数部分以占位符 `?` 表示，**SQL结构已经固定**。
2. **参数传递**：
   后续传入的参数值（如 `id = 5` 或 `id = '1 OR 1=1'`）会被严格视为**数据**，而非SQL代码的一部分。
3. **类型安全处理**：
   数据库会根据参数的类型自动进行转义处理（如字符串添加引号、特殊字符转义），确保参数值不会破坏原有SQL结构。

### 参数化查询的安全特性

- **输入值始终为“数据”**：
  即使用户传入恶意参数（如 `id = '1 OR 1=1'`），预编译机制会将其视为一个**完整的字符串值**，而非可执行的SQL片段。
  - 生成的SQL实际为：`SELECT * FROM user WHERE id = '1 OR 1=1'`
  - 由于 `id` 字段的值被正确转义为字符串，无法改变SQL逻辑，查询会直接查找 `id = '1 OR 1=1'` 的记录（通常不存在），而不会返回所有数据。
- **特殊字符自动转义**：
  若参数包含引号、分号等特殊字符（如 `name = "John'; DROP TABLE user; --"`），预编译机制会将其转义为普通字符，避免闭合原有SQL语句或执行额外操作。

## 什么时候必须使用`${}`

在MyBatis中，动态表名、列名等场景必须使用 `${}`，而不能使用 `#{}`，这是因为 **SQL语法结构本身的限制**，而非MyBatis的设计缺陷。但这种场景确实存在SQL注入风险，需要开发者通过**严格的校验机制**来确保安全。

### 为什么动态表名、列名必须用`${}`？

1. **`#{}` 的预编译机制会导致语法错误**

- **`#{}` 会为参数值添加引号**（如字符串类型），而表名、列名是SQL的标识符（Identifier），不能加引号。
  **错误示例**：

  ```sql
  SELECT * FROM #{tableName} WHERE id = 1
  ```

  - 若 `tableName = "user"`，实际生成的SQL为：

    ```sql
    SELECT * FROM 'user' WHERE id = 1  -- 语法错误！
    ```

  - 表名被错误地包裹在引号中，导致SQL执行失败。

- **`${}` 直接替换为字符串**，不会添加引号：

  ```sql
  SELECT * FROM ${tableName} WHERE id = 1
  ```

  - 生成的SQL为：

    ```sql
    SELECT * FROM user WHERE id = 1  -- 语法正确
    ```

2. **SQL语法限制：标识符不能参数化**

- 在标准SQL中，表名、列名、排序关键字（如 `ORDER BY`）等属于**SQL语法结构**，而非参数值。
  这些部分必须在SQL语句的**预编译前确定**，无法通过占位符 `?` 动态绑定。

  - 例如，以下写法在JDBC中不合法：

    ```java
    PreparedStatement ps = connection.prepareStatement("SELECT * FROM ?");
    ps.setString(1, "user");  // 报错：表名不能通过占位符绑定！
    ```

### 使用`${}`，依旧存在SQL注入风险

如果直接使用用户输入的参数作为表名或列名，且未做校验，会导致SQL注入。
**示例攻击**：

```sql
SELECT * FROM ${tableName}
```

- 若 `tableName = "user; DROP TABLE orders; --"`，生成的SQL为：

  ```sql
  SELECT * FROM user; DROP TABLE orders; --  -- 执行后删除orders表！
  ```

虽然必须使用 `${}`，但可以通过以下方法**严格规避风险**：

#### 1. **白名单校验**

- **限制参数值只能从预定义的合法集合中选取**（如固定的表名、列名）。
  **示例代码**：

  ```java
  // 定义合法的表名集合
  private static final Set<String> ALLOWED_TABLES = Set.of("user", "product", "order");
  
  public void queryTable(String tableName) {
      if (!ALLOWED_TABLES.contains(tableName)) {
          throw new IllegalArgumentException("Invalid table name!");
      }
      // 执行SQL：SELECT * FROM ${tableName}
  }
  ```

#### 2. **避免直接使用用户输入**

- 动态表名/列名应来自系统内部逻辑（如根据业务规则生成），而非用户直接输入。
  例如：根据用户类型选择不同的表（`user_vip`、`user_normal`），但表名由代码生成，而非前端传递。

#### 3. **转义标识符（部分数据库支持）**

- 某些数据库（如MySQL）支持对表名/列名进行反引号转义，但需谨慎使用：

  ```sql
  SELECT * FROM `${tableName}`
  ```

  - 若 `tableName = "user; DROP TABLE orders; --"`，生成的SQL为：

    ```sql
    SELECT * FROM `user; DROP TABLE orders; --`  -- 表名无效，执行失败
    ```

  - 但这仅能防止语法错误，仍需结合白名单校验！

# MyBatis缓存

## 一级缓存

为解决同一SqlSession中可能存在的**反复执行相同查询语句**导致的反复查询数据库相同内容造成的资源浪费问题，MyBatis在SqlSession对象中建立一个简单的缓存，将每次查询到的结果缓存起来，如果下场有相同的查询，可以直接通过缓存返回查询内容。

### 作用范围

- **默认开启**，生命周期与SqlSession绑定
- 当执行`commit()`、`close()`或增删改操作时，缓存会自动清空

### 工作原理

- **缓存key**：由SQL语句、参数、分页参数等生成唯一标识
- **查询流程**：
  1. 执行查询时，先检查一级缓存是否存在匹配结果
  2. 存在则直接返回缓存数据，否则查询数据库并缓存结果

### 示例

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

// 第一次查询（访问数据库）
User user1 = mapper.selectUserById(1); 

// 第二次查询（命中一级缓存）
User user2 = mapper.selectUserById(1); 

sqlSession.commit(); // 清空缓存
```

### 注意事项

- **脏数据风险**：若手动修改数据库绕过MyBatis，可能导致缓存与数据库不一致。
- **作用域限制**：不同`SqlSession`的缓存互相隔离。

## 二级缓存

二级缓存是Application级别的缓存，但二级缓存并非在Application中只用一个Cache缓存，而是每一个Mapper可以拥有一个Cache缓存。

### 作用范围

- **手动开启**，作用域为`Mapper`接口或XML命名空间，跨`SqlSession`共享。
- 生命周期与应用同步，直到显式清楚或缓存策略过期。

### 配置步骤

1. **全局启用**（`mybatis-config.xml`）：

   ```xml
   <settings>
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. **Mapper文件启用**（添加 `<cache/>` 标签）：

   ```xml
   <mapper namespace="com.example.UserMapper">
       <cache eviction="LRU" flushInterval="60000" size="1024"/>
   </mapper>
   ```

### 缓存策略参数

| 参数            | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| `eviction`      | 缓存淘汰策略（LRU、FIFO、SOFT、WEAK）                        |
| `flushInterval` | 缓存刷新间隔（毫秒）                                         |
| `size`          | 缓存最大对象数                                               |
| `readOnly`      | 是否只读（`true`：返回缓存对象的引用；`false`：返回深拷贝副本） |

### 示例

```java
// SqlSession1 查询并提交（数据进入二级缓存）
SqlSession session1 = sqlSessionFactory.openSession();
User user1 = session1.getMapper(UserMapper.class).selectUserById(1);
session1.commit();
session1.close();

// SqlSession2 直接命中二级缓存
SqlSession session2 = sqlSessionFactory.openSession();
User user2 = session2.getMapper(UserMapper.class).selectUserById(1);
session2.close();
```

### 注意事项

- **序列化要求**：缓存对象需实现 `Serializable` 接口。
- **事务提交后生效**：只有 `SqlSession` 提交后，数据才会写入二级缓存。
- **避免关联查询循环引用**：复杂对象可能导致序列化失败或缓存膨胀。

# MyBatis是如何进行分页的？分页的原理是什么？

## 逻辑分页

逻辑分页指从数据库**查询全部数据**到应用层（如Java内存），再通过代码截取当前页的数据。分页逻辑**完全由应用程序处理**，与数据库无关。

### 实现原理

1. 执行一条未分页的 SQL（如 `SELECT * FROM table`），获取所有数据。
2. 将数据加载到内存中。
3. 通过代码截取指定范围的数据（如 `List.subList()`）。

### 示例代码

```java
List<User> allUsers = userMapper.selectAll(); // 查询全部数据
int pageNum = 2; // 第2页
int pageSize = 10; // 每页10条

// 截取当前页数据（假设索引从0开始）
List<User> pageData = allUsers.subList(
    (pageNum-1) * pageSize, 
    Math.min(pageNum * pageSize, allUsers.size())
);
```

### 优点

- **实现简单**：无需处理复杂的分页 SQL。
- **跨数据库兼容**：不依赖数据库的分页语法。

### 缺点

- **性能极差**：数据量大时，查询和传输全部数据会占用大量内存和网络带宽。
- **不适合生产环境**：仅适用于小数据量测试或管理后台。

## 物理分页

物理分页是指在**数据库层面**直接按分页条件查询，仅返回当前页的数据。分页逻辑**由数据库处理**，应用层仅获取所需数据。

### 实现原理

1. 在 SQL 中添加分页关键字（如 `LIMIT`、`OFFSET`、`ROWNUM`）。
2. 数据库仅执行分页查询，返回指定范围内的数据。

### 示例代码

```sql
-- MySQL
SELECT * FROM user LIMIT 10 OFFSET 20; -- 第3页（每页10条）
```

### 优点

- **高性能**：仅传输和处理当前页数据，节省内存和网络资源。
- **适合生产环境**：支持大数据量场景。

### 缺点

- **SQL复杂度高**：需要编写数据库特定的分页语法。
- **深度分页性能问题**：偏移量`OFFSET`过大时，数据库仍需扫描大量数据（可以通过游标分页优化）。

### 物理分页的优化技巧

1. 避免深度分页

- **问题**：`LIMIT 100000, 10` 会导致数据库扫描前 100010 行。

- **优化方案**：使用 **游标分页**（基于排序字段和上一页最后一条数据的值）：

  ```sql
  -- 传统分页（性能差）
  SELECT * FROM user ORDER BY id LIMIT 100000, 10;
  
  -- 游标分页（性能高）
  SELECT * FROM user 
  WHERE id > #{lastId}  -- 上一页最后一条数据的id
  ORDER BY id 
  LIMIT 10;
  ```

2. 索引优化

- 确保 `ORDER BY` 和 `WHERE` 条件中的字段有索引。
- 避免全表扫描，提升分页查询速度。

3. 分页插件

- 使用 MyBatis 插件（如 **PageHelper**）自动生成物理分页 SQL，简化开发：

  ```java
  PageHelper.startPage(2, 10); // 第2页，每页10条
  List<User> users = userMapper.selectAll();
  ```

## MyBatis分页的常见方式

### 手动编写SQL分页

直接在 SQL 中拼接分页语法（如 MySQL 的 `LIMIT`）：

```xml
<select id="selectUsersByPage" resultType="User">
    SELECT * FROM user
    LIMIT #{offset}, #{pageSize}
</select>
```

- **优点**：简单直接，性能高。
- **缺点**：需要处理不同数据库的分页语法差异，代码冗余。

### 使用分页插件

通过拦截器动态修改 SQL，自动添加分页语法，并封装分页结果。
**示例代码**：

```java
// 启动分页（页码=1，每页10条）
PageHelper.startPage(1, 10);
List<User> users = userMapper.selectAllUsers();
PageInfo<User> pageInfo = new PageInfo<>(users);
```

- **优点**：无需手动处理分页逻辑，支持多数据库。
- **缺点**：依赖插件，需理解其拦截机制。

### 逻辑分页（最不推荐）

查询全部数据后，通过代码截取分页结果：

```java
List<User> allUsers = userMapper.selectAllUsers();
List<User> pageData = allUsers.subList((pageNum-1)*pageSize, pageNum*pageSize);
```

- **优点**：实现简单。
- **缺点**：数据量大时性能极差，不推荐使用。

## PageHelper的核心原理

1. **分页参数绑定**

- 调用 `PageHelper.startPage(pageNum, pageSize)`，将分页参数（页码、每页条数）存储到 **`ThreadLocal`** 中，确保线程安全。

- **代码示例**：

  ```java
  // 分页参数存储到当前线程
  Page<?> page = PageHelper.startPage(1, 10);
  ```

2. **拦截 SQL 执行**

- 插件通过实现 MyBatis 的 `Interceptor` 接口，拦截 `Executor` 的 `query` 方法。
- **拦截逻辑**：
  1. 检测当前线程是否存在分页参数。
  2. 存在分页参数时，修改原始 SQL，添加分页语法。

3. **生成分页 SQL**

- 根据数据库类型（如 MySQL、Oracle）生成对应的分页语句：

  - **MySQL**：

    ```sql
    -- 原始 SQL
    SELECT * FROM user;
    -- 分页 SQL
    SELECT * FROM user LIMIT 0, 10;
    ```

  - **Oracle**：

    ```sql
    -- 原始 SQL
    SELECT * FROM user;
    -- 分页 SQL
    SELECT * FROM (
        SELECT tmp.*, ROWNUM row_id FROM (
            SELECT * FROM user
        ) tmp WHERE ROWNUM <= 10
    ) WHERE row_id > 0;
    ```

4. **查询总记录数**

- 自动执行一条 **`COUNT` 查询**，获取满足条件的总记录数：

  ```sql
  SELECT COUNT(*) FROM user;
  ```

- **优化**：若 SQL 包含 `GROUP BY`，则改写为统计临时表记录数：

  ```sql
  SELECT COUNT(*) FROM (原始SQL) tmp;
  ```

5. **封装分页结果**

- 将分页数据（当前页数据、总记录数、总页数等）封装到 `PageInfo` 对象：

  ```java
  PageInfo<User> pageInfo = new PageInfo<>(users);
  System.out.println("总记录数：" + pageInfo.getTotal());
  System.out.println("总页数：" + pageInfo.getPages());
  ```

# 简述MyBatis的插件运行原理，以及如何编写一个插件

来源：https://pdai.tech/md/framework/orm-mybatis/mybatis-y-plugin.html



# MyBatis动态SQL是做什么的？都有哪些动态SQL？简述一下原理？



# 为什么说MyBatis是半自动ORM映射工具？与全自动的区别在哪？



# MyBatis是否支持延迟加载？如果支持，它的实现原理是？