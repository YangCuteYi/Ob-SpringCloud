## 1 <font color="#ffc000">常用命令</font>

![[Docker 命令使用例图.png]]

|**命令**|**说明**|**文档地址**|
|---|---|---|
|docker pull|拉取镜像|[docker pull](https://docs.docker.com/engine/reference/commandline/pull/)|
|docker push|推送镜像到DockerRegistry|[docker push](https://docs.docker.com/engine/reference/commandline/push/)|
|docker images|查看本地镜像|[docker images](https://docs.docker.com/engine/reference/commandline/images/)|
|docker rmi|删除本地镜像|[docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)|
|docker run|创建并运行容器（不能重复创建）|[docker run](https://docs.docker.com/engine/reference/commandline/run/)|
|docker stop|停止指定容器|[docker stop](https://docs.docker.com/engine/reference/commandline/stop/)|
|docker start|启动指定容器|[docker start](https://docs.docker.com/engine/reference/commandline/start/)|
|docker restart|重新启动容器|[docker restart](https://docs.docker.com/engine/reference/commandline/restart/)|
|docker rm|删除指定容器|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/rm/)|
|docker ps|查看容器|[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)|
|docker logs|查看容器运行日志|[docker logs](https://docs.docker.com/engine/reference/commandline/logs/)|
|docker exec|进入容器|[docker exec](https://docs.docker.com/engine/reference/commandline/exec/)|
|docker save|保存镜像到本地压缩文件|[docker save](https://docs.docker.com/engine/reference/commandline/save/)|
|docker load|加载本地压缩文件到镜像|[docker load](https://docs.docker.com/engine/reference/commandline/load/)|
|docker inspect|查看容器详细信息|[docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/)|


### 1.1 <font color="#ffc000">案例</font>
通过 `Docker` 部署 `Nginx`
```PowerShell
# 第1步，去DockerHub查看nginx镜像仓库及相关信息

# 第2步，拉取Nginx镜像
docker pull nginx

# 第3步，查看镜像
docker images
# 结果如下：
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    605c77e624dd   16 months ago   141MB
mysql        latest    3218b38490ce   17 months ago   516MB

# 第4步，创建并允许Nginx容器
docker run -d --name nginx -p 80:80 nginx

# 第5步，查看运行中容器
docker ps
# 也可以加格式化方式访问，格式会更加清晰
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"

# 第6步，访问网页，地址：http://虚拟机地址

# 第7步，停止容器
docker stop nginx

# 第8步，查看所有容器
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"

# 第9步，再次启动nginx容器
docker start nginx

# 第10步，再次查看容器
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}\t{{.Names}}"

# 第11步，查看容器详细信息
docker inspect nginx

# 第12步，进入容器,查看容器内目录
docker exec -it nginx bash
# 或者，可以进入MySQL
docker exec -it mysql mysql -uroot -p

# 第13步，删除容器
docker rm nginx
# 发现无法删除，因为容器运行中，强制删除容器
docker rm -f nginx
```

---

### 1.2 <font color="#ffc000">命令别名</font>
在 **Linux** 中，可以给命令起别名的方式，来简化命令的书写，提高工作效率

**<font color="#ffc000">实现</font>**：
> 1. 修改 *<font color="#ffc000">/root/.bashrc</font>* 文件
> 
> ```PowerShell
> vi /root/.bashrc
> ```
> ---
> 2. <font color="#ffc000">可以看到以下内容</font>：
> 
> ```PowerShell
> # .bashrc
> 
> # User specific aliases and functions
> 
> alias rm='rm -i'
> alias cp='cp -i'
> alias mv='mv -i'
> 
> # Source global definitions
> if [ -f /etc/bashrc ]; then
>         . /etc/bashrc
> fi
> ```
> ---
> 3. <font color="#ffc000">添加一个别名，实现 `docker images`</font>
> - **<font color="#ffc000">格式</font>**：`alias 别名='命令'`
> ```PowerShell
> # 创建一个别名，执行 `docker images` 时只需输入 `dis`
> alias dis='docker images'
> ```
> ---
> 4. <font color="#ffc000">保存并生效文件</font>
> 
> ```PowerShell
> source /root/.bashrc
> ```

---
---

## 2 <font color="#ffc000">数据卷</font>
**<font color="#ffc000">引言</font>**：

*当我们在 `Docker` 容器内安装了 `Nginx` 之后，如果想要修改 `Nginx` 其中的静态资源，我们需要使用命令 `docker exec -it nginx bash` 进入 `Nginx` 容器，再找到对应路径下的静态资源，例如 `/usr/share/nginx/html/index.html` ，此时使用 `vi` 命令来修改静态资源时，我们会发现，系统提示找不到该命令。*

<font color="#ffc000">*这是因为 `Docker` 容器在运行时，只具备了运行该容器最基本的系统函数和依赖*</font>

为了方便容器间的数据持久化和共享，`Docker` 提供了 **数据卷**（Volumes）机制，允许我们将容器内的数据与宿主机进行绑定或映射

### 2.1 <font color="#ffc000">什么是数据卷</font>

**数据卷（volume）** 是一个虚拟目录，是容器内目录与宿主机目录之间映射的桥梁

![[Docker 数据卷挂载流程图.png]]

以引言为例，
> 
> 1. 创建了两个数据卷：`conf`、`html`
> 2. `Nginx` 容器内部的 `conf` 目录和 `html` 目录分别与两个数据卷关联
> 3. 数据卷 `conf` 和 `html` 分别指向了宿主机的 `/var/lib/docker/volumes/conf/_data` 目录和 `/var/lib/docker/volumes/html/_data` 目录
> 4. 此时，`Nginx` 容器内的 `conf` 和 `html` 目录就与宿主机的 `conf`和`html` 目录关联起来，称为 **<font color="#ffc000">挂载</font>**
> 5. 这样一来，操作宿主机的 `/var/lib/docker/volumes/html/_data` 就是在操作 `Nginx` 容器内的 `/usr/share/nginx/html`

<font color="#ffc000">⚠️</font> `/var/lib/docker/volumes` 这个目录就是默认的存放所有容器数据卷的目录，其下再根据数据卷名称创建新目录，格式为 `/数据卷名/_data`


> [!question] 
> Q：为什么不让容器目录直接指向宿主机目录呢？
> 
> A：
> 1. 因为直接指向宿主机目录就与宿主机强耦合了，如果切换了环境，宿主机目录就可能发生改变了。由于容器一旦创建，目录挂载就无法修改，这样容器就无法正常工作了
> 2. 但是容器指向数据卷，一个逻辑名称，而数据卷再指向宿主机目录，就不存在强耦合。如果宿主机目录发生改变，只要改变数据卷与宿主机目录之间的映射关系即可。

---

### 2.2 <font color="#ffc000">数据卷命令</font>

|**命令**|**说明**|**文档地址**|
|---|---|---|
|docker volume create|创建数据卷|[docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/)|
|docker volume ls|查看所有数据卷|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_ls/)|
|docker volume rm|删除指定数据卷|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_prune/)|
|docker volume inspect|查看某个数据卷的详情|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_inspect/)|
|docker volume prune|清除数据卷|[docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/)|

- 容器与数据卷的挂载要在创建容器时配置，对于创建好的容器，是不能设置数据卷的
- <font color="#ffc000">格式</font>：`docker run -d --name 容器自定义名 -p 宿主机端口:容器内端口 -v 数据卷名:容器内目录 镜像名`
---

### 2.3 <font color="#ffc000">引言实现</font>

```PowerShell
# 1.首先创建容器并指定数据卷，注意通过 -v 参数来指定数据卷
docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx

# 2.然后查看数据卷
docker volume ls
# 结果
DRIVER    VOLUME NAME
local     29524ff09715d3688eae3f99803a2796558dbd00ca584a25a4bbc193ca82459f
local     html

# 3.查看数据卷详情
docker volume inspect html
# 结果
[
    {
        "CreatedAt": "2024-05-17T19:57:08+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": null,
        "Scope": "local"
    }
]

# 4.查看/var/lib/docker/volumes/html/_data目录
ll /var/lib/docker/volumes/html/_data
# 可以看到与nginx的html目录内容一样，结果如下：
总用量 8
-rw-r--r--. 1 root root 497 12月 28 2021 50x.html
-rw-r--r--. 1 root root 615 12月 28 2021 index.html

# 5.进入该目录，并随意修改index.html内容
cd /var/lib/docker/volumes/html/_data
vi index.html

# 6.打开页面，查看效果

# 7.进入容器内部，查看/usr/share/nginx/html目录内的文件是否变化
docker exec -it nginx bash
```

---
---

## 3 <font color="#ffc000">匿名数据卷</font>

### 3.1 <font color="#ffc000">什么是匿名数据卷</font>

我们可以使用 `docker inspect mysql` 查看 `mysql` 容器的信息：

```JSON
{
  "Config": {
    // ... 略
    "Volumes": {
      "/var/lib/mysql": {}
    }
    // ... 略
  }
}
```

```JSON
        "Mounts": [
            {
                "Type": "volume",
                "Name": "5169ef04175b1a069612f820adf6df520c6dc2f166e0e2ef5fdc7e97b9b72adb",
                "Source": "/var/lib/docker/volumes/5169ef04175b1a069612f820adf6df520c6dc2f166e0e2ef5fdc7e97b9b72adb/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```
可以看到这个容器声明了一个本地目录，需要挂载数据卷，但是数据卷未定义，这就是 <font color="#ffc000">匿名数据卷</font> 


---
### 3.2 <font color="#ffc000">为什么会有匿名数据卷</font>
在容器信息中有几个关键属性：

- `Name`：数据卷名称。由于定义容器未设置容器名，这里的就是匿名卷自动生成的名字，一串hash值
- `Source`：宿主机目录
- `Destination`：容器内的目录

<font color="#ffc000">⚠️</font> 与一般数据卷一样，上述信息的意思是：

将 `mysql` 容器内的 `/var/lib/mysql` 目录，与数据卷 `5169ef04175b1a069612f820adf6df520c6dc2f166e0e2ef5fdc7e97b9b72adb` 进行挂载，因此在宿主机就可以访问 `mysql` 容器的 `var/lib/mysql` 目录

```Bash
ls -l /var/lib/docker/volumes/29524ff09715d3688eae3f99803a2796558dbd00ca584a25a4bbc193ca82459f/_data
```

> [!question] 
> Q：为什么在创建 `mysql` 容器时没有指定创建数据卷，但`mysql` 容器却自动挂载一个匿名数据卷呢？
> 
> A：在创建容器时，Docker 会自动为需要持久化存储的目录创建一个数据卷
以确保容器重启时，MySQL 数据不会丢失，这样即使容器删除，匿名数据卷依然存在，容器可以重新创建并继续指定该数据卷中的数据

---
---
## 4 <font color="#ffc000">本地目录挂载</font>

<font color="#ffc000">引言</font>：

*对于 `mysql` 容器自动创建的匿名数据卷，当我们需要对 `mysql` 容器进行升级时，需要卸载旧的容器重新生成，此时也会重新生成一个新的匿名数据卷，这样会造成新的容器可能无法使用旧的匿名数据卷*

*其中一种方式是将匿名数据卷的数据拷贝到新的容器当中，但匿名数据卷的名字太长，十分不方便，可以使用 <font color="#ffc000">命名数据卷</font> *


除此之外，我们可以将<font color="#ffc000">容器目录直接挂载到宿主机的指定目录下</font>，而无需使用数据卷的形式

### 4.1 <font color="#ffc000">格式</font>

```PowerShell
-v /宿主机路径:/容器路径
```
- 本地目录或文件必须以 `/`  或 `./` 开头，如果直接以名字开头，会被识别为数据卷名而非本地目录名
---
### 4.2 <font color="#ffc000">案例</font>

基于宿主机本地目录实现 `MySQL` 数据目录、配置文件、初始化脚本的挂载

<font color="#ffc000">实现</font>：


```Bash
# 1.删除原来的MySQL容器
docker rm -f mysql

# 2.进入root目录
cd ~

# 3.创建并运行新mysql容器，挂载本地目录
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  -v root/mysql/data:/var/lib/mysql \
  -v root/mysql/conf:/etc/mysql/conf.d \
  -v root/mysql/init:/docker-entrypoint-initdb.d \
  mysql

# 4.查看root目录，可以发现~/mysql/data目录已经自动创建好了
ls -l mysql
# 结果：
总用量 4
drwxr-xr-x. 2 root    root   20 5月  19 15:11 conf
drwxr-xr-x. 7 polkitd root 4096 5月  19 15:11 data
drwxr-xr-x. 2 root    root   23 5月  19 15:11 init

# 查看data目录，会发现里面有大量数据库数据，说明数据库完成了初始化
ls -l data

# 5.查看MySQL容器内数据
# 5.1.进入MySQL
docker exec -it mysql mysql -uroot -p123
# 5.2.查看编码表
show variables like "%char%";
# 5.3.结果，发现编码是utf8mb4没有问题
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+

# 6.查看数据
# 6.1.查看数据库
show databases;
# 结果，hmall是黑马商城数据库
+--------------------+
| Database           |
+--------------------+
| hmall              |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
# 6.2.切换到hmall数据库
use hmall;
# 6.3.查看表
show tables;
# 结果：
+-----------------+
| Tables_in_hmall |
+-----------------+
| address         |
| cart            |
| item            |
| order           |
| order_detail    |
| order_logistics |
| pay_order       |
| user            |
+-----------------+
# 6.4.查看address表数据
+----+---------+----------+--------+----------+-------------+---------------+-----------+------------+-------+
| id | user_id | province | city   | town     | mobile      | street        | contact   | is_default | notes |
+----+---------+----------+--------+----------+-------------+---------------+-----------+------------+-------+
| 59 |       1 | 北京     | 北京   | 朝阳区    | 13900112222 | 金燕龙办公楼   | 李佳诚    | 0          | NULL  |
| 60 |       1 | 北京     | 北京   | 朝阳区    | 13700221122 | 修正大厦       | 李佳红    | 0          | NULL  |
| 61 |       1 | 上海     | 上海   | 浦东新区  | 13301212233 | 航头镇航头路   | 李佳星    | 1          | NULL  |
| 63 |       1 | 广东     | 佛山   | 永春      | 13301212233 | 永春武馆       | 李晓龙    | 0          | NULL  |
+----+---------+----------+--------+----------+-------------+---------------+-----------+------------+-------+
4 rows in set (0.00 sec)
```

---
---
## 5 <font color="#ffc000">构建镜像</font>

### 5.1 <font color="#ffc000">镜像结构</font>

自定义镜像本质：依次准备好程序运行的基础环境、依赖、应用本身、运行配置等文件，并且打包而成

<font color="#ffc000">例如</font>：

> 从零部署一个Java应用，流程为
> - 准备一个linux服务（CentOS）
> - 安装并配置JDK
> - 上传Jar包
> - 运行jar包
> 
> 因此，打包镜像也分为
> - 准备Linux运行环境（基础运行环境）
> - 安装并配置JDK
> - 拷贝jar包
> - 配置启动脚本

上述步骤中的每一次操作其实都是在生产一些文件（系统运行环境、函数库、配置最终都是磁盘文件），所以 <font color="#ffc000">镜像就是一堆文件的集合</font>

并按照操作的步骤分层叠加而成，每一层形成的文件都会单独打包并标记一个唯一id，称为 **<font color="#ffc000">Layer</font>**（**<font color="#ffc000">层</font>**）

![[Docker 镜像结构图.png]]

---

### 5.2 <font color="#ffc000">Dockerfile</font>

<font color="#ffc000">引言</font>：

*制作镜像的过程中，需要逐层处理和打包，比较复杂，所以 `Docker` 就提供了自动打包镜像的功能。我们只需要将打包的过程，每一层要做的事情用固定的语法写下来，交给 `Docker` 去执行*

记录镜像结构的文件就称为 **`Dockerfile`** 

<font color="#ffc000">语法</font>：

|**指令**|**说明**|**示例**|
|---|---|---|
|**FROM**|指定基础镜像|`FROM centos:6`|
|**ENV**|设置环境变量，可在后面指令使用|`ENV key value`|
|**COPY**|拷贝本地文件到镜像的指定目录|`COPY ./xx.jar /tmp/app.jar`|
|**RUN**|执行Linux的shell命令，一般是安装过程的命令|`RUN yum install gcc`|
|**EXPOSE**|指定容器运行时监听的端口，是给镜像使用者看的|EXPOSE 8080|
|**ENTRYPOINT**|镜像中应用的启动命令，容器运行时调用|ENTRYPOINT java -jar xx.jar|

<font color="#ffc000">案例1 </font> <u>基于 Ubuntu 镜像来构建一个 Java 应用的 Dockerfile 文件</u>

> ```Dockerfile
> # 指定基础镜像
> FROM ubuntu:16.04
> # 配置环境变量，JDK的安装目录、容器内时区
> ENV JAVA_DIR=/usr/local
> ENV TZ=Asia/Shanghai
> # 拷贝jdk和java项目的包
> COPY ./jdk8.tar.gz $JAVA_DIR/
> COPY ./docker-demo.jar /tmp/app.jar
> # 设定时区
> RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
> # 安装JDK
> RUN cd $JAVA_DIR \
>  && tar -xf ./jdk8.tar.gz \
>  && mv ./jdk1.8.0_144 ./java8
> # 配置环境变量
> ENV JAVA_HOME=$JAVA_DIR/java8
> ENV PATH=$PATH:$JAVA_HOME/bin
> # 指定项目监听的端口
> EXPOSE 8080
> # 入口，java项目的启动命令
> ENTRYPOINT ["java", "-jar", "/app.jar"]
> ```
> 
> <font color="#ffc000">简化</font>：
> - 已经有人制作了基础的系统加 JDK 环境，我们在此基础上制作 java 镜像，就可以省去JDK的配置
> ```Dockerfile
> # 基础镜像
> FROM openjdk:11.0-jre-buster
> # 设定时区
> ENV TZ=Asia/Shanghai
> RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
> # 拷贝jar包
> COPY docker-demo.jar /app.jar
> # 入口
> ENTRYPOINT ["java", "-jar", "/app.jar"]
> ```

---
<font color="#ffc000">案例2</font> <u>基于案例1的 Dockerfile 文件，构建 java 应用镜像</u>

前置条件：`docker-demo.jar` 与 `Dockerfile` 都在 `/root/demo` 下

```Bash
# 指定 Dockerfile 目录构建镜像
docker build -t docker-demo /root/demo
```

- `docker build`：构建一个 `docker` 镜像
- `-t docker-demo` ：`-t` 参数是指定镜像的名称（`repository`和`tag`）
- `/root/demo`：指定 `Dockerfile` 所在目录