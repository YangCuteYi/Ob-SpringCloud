使用 `OpenFeign` 后，定义的客户端有可能在其他服务中也需要使用，这样会导致重复编码

使用 <font color="#ffc000">抽取</font> 的思想，来避免重复编码

- <font color="#ffc000">思想1</font>：将重复编码的客户端抽取到一个公共的模块（<font color="#ffc000">项目耦合度偏高</font>）
- <font color="#ffc000">思想2</font>：每个微服务内部抽取相关代码到一个模块（<font color="#ffc000">耦合度降低</font>）

##  以 `hmall` 项目为例 - <font color="#ffc000">思想1</font>

抽取  *<font color="#ffc000">cart-service</font>* 服务下的 `OpenFeign` 客户端，到新模块 *<font color="#ffc000">hm-api</font>* 服务中

### 1 <font color="#ffc000">定义公共 `api` 服务</font>

在父工程下定义 `hm-api` 模块

![[微服务 定义公共API模块.png]]


### 2 <font color="#ffc000">引入依赖</font>

*<font color="#ffc000">hm-api </font> ->  <font color="#ffc000">pom.xml</font>*

```xml
<!--openFeign-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-openfeign</artifactId>  
</dependency>  
<!--负载均衡器-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
</dependency>
```

### 3 <font color="#ffc000">抽取公共部分</font>

将 *<font color="#ffc000">cart-service</font>* 服务下的 `OpenFeign` 客户端，以及必要部分 `ItemDTO` 抽取到 *<font color="#ffc000">hm-api</font>* 

![[微服务 抽取公共部分.png]]


### 4 <font color="#ffc000">使用者引入 `api` 依赖</font>

任何微服务要调用 `OpenFeign` 客户端处理的接口，只需要引入 `hm-api` 模块依赖即可，无需在自己服务中编写 `Feign` 客户端

*<font color="#ffc000">cart-service </font> ->  <font color="#ffc000">pom.xml</font>*

```XML
  <!--feign模块-->
  <dependency>
      <groupId>com.heima</groupId>
      <artifactId>hm-api</artifactId>
      <version>1.0.0</version>
  </dependency>
```


### 5 <font color="#ffc000">扫描包配置</font>


> [!BUG] 
> Q：启动时显示 `OpenFeign` 客户端未找到
> A：这是因为 `OpenFeign` 客户端被定义到了 *<font color="#ffc000">hm-api</font>* 中，启动类无法扫描
> S：
> - <font color="#ffc000">方法一</font>：在使用者服务启动类上声明扫描包
> ```java
> // 启用 Feign 客户端支持，允许通过 Feign 调用其他微服务，并声明扫描包
> @EnableFeignClients(basePackages = "com.hmall.api.client") 
> ```
> - <font color="#ffc000">方法二</font>：在使用者服务启动类上声明 `Feign` 客户端
> ```java
> // 启用 Feign 客户端支持，允许通过 Feign 调用其他微服务，声明客户端
> @EnableFeignClients(clients = {ItemClient.class}) 
> ```
