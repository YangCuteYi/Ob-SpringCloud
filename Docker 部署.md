从零开始，用 `Docker` 部署Java项目

## 1 <font color="#ffc000">部署后端</font>

1. <font color="#ffc000">打包项目</font>

![[Docker 部署1.png]]

---
2. <font color="#ffc000">可以看到打包好的 `jar` 包</font>

![[Docker 部署2.png]]

---

3. <font color="#ffc000">编写 `Dockerfile` 文件</font>

```Dockerfile
# 基础镜像  
FROM openjdk:11.0-jre-buster  
# 设定时区  
ENV TZ=Asia/Shanghai  
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone  
# 拷贝jar包  
COPY hm-service.jar /app.jar  
# 入口  
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
---

4.  <font color="#ffc000">将 `jar` 包与 `Dockerfile` 文件一同上传到虚拟机的 `/root` 目录</font>

![[Docker 部署3.png]]

---
5. <font color="#ffc000">构建项目镜像</font>
- 在 /root 目录下运行

```bash
docker build -t hmall .
```

---
6. <font color="#ffc000">创建自定义网络</font>
- 创建自定义网络 `test`
```bash
docker network create test
```

---
7. <font color="#ffc000">创建 `mysql` 容器，并加入到自定义网络</font>

```bash
docker run -d --name mysql --network test -p 3306:3306 mysql
```

---
8. <font color="#ffc000">创建并运行容器</font>
- 通过 `--network` 将其加入自定义网络 `test`

```bash
docker run -d --name hmall --network test -p 8080:8080 hmall
```

---
---
## 2 <font color="#ffc000">部署前端</font>



[Docker 部署前端](https://b11et3un53m.feishu.cn/wiki/MWQIw4Zvhil0I5ktPHwcoqZdnec#LkA1ddAsDo1TASxvtiQcScVznNc)

---
---

## 3 <font color="#ffc000">DockerCompose</font>

前面部署前后端的操作，非常繁琐

使用 **Docker Compose** 就可以帮助我们实现 **<font color="#ffc000">多个相互关联的 `Docker` 容器的快速部署</font>**

使用一个单独的 `docker-compose.yml` 模板文件（YAML 格式），来定义一组相关联的应用容器

### 3.1 <font color="#ffc000">语法</font>

`docker-compose` 文件中可以定义多个相互关联的应用容器，每一个应用容器被称为一个服务（`service`）

|**docker run 参数**|**docker compose 指令**|**说明**|
|---|---|---|
|--name|container_name|容器名称|
|-p|ports|端口映射|
|-e|environment|环境变量|
|-v|volumes|数据卷配置|
|--network|networks|网络|
|...|...|...|

<font color="#ffc000">示例</font>：

> 用 `docker run` 部署 `mysql`
> 
> ```Bash
> docker run -d \
>   --name mysql \
>   -p 3306:3306 \
>   -e TZ=Asia/Shanghai \
>   -e MYSQL_ROOT_PASSWORD=123 \
>   -v /root/opt/mysql/data:/var/lib/mysql \
>   -v /root/opt/mysql/conf:/etc/mysql/conf.d \
>   -v /root/optmysql/init:/docker-entrypoint-initdb.d \
>   --network hmall
>   mysql
> ```
> ---
> 用 `docker-compose.yml` 定义
> 
> ```YAML
> version: "3.8"
> 
> services:
>   mysql:
>     image: mysql
>     container_name: mysql
>     ports:
>       - "3306:3306"
>     environment:
>       TZ: Asia/Shanghai
>       MYSQL_ROOT_PASSWORD: 123
>     volumes:
>       - "/root/opt/mysql/conf:/etc/mysql/conf.d"
>       - "/root/opt/mysql/data:/var/lib/mysql"
>       - "/root/optmysql/init:/docker-entrypoint-initdb.d"
>     networks:
>       - new
> networks:
>   new:
>     name: hmall
> ```

---

### 3.2 <font color="#ffc000">命令</font>

```Bash
docker compose [OPTIONS] [COMMAND]
```

|**类型**|**参数或指令**|**说明**|
|---|---|---|
|Options|-f|指定compose文件的路径和名称|
|-p|指定project名称。project就是当前compose文件中设置的多个service的集合，是逻辑概念|
|Commands|up|创建并启动所有service容器|
|down|停止并移除所有容器、网络|
|ps|列出所有启动的容器|
|logs|查看指定容器的日志|
|stop|停止容器|
|start|启动容器|
|restart|重启容器|
|top|查看运行的进程|
|exec|在指定的运行中容器中执行命令|

---
### 3.3 <font color="#ffc000">案例实现</font>

对 [[#1 <font color=" ffc000">部署后端</font>| 1 部署后端]] 和 [[#2 <font color=" ffc000">部署前端</font>| 2 部署前端]] ，用 `docker-compose.yml` 定义实现

1. <font color="#ffc000">定义 `docker-compose.yml` </font>

```yml
version: "3.8"

services:
  mysql:
    user: root
    image: mysql:8.0
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "/root/opt/mysql/conf:/etc/mysql/conf.d"
      - "/root/opt/mysql/data:/var/lib/mysql"
      - "/root/opt/mysql/init:/docker-entrypoint-initdb.d"
    networks:
      - hm-net
  hmall:
    user: root
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: hmall
    ports:
      - "8080:8080"
    networks:
      - hm-net
    depends_on:
      - mysql
  nginx:
    user: root
    image: nginx
    container_name: nginx
    ports:
      - "18080:18080"
      - "18081:18081"
    volumes:
      - "/root/nginx/nginx.conf:/etc/nginx/nginx.conf"
      - "/root/nginx/html:/usr/share/nginx/html"
    depends_on:
      - hmall
    networks:
      - hm-net
networks:
  hm-net:
    name: hmall
```

2. <font color="#ffc000">创建并启动所有 `service` 容器</font>

```bash
docker compose up -d
```

3. <font color="#ffc000">停止所有 `service` 容器</font>

```
docker compose stop
```

4. <font color="#ffc000">启动所有 `service` 容器</font>
```
docker compose start
```

5. <font color="#ffc000">停止并移除所有容器、网络</font>
```
docker compose down
```