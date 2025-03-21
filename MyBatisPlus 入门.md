## 1 <font color="#ffc000">简述</font>
**<font color="#ffc000">MyBatis-Plus</font>** （简称 MP） 是在 MyBatis 基础上的增强框架 ^3428bf

**特点**：
1. 无侵入性
2. 代码生成器
3. 通用Mapper
4. 分页插件
5. 性能分析插件
6. 多数据源支持
......

---
---
## 2 <font color="#ffc000">基本实现</font>

### 2.1 <font color="#ffc000">导入依赖</font>
-  `MyBatisPlus` 官方提供了起步依赖 `starter`，其中集成了 `Mybatis` `和MybatisPlus` 的所有功能，并且实现了自动装配效果，可以替换掉 `Mybatis` 的起步依赖

```XML
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
```

---
### 2.2 <font color="#ffc000">继承`BaseMapper`接口</font>
为了简化单表CRUD，`MybatisPlus` 提供了一个基础的 `BaseMapper` 接口，其中已经实现了单表的CRUD：

![[MyBatisPlus BaseMapper接口.png]]

- `BaseMapper<泛型>` 的泛型指定操作的实体类

---
---
## 3 <font color="#ffc000">实现原理</font>
在 [[#2 <font color=" ffc000">基本实现</font>|2 基本实现]] 中，仅仅引入依赖，继承 `BaseMapper` 就能使用 `MybatisPlus`

> [!question]
> Q：MybatisPlus 如何知道要查询的是哪张表？表中有哪些字段呢？
> A：根据 PO 实体的信息来推断出表的信息，从而生成SQL的

**<font color="#ffc000">默认实现</font>**：

1. **表名推断**：实体类名通过驼峰转下划线的方式映射为表名
```
例：类名 User 驼峰转下划线映射为表名 user
```
2. **字段名推断**：实体类的字段名通过驼峰转下划线映射为数据库表的字段名，字段类型通过变量类型推断
```
例：实体类字段名 createTime 映射为数据库中的 create_time 字段
```
3. **主键推断**：默认将实体类中名为 id 的字段识别为主键

---
---

## 4 <font color="#ffc000">常用注解</font>
在很多情况下，`MybatisPlus` 的默认实现与实际场景不符，因此可以通过 `MybatisPlus` 提供的一些注解便于声明表信息

#### 4.1 <font color="#ffc000">@TableName</font>
**<font color="#ffc000">作用</font>**：
- 表名注解，标识实体类对应的表
- 使用位置：实体类

**<font color="#ffc000">示例</font>**：

```Java
@TableName("user")
public class User {
    private Long id;
    private String name;
}
```

---

### 4.2 <font color="#ffc000">@TableId</font>
**<font color="#ffc000">作用</font>**：
- 主键注解，标识实体类中的主键字段
- 使用位置：实体类的主键字段

**<font color="#ffc000">属性</font>**：
- `value`：字符串类型，表名
- `type`：枚举类型，主键生成策略

|**值**|**描述**|
|---|---|
|`IdType.AUTO`|数据库 ID 自增|
|`IdType.NONE`|无状态，该类型为未设置主键类型（注解里等于跟随全局，全局里约等于 INPUT）|
|`IdType.INPUT`|insert 前自行 set 主键值|
|`IdType.ASSIGN_ID`|分配 ID ，使用接口 `IdentifierGenerator` 的方法 `nextId` (默认实现类为 `DefaultIdentifierGenerator` 雪花算法)|


**<font color="#ffc000">示例</font>**：
```Java
@TableName("user")
public class User {
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    private String name;
}
```

---
### 4.3 <font color="#ffc000">@TableField</font>
**<font color="#ffc000">作用</font>**：
- 普通字段注解
- <font color="#ffc000">⚠️</font> 成员变量是以 `isXXX` 命名时，按照 `JavaBean` 的规范， `MybatisPlus` 识别字段时会把 `is` 去除，此时需要 `@TableField` 注解
-  <font color="#ffc000">⚠️</font> 成员变量名与数据库一致，但是与数据库的关键字冲突，使用 `@TableField` 注解给变量名添加转义字符： ` `` `

**<font color="#ffc000">示例</font>**：
```Java
@TableName("user")
public class User {
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
    @TableField("is_Married")
    private Boolean isMarried;
    @TableField("`concat`")
    private String concat;
}
```

---
---
## 5 <font color="#ffc000">常见配置</font>
在 Spring Boot 项目中，可以通过 *<font color="#ffc000">application.yml</font>* 或 *<font color="#ffc000">application.properties</font>* 文件来配置 `MyBatisPlus`

大多数的配置都有默认值，只有一些需要自定义配置的配置项：

- 实体类的别名扫描包
- 全局id类型
- ⚠️ 支持 `xml` 配置手写SQL

```YAML
mybatis-plus:
  type-aliases-package: com.itheima.mp.domain.po # 实体类的别名扫描包
  global-config: # MyBatis-Plus 的全局配置
    db-config: # 数据库相关配置
      id-type: auto # 全局id类型为自增长
```