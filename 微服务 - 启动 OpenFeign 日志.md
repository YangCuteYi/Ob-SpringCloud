 `OpenFeign` 默认不会输入日志，输出日志可能影响性能

在需要调试时，可以手动调整日志级别，从而开启日志

- **<font color="#ffc000">NONE</font>**：不记录任何日志信息，这是默认值
    
- **<font color="#ffc000">BASIC</font>**：仅记录请求的方法，URL以及响应状态码和执行时间
    
- **<font color="#ffc000">HEADERS</font>**：在BASIC的基础上，额外记录了请求和响应的头信息
    
- **<font color="#ffc000">FULL</font>**：记录所有请求和响应的明细，包括头信息、请求体、元数据

## 1 <font color="#ffc000">定义日志级别</font>

<font color="#ffc000">*hm-api*</font> -> ... -> *<font color="#ffc000">config</font>* 下定义类，并设置日志级别信息

```java
/**  
 * Feign 配置类，用于设置 Feign 的日志级别  
 **/  
public class DefaultFeignConfig {  
  
    /***  
     * 配置 Feign 的日志级别  
     * ---  
     * Logger.Level.FULL 返回 FULL 日志级别，记录所有请求和响应的详细信息  
    */  
    @Bean  
    public Logger.Level feignLogLevel() {  
        return Logger.Level.FULL;  
    }  
}
```


## 2 <font color="#ffc000">配置</font>
要让日志级别生效，还需要配置这个类，有两种方式

### 2.1 <font color="#ffc000">局部生效</font>

在某个 `FeignClient` 中配置，只对当前 `FeignClient` 生效

```java
/***  
 * 用于通过 Feign 访问 item-service 微服务，提供商品信息查询服务
 * 指定日志配置
 */  
@FeignClient(value = "item-service", configuration = DefaultFeignConfig.class)  
public interface ItemClient {  

	...
  
}
```

- `@FeignClient` 的 `configuration` 属性指定配置日志级别的类

---
### 2.2 <font color="#ffc000">全局生效</font>

在启动类的 `@EnableFeignClients` 中配置，针对当前微服务下所有 `FeignClient` 生效

```java
// 启用 Feign 客户端支持，允许通过 Feign 调用其他微服务，声明客户端
// 指定日志配置
@EnableFeignClients(clients = {ItemClient.class}, defaultConfiguration = DefaultFeignConfig.class)  
@MapperScan("com.hmall.cart.mapper")  
@SpringBootApplication  
public class CartApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(CartApplication.class, args);  
    }  
}
```