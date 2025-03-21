MyBatisPlus 官方提供了许多插件工具

## 1 <font color="#ffc000">分页插件</font>

在未引入分页插件的情况下， `MybatisPlus` 是不支持分页功能的， `IService` 和 `BaseMapper` 中的分页方法都无法正常起效。 所以，我们必须配置分页插件

---
---

## 2 <font color="#ffc000">配置</font>

项目中新建配置类

<font color="#ffc000">*config -> MybatisConfig*</font>

```java
/**
 * Mybatis配置类
 **/
@Configuration
public class MybatisConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        // 1.初始化核心插件
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        // 2.创建分页插件
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);

        // 3.添加分页插件
        interceptor.addInnerInterceptor(paginationInnerInterceptor);

        return interceptor;
    }
}
```

---
---

## 3 <font color="#ffc000">示例</font>

### 3.1 <font color="#ffc000">需求</font>

定义一个用户分页查询接口，能根据分页条件动态查询

<font color="#ffc000">请求参数</font>：

```JSON
{
    "pageNo": 1,
    "pageSize": 5,
    "sortBy": "balance",
    "isAsc": false,
    "name": "o",
    "status": 1
}
```

<font color="#ffc000">返回值</font>：

```JSON
{
    "total": 100006,
    "pages": 50003,
    "list": [
        {
            "id": 1685100878975279298,
            "username": "user_9****",
            "info": {
                "age": 24,
                "intro": "英文老师",
                "gender": "female"
            },
            "status": "正常",
            "balance": 2000
        }
    ]
}
```

<font color="#ffc000">特殊说明</font>：

- 如果排序字段为空，默认按照更新时间排序
- 排序字段不为空，则按照排序字段排序

---

### 3.2 <font color="#ffc000">步骤</font>

1. 定义 **<font color="#ffc000">分页查询条件实体</font>** ，包含 <font color="#ffc000">分页参数</font>、<font color="#ffc000">排序参数</font>、<font color="#ffc000">过滤条件</font>
2. 定义 <font color="#ffc000">**自定义查询实体**</font> 来继承 <font color="#ffc000">**分页查询条件实体**</font>
3. 定义 <font color="#ffc000">**分页结果DTO**</font> ，包含 <font color="#ffc000">总条数</font>、<font color="#ffc000">总页数</font>、<font color="#ffc000">当前页数据</font>
4. 定义 **<font color="#ffc000">用户页面视图实体</font>** ，包含 <font color="#ffc000">若干用户信息</font>
5. 业务实现需求


---

### 3.3 <font color="#ffc000">实现</font>

> 1. <font color="#ffc000">分页查询条件实体</font> `PageQuery`
> 
>  *<font color="#ffc000">query -> PageQuery</font>*
> - 单独封装成一个类，方便其他自定义查询实体继承
> 
> ```Java
> /**  
>  * 分页查询条件实体  
>  **/  
> @Data  
> @ApiModel(description = "分页查询实体")  
> public class PageQuery {  
>   
>     @ApiModelProperty("页码")  
>     private Long pageNo;  
>   
>     @ApiModelProperty("每页显示数")  
>     private Long pageSize;  
>   
>     @ApiModelProperty("排序字段")  
>     private String sortBy;  
>   
>     @ApiModelProperty("是否升序")  
>     private Boolean isAsc;  
> }
> ```
> ---
>2. <font color="#ffc000">自定义查询实体 **继承** 分页查询条件实体</font>
> 
>  *<font color="#ffc000">query -> UserQuery</font>*
> 
> - 让自定义查询实体来继承分页查询条件实体
> - **`UserQuery`**：继承 **`PageQuery`**，添加特定的查询条件（如用户名、状态、余额范围等）以满足不同查询需求
> 
> ```java
> /***  
>  * 自定义查询条件实体  
>  */
> @Data  
> @ApiModel(description = "用户查询条件实体")  
> public class UserQuery extends PageQuery{ // 继承分页查询条件实体
>   
>     @ApiModelProperty("用户名关键字")  
>     private String name;  
>   
>     @ApiModelProperty("用户状态：1-正常，2-冻结")  
>     private Integer status;  
>   
>     @ApiModelProperty("余额最小值")  
>     private Integer minBalance;  
>   
>     @ApiModelProperty("余额最大值")  
>     private Integer maxBalance;  
> }
> ```
> ---
> 3. <font color="#ffc000">分页结果DTO `PageDTO`</font>
> 
> *<font color="#ffc000">dto -> PageDTO</font>*
> 
> - 动态结果集，由查询实体决定
> 
> ```java
> /**  
>  * 分页结果DTO  
>  **/
> @Data  
> @ApiModel(description = "分页结果")  
> public class PageDTO<T> {  
>   
>     @ApiModelProperty("总条数")  
>     private Long total;  
>   
>     @ApiModelProperty("总页数")  
>     private Long pages;  
>   
>     // 动态结果集，由查询实体决定  
>     @ApiModelProperty("分页结果集")  
>     private List<T> list;  
> }
> ```
> ---
> 4. <font color="#ffc000">用户页面视图实体 `UserVO`</font>
> 
>  *<font color="#ffc000">vo -> UserVO</font>*
> - 用于封装查询结果的显示视图，包含用户信息、状态、余额等字段
> ```java
> @Data  
> @ApiModel(description = "用户VO实体")  
> public class UserVO {  
>   
>     @ApiModelProperty("用户id")  
>     private Long id;  
>   
>     @ApiModelProperty("用户名")  
>     private String username;  
>   
>     @ApiModelProperty("详细信息")  
>     private UserInfo info;  
>   
>     @ApiModelProperty("使用状态（1正常 2冻结）")  
>     private UserStatus status;  
>   
>     @ApiModelProperty("账户余额")  
>     private Integer balance;  
>   
>     @ApiModelProperty("收获地址列表")  
>     private List<AddressVO> addresses;  
> }
> ```
> ---
> 
> 5. <font color="#ffc000">开发接口</font> `UserController`
> - 返回 分页结果集DTO `PageDTO` ，其中分页结果集为 `UserVO` 类型
> 
> ```java
> @RestController
> @RequestMapping("users")
> @RequiredArgsConstructor
> public class UserController {
> 
>     private final UserService userService;
> 
>     @GetMapping("/page")
>     public PageDTO<UserVO> queryUsersPage(UserQuery query){
>         return userService.queryUsersPage(query);
>     }
> 
>     // 。。。 
> }
> ```
> ---
> 6. <font color="#ffc000">业务实现</font>
> 
> *<font color="#ffc000">service -> impl -> UserServiceImpl</font>*
> 
> 
> ```java
> @Service  
> public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {  
> 
> 	@Override  
>     public PageDTO<UserVO> queryUsersPage(UserQuery userQuery) {  
>   
>         String name = userQuery.getName();  
>         Integer status = userQuery.getStatus();  
>   
>         // 1.构建分页条件  
>         Page<User> page = Page.of(userQuery.getPageNo(), userQuery.getPageSize());  
>   
>         // 2.校验排序条件是否为空  
>         if (userQuery.getSortBy() != null) {  
>             // 不为空，构建排序条件  
>             page.addOrder(new OrderItem(userQuery.getSortBy(), userQuery.getIsAsc()));  
>         } else {  
>             // 为空，默认按照更新时间排序  
>             page.addOrder(new OrderItem("update_time", false));  
>         }  
>   
>         // 3.分页查询  
>         Page<User> p = lambdaQuery()  
>                 .like(name != null, User::getUsername, name)  
>                 .eq(status != null, User::getStatus, status)  
>                 .page(page);  
>   
>         // 4.封装返回结果  
>         PageDTO<UserVO> result = new PageDTO<UserVO>();  
>         result.setTotal(p.getTotal()); // 总条数  
>         result.setPages(p.getPages()); // 总页数  
>   
>         if (CollUtil.isEmpty(p.getRecords())) {  
>             result.setList(Collections.emptyList()); // 如果是空数据，返回空集合  
>         }  
>   
>         // 将User转为VO  
>         List<UserVO> userVOs = BeanUtil.copyToList(p.getRecords(), UserVO.class);  
>         result.setList(userVOs); // 总数据  
>   
>         return result;  
>     } 
>      
> }
> ```

---
---

## 4 <font color="#ffc000">简化</font>（示例实现）

在 [[#3.3 <font color=" ffc000">实现</font>|3.1 实现]] 中，业务实现的代码比较复杂，可以改造 <font color="#ffc000">分页查询条件实体</font> `PageQuery` 与 <font color="#ffc000">分页结果DTO</font> `PageDTO` 来简化代码

### 4.1 <font color="#ffc000">分页查询条件实体</font> `PageQuery`

```Java
/**  
 * 分页查询条件实体  
 **/  
@Data  
@ApiModel(description = "分页查询实体")  
public class PageQuery {  
  
    @ApiModelProperty("页码")  
    private Long pageNo;  
  
    @ApiModelProperty("每页显示数")  
    private Long pageSize;  
  
    @ApiModelProperty("排序字段")  
    private String sortBy;  
  
    @ApiModelProperty("是否升序")  
    private Boolean isAsc;  

    public <T>  Page<T> toMpPage(OrderItem ... orders){
        // 1.分页条件
        Page<T> p = Page.of(pageNo, pageSize);
        // 2.排序条件
        // 2.1.先看前端有没有传排序字段
        if (sortBy != null) {
            p.addOrder(new OrderItem(sortBy, isAsc));
            return p;
        }
        // 2.2.再看有没有手动指定排序字段
        if(orders != null){
            p.addOrder(orders);
        }
        return p;
    }

    public <T> Page<T> toMpPage(String defaultSortBy, boolean isAsc){
        return this.toMpPage(new OrderItem(defaultSortBy, isAsc));
    }

    public <T> Page<T> toMpPageDefaultSortByCreateTimeDesc() {
        return toMpPage("create_time", false);
    }

    public <T> Page<T> toMpPageDefaultSortByUpdateTimeDesc() {
        return toMpPage("update_time", false);
    }
}
```
---

### 4.2 <font color="#ffc000">分页结果DTO</font> `PageDTO`

```Java
/**  
 * 分页结果DTO  
 **/
@Data  
@ApiModel(description = "分页结果")  
public class PageDTO<T> {  
  
    @ApiModelProperty("总条数")  
    private Long total;  
  
    @ApiModelProperty("总页数")  
    private Long pages;  
  
    // 动态结果集，由查询实体决定  
    @ApiModelProperty("分页结果集")  
    private List<T> list;  

    /**
     * 返回空分页结果
     * @param p MybatisPlus的分页结果
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> empty(Page<P> p){
        return new PageDTO<>(p.getTotal(), p.getPages(), Collections.emptyList());
    }

    /**
     * 将MybatisPlus分页结果转为 VO分页结果
     * @param p MybatisPlus的分页结果
     * @param voClass 目标VO类型的字节码
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> of(Page<P> p, Class<V> voClass) {
        // 1.非空校验
        List<P> records = p.getRecords();
        if (records == null || records.size() <= 0) {
            // 无数据，返回空结果
            return empty(p);
        }
        // 2.数据转换
        List<V> vos = BeanUtil.copyToList(records, voClass);
        // 3.封装返回
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);
    }

    /**
     * 将MybatisPlus分页结果转为 VO分页结果，允许用户自定义PO到VO的转换方式
     * @param p MybatisPlus的分页结果
     * @param convertor PO到VO的转换函数
     * @param <V> 目标VO类型
     * @param <P> 原始PO类型
     * @return VO的分页对象
     */
    public static <V, P> PageDTO<V> of(Page<P> p, Function<P, V> convertor) {
        // 1.非空校验
        List<P> records = p.getRecords();
        if (records == null || records.size() <= 0) {
            // 无数据，返回空结果
            return empty(p);
        }
        // 2.数据转换
        List<V> vos = records.stream().map(convertor).collect(Collectors.toList());
        // 3.封装返回
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);
    }
}
```