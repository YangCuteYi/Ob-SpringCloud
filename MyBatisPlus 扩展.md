
MyBatisPlus 官方内部提供了许多扩展

## 1 <font color="#ffc000">代码生成 </font>

**MybatisPlus** 官方提供了代码生成器，能根据数据库表结构生成 `PO` 、`Mapper`、`Service` 等相关代码，只不过代码生成器同样要编码使用，也很麻烦

可以使用一款 `MybatisPlus` 的插件，它可以基于<font color="#ffc000">图形化界面</font>完成 `MybatisPlus` 的代码生成

### 1.1 <font color="#ffc000">安装</font>

![[MyBatisPlus 代码生成插件安装.png]]

---
### 1.2 <font color="#ffc000">使用</font>

1. 配置数据库地址

*<font color="#ffc000">Tools -> Config Database</font>*

2. 配置生成信息

*<font color="#ffc000">Tools -> Code Generator</font>*

![[MyBatisPlus 代码生成插件配置信息.png]]

---
---
## 2 Db <font color="#ffc000">静态工具</font>

**MybatisPlus** 提供一个静态工具类：`Db`，能够通过 **静态方法** 直接访问数据库，执行常见的 **CRUD** 操作，而不需要依赖于 `Service` 注入，来避免 [[循环依赖]] 的问题


| **方法**   |  **功能描述**   | **示例**    |
| --- | --- | --- |
| `getById`    | 根据主键 `ID` 查询数据    |`User user = Db.getById(userId, User.class);`     |
|`lambdaQuery`     |使用 `LambdaQueryWrapper` 查询数据     |`List<User> users = Db.lambdaQuery(User.class).eq(User::getStatus, 1).list();`     |
|`lambdaUpdate`    |使用 `LambdaUpdateWrapper` 更新数据    | `Db.lambdaUpdate(User.class).set(User::getBalance, 1000).eq(User::getId, userId).update();`    |
|`removeById`    | 根据 `ID` 删除数据    | `Db.removeById(userId, User.class);`    |
| `save`    | 执行插入操作    | `Db.save(user);`    |
| `saveBatch`    | 批量插入操作    | `Db.saveBatch(users);`    |
| `update`    | 更新数据（根据主键 `ID`）    |`Db.update(user);`     |
| `list`    | 查询所有数据    | `List<User> users = Db.list(User.class);`    |
|...|...|...|

> [!question]
> Q：为什么使用 `Db` 静态方法时，需要传递**实体类的字节码文件**（例如 `User.class` ）
> 
> A：目的是通过<font color="#ffc000"> 反射技术 </font>获取实体类的相关信息（如字段、类型等），并动态构造 SQL 语句

---
---
## 3 <font color="#ffc000">逻辑删除</font>

**<font color="#ffc000">引言</font>**：

*对于一些比较重要的数据，往往会采用逻辑删除的方案，即：*

- *在表中添加一个字段标记数据是否被删除*
- *当删除数据时把标记置为true*  
- *查询时过滤掉标记为true的数据*

*一旦采用了逻辑删除，所有的查询和删除逻辑都要跟着变化，非常繁琐。为了解决这个问题，**MybatisPlus** 添加了对逻辑删除的支持*

### 3.1 <font color="#ffc000">使用</font>

> 1. <font color="#ffc000">全局配置</font>
> 
> ```YAML
> # application.yml
> mybatis-plus:
>   global-config:
>     db-config:
>       logic-delete-field: deleted  # 逻辑删除字段名
>       logic-delete-value: 1        # 删除标记值
>       logic-not-delete-value: 0    # 未删除标记值
> ```
> ---
> 2. <font color="#ffc000">实体类添加逻辑字段与注解 @TableLogic</font>
> - 逻辑字段上添加 `@TableLogic` 注解
> 
> ```java
> @TableLogic
> private Integer deleted; // 逻辑删除字段，值为1表示已删除，0表示未删除
> ```

---

### 3.2 <font color="#ffc000">启用逻辑删除后的SQL变化</font>

**<font color="#ffc000">查询操作</font>**：

在查询数据时，**MybatisPlus** 会自动忽略标记为删除的记录（`deleted = 1`），即只会查询 `deleted = 0` 的数据，避免了手动添加过滤条件

```java
SELECT * FROM user WHERE deleted = 0;
```


**<font color="#ffc000">删除操作</font>**：

删除操作会将 `deleted` 字段的值更新为 1（表示已删除），而不是从数据库中物理删除数据

```java
UPDATE user SET deleted = 1 WHERE id = ?;
```

---

## 4 <font color="#ffc000">枚举处理器</font>

**<font color="#ffc000">引言</font>**：

*在实际开发中，数据库和 Java 代码之间往往需要对字段类型进行相应的转换。例如，数据库中的状态字段通常会用整数表示不同的状态，如 1 表示正常，2 表示异常。然而，Java 代码中的状态字段却可能被定义为枚举或整数类型，这可能导致类型不一致的情况，需要手动进行类型的转换*

*为了避免手动转换和确保代码与数据库类型的自动匹配，**MyBatisPlus** 提供了**枚举处理器**，通过它我们可以将 Java 枚举与数据库字段自动映射，简化了数据类型转换的工作，例如，在定义枚举时，**MyBatisPlus** 可以自动将枚举值转换为数据库中的整数或字符串*

### 4.1 <font color="#ffc000">使用</font>

> 1. <font color="#ffc000">全局配置枚举处理器</font>
> 
> ```YAML
> mybatis-plus:
>   configuration:
>     default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
> ```
> ---
> 2. <font color="#ffc000">枚举类字段上加注解 @EnumValue</font>
> - <font color="#ffc000">@EnumValue </font>注解的作用是指定枚举类中的某个字段作为数据库中实际存储的值，例如当枚举与数据库字段的存储值不完全相同时使用
> 
> ```Java
> 示例：
> 
> @Getter
> public enum UserStatus {
>     NORMAL(1, "正常"),
>     ABNORMAL(2, "异常")
>     ;
> 
> 	@EnumValue
>     private final int value;
>     private final String desc;
> 
>     UserStatus(int value, String desc) {
>         this.value = value;
>         this.desc = desc;
>     }
> }
> ```
---
### 4.2 **<font color="#ffc000">场景</font>**
> 
> 1. 假设实体类中有一个 `status` 字段，数据库中存储的是数字 1 和 2，分别表示 “正常” 和 “异常” 。在 Java 中，可能会使用枚举来表示这个字段的状态
> 
> ```java
> 示例：
> 。。。
> @Getter
> public enum UserStatus {
>     NORMAL(1, "正常"),
>     ABNORMAL(2, "异常")
>     ;
> 
> 	@EnumValue
>     private final int value;
>     private final String desc;
> 
>     UserStatus(int value, String desc) {
>         this.value = value;
>         this.desc = desc;
>     }
> }
> ```
> 
> - <font color="#ffc000">@EnumValue</font> 注解标注了 `value` 字段，表示这个字段的值应该用于与数据库中对应的 `status` 字段进行映射
> - 数据库中的 1 将映射为 `UserStatus.NORMAL` ，2 将映射为 `UserStatus.ABNORMAL`
> ---
> 2. 当我们执行查询时，**MyBatisPlus** 会自动将数据库中存储的数字（如 1 或 2）转换为对应的枚举对象
> 
> ```java
> 示例：
> ...
> List<User> users = Db.lambdaQuery(User.class)
>     .eq(User::getStatus, UserStatus.NORMAL)  // 使用枚举对象进行查询
>     .list();
> ```
> ---
> 3. `UserStatus.NORMAL` 会自动被转换为对应的数据库值 1，因此实际执行的 SQL 会是：
> 
> ```java
> SELECT * FROM user WHERE status = 1;
> ```

---
---
## 5 <font color="#ffc000">JSON处理器</font>

**<font color="#ffc000">引言</font>**：

*在现代开发中，数据库中存储的字段经常会涉及到 **JSON** 数据类型，例如，我们在用户的 `info` 字段中可能会存储一些非结构化的数据（如用户的偏好设置、个性化信息等）。这时我们就需要一种方便的方式来处理这些 JSON 数据*

*MyBatisPlus 提供了内建的 JSON 处理器，它可以帮助我们自动转换数据库中存储的 JSON 字符串为 Java 对象，或者将 Java 对象转换为 JSON 格式存储到数据库中*

### 5.1 <font color="#ffc000">使用</font>

> 1. <font color="#ffc000">创建转换对应的 `Java` 实体</font>
> 
> ```java
> 示例：
> 
> @Data
> public class UserInfo {
>     private Integer age; // 字段对应数据库中的JSON属性
>     private String gender;
> }
> ```
> ---
> 2. <font color="#ffc000">在字段上指定</font> `TypeHandler`
> - 使用注解 <font color="#ffc000">@TableField(typeHandler = JacksonTypeHandler.class)</font>
> 
> ```java
> @TableField(typeHandler = JacksonTypeHandler.class)
> private UserInfo info;  // 需要转换的字段，`UserInfo` 是一个 Java 对象
> ```
> ---
> 3. <font color="#ffc000">声明自动映射</font>
> - 在需要转换的字段所在类上添加 `@TableName(value = "user", autoResultMap = true)`
> - 允许 **MyBatisPlus** 自动将数据库中的 JSON 字段与 Java 对象进行映射
> ```java
> @TableName(value = "user", autoResultMap = true)
> public class User{
> 
>      @TableField(typeHandler = JacksonTypeHandler.class)
>      private UserInfo info;  // 需要转换的字段，`UserInfo` 是一个 Java 对象
> }
> ```

---

### 5.2 <font color="#ffc000">场景</font>
> 1. 假设数据库中有一个 **User** 表，其中的 `info` 字段存储了用户的个性化设置数据，这些数据采用 JSON 格式。可以使用 **MyBatisPlus** 提供的 JSON 处理器来自动将这个 JSON 数据与 Java 对象之间进行转换
> 
> ```json
> {
>     "age": 20,
>     "gender": "male",
> }
> ```
> ---
> 2. 通过 `UserInfo` 类，我们可以将该字段与 Java 对象映射。每当我们读取数据库时，**MyBatisPlus** 会自动将该 JSON 字符串转换为 `UserInfo` 对象，用户可以像访问普通 Java 对象字段一样访问
> 
> ```java
> @Data
> public class UserInfo {
>     private Integer age;  // 字段对应数据库中的JSON属性
>     private String gender;
> }
> ```
> ---
> 3. 业务查询数据库时，就可以使用
> 
> ```java
> System.out.println(userInfo.age()); //  20
> ```
> 