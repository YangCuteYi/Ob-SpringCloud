## 1 <font color="#ffc000">条件构造器</font>
条件构造器（`Wrapper`）是 <font color="#ffc000">**MyBatisPlus**</font> 提供的用于动态生成 SQL 的工具类，将条件构造器作为参数传递给 `BaseMapper` 的方法，可以实现自定义条件查询

![[MyBatisPlus 条件构造器API.png]]

`Wrapper` 抽象类默认实现子类：
- <font color="#ffc000">QueryWrapper</font>：用于构建查询条件
- <font color="#ffc000">UpdateWrapper</font>：用于构建更新条件
- <font color="#ffc000">LambdaQueryWrapper</font>：使用 Lambda 表达式构建查询条件
- <font color="#ffc000">LambdaUpdateWrapper</font>：使用 Lambda 表达式构建更新条件

---
---
## 2 <font color="#ffc000">常用构造方法</font>

|**方法**     | **功能**    |
| --- | --- |
| eq    | 等值条件    |
| ne    | 不等条件    |
| gt    | 大于条件    |
| ge    | 大于等于条件   |
| lt   | 小于条件    |
| le   | 小于等于条件    |
| between    | 区间条件    |
| like    | 模糊查询    |
| notLike    | 非模糊查询    |
| orderByAsc    | 按升序排序    |
| orderByDesc    | 按降序排序    |
| isNull    | 判断字段为空    |
| isNotNull    | 判断字段不为空    |
| in    | 包含条件    |
| notIn    | 不包含条件    |

---
---
## 3 <font color="#ffc000">QueryWrapper</font>
**<font color="#ffc000">作用</font>**：用于构建查询条件

**<font color="#ffc000">示例</font>**：
> 1. 需求：查询出名字中带`o`的，存款大于等于1000元的`id, username`字段
> 
> ```Java
> @Test
> void testQueryWrapper() {
>     // 1.构建查询条件 where name like "%o%" AND balance >= 1000
>     QueryWrapper<User> wrapper = new QueryWrapper<User>()
>             .select("id", "username")
>             .like("username", "o")
>             .ge("balance", 1000);
>             
>     // 2.查询数据
>     List<User> users = userMapper.selectList(wrapper);
>     users.forEach(System.out::println);
> }
> ```
> 
> 2. 需求：更新用户名为 `jack` 的用户的余额为2000
> ```Java
> @Test
> void testUpdateByQueryWrapper() {
>     // 1.构建查询条件 where name = "Jack"
>     QueryWrapper<User> wrapper = new QueryWrapper<User>().eq("username", "Jack");
>     
>     // 2.更新数据，user中非null字段都会作为set语句
>     User user = new User();
>     user.setBalance(2000);
>     userMapper.update(user, wrapper);
> }
> ```

---
---
## 4 <font color="#ffc000">UpdateWrapper</font>
**<font color="#ffc000">作用</font>**：用于构建更新条件

**<font color="#ffc000">示例</font>**：

需求：更新 `id` 为 `1,2,4` 的用户的余额，扣200

```js
@Test  
void testUpdateWrapper() {  
    // 1.构造更新条件  
    UpdateWrapper<User> UpdateWrapper = new UpdateWrapper<User>()  
            .setSql("balance = balance - 200")  // 自定义SQL
            .in("id",1,2,4);  
  
    // 2.更新数据  
    userMapper.update(null, UpdateWrapper);  
}
```


---
---
## 5 <font color="#ffc000">LambdaQueryWrapper</font>
在 `QueryWrapper` 与 `UpdateWrapper` 构造条件时，使用字符串直接嵌入代码中，是一种 <font color="#ffc000">字符串魔法值</font> 现象，并不规范


> [!question] 
> Q：如何不写字段名，又能知道字段名呢？（避免魔法值）
> A：使用基于变量的 `gettter` 方法结合反射技术 -> `方法引用`和`Lambda`表达式

**<font color="#ffc000">作用</font>**：以 Lambda 表达式方式构建查询条件的工具类，可以避免使用字符串硬编码（魔法值）

**<font color="#ffc000">示例</font>**：
> 1. 需求：查询出名字中带`o`的，存款大于等于1000元的`id, username`字段
> 
> ```Java
> @Test
> void testLambdaQueryWrapper() {
>     // 1.构建查询条件 where username like "%o%" AND balance >= 1000
>     LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
>             .select(User::getId, User::getUsername) // 指定返回字段
>             .like(User::getUsername, "o") // 模糊查询
>             .ge(User::getBalance, 1000); // 大于等于条件
> 
>     // 2.查询数据
>     List<User> users = userMapper.selectList(wrapper);
>     users.forEach(System.out::println);
> }
> ```
> 
> 2. 需求：查询 `status` 为正常（1）的用户，按 `balance` 降序排列
> 
> ```Java
> @Test
> void testLambdaQueryWrapperOrderBy() {
>     // 1.构建查询条件 where status = 1 order by balance DESC
>     LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
>             .eq(User::getStatus, 1) // 等值查询
>             .orderByDesc(User::getBalance); // 按余额降序排列
> 
>     // 2.查询数据
>     List<User> users = userMapper.selectList(wrapper);
>     users.forEach(System.out::println);
> }
> ```

---
---
## 6 <font color="#ffc000">自定义SQL</font>
**<font color="#ffc000">引言</font>**：

*在 1 - 5 笔记中，使用条件构造器很方便的完成了 SQL 查询，但是这些查询语句都写在了业务代码中，这在企业级开发中是不规范的*

*同时，复杂查询条件（如 `in` 查询），只能写在 <font color="#ffc000">Mapper.xml</font> ，过于麻烦*

💡 可以利用 **MyBatisPlus** 的 `Wrapper` 来构建复杂的 `where` 条件，然后自己结合 *<font color="#ffc000">Mapper.xml</font>* 定义 SQL 语句中剩下的部分

**<font color="#ffc000">示例</font>**：

> 需求：批量更新 `id` 为 1,2,4 的用户余额，每人增加 200
> 
> ---
> 实现：
> 
> 1. <font color="#ffc000">构建条件</font>
> 
> 使用 `LambdaQueryWrapper` 构造 `where` 条件，指定 `id` 在 1,2,4 范围内：
> 
> ```java
> List<Long> ids = List.of(1L, 2L, 4L); // 待更新的用户ID集合
> int amount = 200; // 更新金额
> 
> // 构建条件：where id in (1, 2, 4)
> LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
>         .in(User::getId, ids);
> ```
> 
> 2. <font color="#ffc000">Mapper方法声明</font>
> 
> 在 `Mapper` 接口中声明自定义 SQL 方法，使用 `@Param` 注解将参数传入
> 
> ```java
> void updateBalanceByIds(@Param("ew") LambdaQueryWrapper<User> wrapper, @Param("amount") int amount);
> ```
> 
> - ⚠️ `Wrapper` 参数需要用 `@Param("ew")` 标注，名称必须为 `ew`，否则无法解析条件
> - `ew` 即为 `Constants.WRAPPER`
> 
> 3. <font color="#ffc000">自定义SQL语句</font>
> 
> 在 *<font color="#ffc000">Mapper.xml</font>* 中定义自定义 SQL，结合 `Wrapper` 动态条件：
> 
> ```java
> <update id="updateBalanceByIds">
>     UPDATE user
>     SET balance = balance + #{amount} 
>     ${ew.customSqlSegment}
> </update>
> ```
> - ⚠️ `${ew.customSqlSegment}`：`Wrapper` 解析生成的 `where` 条件部分（固定写法）
> 
> 4. <font color="#ffc000">调用自定义SQL</font>
> 
> 在业务代码中调用 `Mapper` 方法
> 
> ```java
> userMapper.updateBalanceByIds(wrapper, amount);
> ```
> 
> 完整Demo：[[MyBatisPlus - 自定义SQL]]

---
---

## 7 <font color="#ffc000">通用Service接口</font>
**MybatisPlus** 不仅提供了 **BaseMapper**，还提供了通用的 **Service** 接口及默认实现，封装了一些常用的 **service** 模板方法：[模版方法](https://b11et3un53m.feishu.cn/wiki/PsyawI04ei2FQykqfcPcmd7Dnsc#Tgrvd7xRsoBE07xgmB4cYk4Hnce)

### 7.1 <font color="#ffc000">核心</font>
1. `IService`：通用接口，需要被自定义 `Service` 接口 **<font color="#ffc000">继承</font>**
```java
public interface IService<T> {
    boolean save(T entity);
    boolean saveBatch(Collection<T> entityList);
    boolean updateById(T entity);
    T getById(Serializable id);
    List<T> list(Wrapper<T> queryWrapper);
    // 其他通用方法...
}
```

2. `ServiceImpl`：默认实现，封装了对 `BaseMapper` 的调用逻辑，需要被自定义 `Service` 接口实现类 **<font color="#ffc000">继承</font>**
```java
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {
    @Autowired
    protected M baseMapper;

    @Override
    public boolean save(T entity) {
        return retBool(baseMapper.insert(entity));
    }

    @Override
    public T getById(Serializable id) {
        return baseMapper.selectById(id);
    }

    // 其他方法封装...
}
```

![[MyBatisPlus 通用Service接口.png]]

---

### 7.2  <font color="#ffc000">使用流程</font>
1. 自定义 `Service` 接口 <font color="#ffc000">继承</font> `IService` 接口
- 泛型为对应的实体类类型
```java
public interface CustomService extends IService<Entity> {

}
```

2. 自定义 `Service` 实现类，实现自定义接口并 <font color="#ffc000">继承</font> `Servicelmpl` 类

```java
@Service  
public class CustomServiceImpl extends ServiceImpl<CustomMapper,Entity> implements CustomService{  
  
}
```
- ⚠️ 继承 `ServiceImpl` 类，范型指定 `mapper` 的类型和实体类的类型

---
---
### 7.3 <font color="#ffc000">基于Lambda查询</font>

**<font color="#ffc000">引言</font>**：

*IService 中对 `LambdaQueryWrapper` 和 `LambdaUpdateWrapper` 的用法进一步做了简化；我们无需自己通过 `new` 的方式来创建 `Wrapper` ，而是直接调用 `lambdaQuery` 和 `lambdaUpdate`* 

**<font color="#ffc000">示例</font>**：

> 需求：根据复杂条件查询用户
> 
> ---
> 
> 实现1：
> 
> <font color="#ffc000">使用 `LambdaQueryWrapper` 构造查询条件</font>
> 
> ```java
> 	// 1.使用 LambdaQueryWrapper 构造查询条件
>     LambdaQueryWrapper<User> wrapper = new QueryWrapper<User>().lambda()
>             .like(username != null, User::getUsername, username)
>             .eq(status != null, User::getStatus, status)
>             .ge(minBalance != null, User::getBalance, minBalance)
>             .le(maxBalance != null, User::getBalance, maxBalance);
> 
> 	// 2.使用 Service 层调用查询方法
> 	List<User> users = userService.list(wrapper);
> 		
> ```
> - `username != null` 这样的参数：当条件成立时才会添加这个查询条件，`类似Mybatis` 的 *<font color="#ffc000">mapper.xml</font>* 文件中的 `<if>` 标签
> ---
> 实现2：
> 
> ⚠️ <font color="#ffc000">使用 `IService` 的基于 `Lambda` 简化查询</font>
> - `Service` 直接调用 `lambdaQuery`
> ```Java
>     List<User> users = userService.lambdaQuery()
>             .like(username != null, User::getUsername, username)
>             .eq(status != null, User::getStatus, status)
>             .ge(minBalance != null, User::getBalance, minBalance)
>             .le(maxBalance != null, User::getBalance, maxBalance)
>             .list();
> ```
> - `.one()`：最多1个结果
> - `.list()`：返回集合结果
> - `.count()`：返回计数结果

---
---
## 8 <font color="#ffc000">批量新增</font> - `rewriteBatchedStatements`

**<font color="#ffc000">引言</font>**：

*使用 `MyBatisPlus` 提供的方法批量插入数据时，通常会遇到性能瓶颈。通过 `rewriteBatchedStatements` 配置，可以使得批量插入变得更加高效*

<font color="#ffc000">**配置**</font>：

在 *<font color="#ffc000">application.yml</font>* 配置文件中，启用 `rewriteBatchedStatements` 配置：

```YAML
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/mp?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: xxx
```

或者

```YAML e
mybatis-plus:
  global-config:
    db-config:
      rewrite-batched-statements: true
```

启用该配置后，`MyBatisPlus` 在进行批量插入时，会通过<font color="#ffc000">一次</font> SQL 批量执行插入操作，而不是逐条插入