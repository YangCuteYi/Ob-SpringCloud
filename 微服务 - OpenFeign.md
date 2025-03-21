[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)[](微服务%20-%20网关服务.md)`OpenFeign` 是一个声明式的 **Web** 服务客户端，它简化了与其他服务的通信。通过 `OpenFeign`，开发者可以直接使用注解来声明 `RESTful API` 请求，而无需手动编写 `HTTP` 客户端代码

## 1 <font color="#ffc000">入门案例</font>

以 [[微服务 - 服务治理问题]] 引言为例，在购物车服务中访问商品服务，获取商品信息

<font color="#ffc000">实现</font>：
> 
> 1. <font color="#ffc000">引入依赖</font>
> 
> 引入 `OpenFeign` 的依赖和 `loadBalancer` 依赖
> 
> ```XML
>   <!--openFeign-->
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-openfeign</artifactId>
>   </dependency>
>   <!--负载均衡器-->
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
>   </dependency>
> ```
> 
> ---
> 2. <font color="#ffc000">启用 `OpenFeign`</font>
> 
> 在启动类上添加 `@EnableFeignClients` 注解
> 
> ```java
> @EnableFeignClients // 启用 Feign 客户端支持，允许通过 Feign 调用其他微服务
> @MapperScan("com.hmall.cart.mapper")  
> @SpringBootApplication  
> public class CartApplication {  
>     public static void main(String[] args) {  
>         SpringApplication.run(CartApplication.class, args);  
>     }  
> }
> ```
> 
> ---
> 3. <font color="#ffc000">编写 `OpenFeign` 客户端</font>
> 
> 在购物车服务中创建 *<font color="#ffc000">client</font>* 包，其中定义接口 `ItemClient` ，编写 `Feign` 客户端
> 
> ```java
> /***  
>  * ItemClient 接口  
>  * 用于通过 Feign 访问 item-service 微服务，提供商品信息查询服务  
>  */  
> @FeignClient("item-service") // 访问 item-service 服务  
> public interface ItemClient {  
>   
>     // 指定 GET 请求路径为 /items    
>     @GetMapping("/items")  
>     List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);  
>   
> }
> ```
> 

> [!example] 关键
> - `@FeignClient("item-service")`：声明服务名称
> - `@GetMapping`：声明请求方式
> - `@GetMapping("/items")`：声明请求路径
> - `@RequestParam("ids") Collection<Long> ids` ：声明请求参数
> - `List<ItemDTO>` ：返回值类型

> ---
> 
> 4. <font color="#ffc000">使用 `OpenFeign`</font>
> 
> - `final + @RequiredArgsConstructor` 依赖注入 `ItemClient` 接口
> 
> ```java
> 
> ...
> 
> // final + @RequiredArgsConstructor 依赖注入  
> private final ItemClient itemClient;
> 
> ...
> 
> // 根据服务名称获取服务实例列表  
> List<ServiceInstance> instances = discoveryClient.getInstances("服务名称"); 
> 
> // 负载均衡，从实例列表中挑选一个示例  
> ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));  
>   
> // 获取服务实例IP  
> URI uri = instance.getUri();
> 
> // 后续业务
> ...
> ```

---
---

## 2 <font color="#ffc000">连接池</font>

在微服务架构中，频繁的服务调用会导致大量的 `HTTP` 请求，而每次发起请求时都需要建立连接，为了优化这种性能瓶颈，可以使用 **<font color="#ffc000">连接池</font>** 来复用已有连接，减少连接建立和销毁的成本

而 `OpenFeign` 底层使用的默认框架为 `HttpURLConnection` ，是不支持连接池的

<font color="#ffc000">因此我们通常会使用带有连接池的客户端来代替默认的 `HttpURLConnection`</font>


> [!example] 常见连接池框架
> - HttpURLConnection：默认实现，不支持连接池
> 
> - Apache HttpClient ：支持连接池
>   
> - OKHttp：支持连接池



<font color="#ffc000">实现</font>：
> 1. <font color="#ffc000">引入依赖</font>
> 
> ```XML
> <!--OK http 的依赖 -->
> <dependency>
>   <groupId>io.github.openfeign</groupId>
>   <artifactId>feign-okhttp</artifactId>
> </dependency>
> ```
> 
> ---
> 
> 2. <font color="#ffc000">开启连接池</font>
> 
> *<font color="#ffc000">application.yml</font>* 配置文件中开启 `Feign` 的连接池功能
> 
> ```YAML
> feign:
>   okhttp:
>     enabled: true # 开启OKHttp功能
> ```
> 
> ---
> 
> 3. <font color="#ffc000">验证</font>
> 
> `org.springframework.cloud.openfeign.loadbalancer.FeignBlockingLoadBalancerClient` 的 `execute` 方法 <font color="#ffc000">断点</font> 执行
> 
> ![[微服务 OpenFeign验证是否更改框架.png]]

---
---

## 3 <font color="#ffc000">最优结构</font>

使用 `OpenFeign` 后，定义的客户端有可能在其他服务中也需要使用，这样会导致重复编码

使用 <font color="#ffc000">抽取</font> 的思想，来避免重复编码

- <font color="#ffc000">思想1</font>：将重复编码的客户端抽取到一个公共的模块（<font color="#ffc000">项目耦合度偏高</font>）
- <font color="#ffc000">思想2</font>：每个微服务内部抽取相关代码到一个模块（<font color="#ffc000">耦合度降低</font>）

![[微服务 抽取思想.png]]

<font color="#ffc000">实现</font>：[[微服务 - 基于 OpenFeign 最佳项目结构]]


---
---

## 4 <font color="#ffc000">日志配置</font>

`OpenFeign` 只会在 `FeignClient` 所在包的日志级别为** `DEBUG` **时，才会输出日志，其日志级别有4级：

- **<font color="#ffc000">NONE</font>**：不记录任何日志信息，这是默认值
    
- **<font color="#ffc000">BASIC</font>**：仅记录请求的方法，URL以及响应状态码和执行时间
    
- **<font color="#ffc000">HEADERS</font>**：在BASIC的基础上，额外记录了请求和响应的头信息
    
- **<font color="#ffc000">FULL</font>**：记录所有请求和响应的明细，包括头信息、请求体、元数据
    

启动日志：[[微服务 - 启动 OpenFeign 日志]]