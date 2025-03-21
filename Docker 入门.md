## 1 <font color="#ffc000">简述</font>

**Docker** 是一个开源的应用容器引擎，能够帮助开发者将应用及其所有依赖（环境）打包成一个可移植的容器。这个容器可以在任何流行的 Linux 机器上运行，也支持虚拟化功能

---
---
## 2 <font color="#ffc000">容器 container</font>
**<font color="#ffc000">引言</font>**：

*在安装了 `Docker` 之后，如果我们想要安装 `MySQL` ，只需要输入 `Docker` 特定的命令就能自动安装了*

*要知道，**不同操作系统下其安装包、运行环境是都不相同的**！如果是**手动安装，必须手动解决安装包不同、环境不同的、配置不同的问题**！*

*而使用Docker，这些完全不用考虑。就是因为Docker会自动搜索并下载MySQL。注意：这里下载的不是安装包，而是**镜像。**镜像中不仅包含了MySQL本身，还包含了其运行所需要的环境、配置、系统级函数库。因此它在运行时就有自己独立的环境，就可以跨系统运行，也不需要手动再次配置环境了。这套独立运行的隔离环境我们称为 <font color="#ffc000">容器</font> *

容器是一个虚拟化的独立环境，其中包含了 <font color="#ffc000">应用程序和它的依赖包</font>。每个容器运行时都是完全隔离的，彼此之间没有任何接口，确保了相互独立，避免了相互干扰

**独立性**：每个容器都具有自己的文件系统、网络和进程空间
**可移植性**：开发者只需构建一次容器，便可在任何支持 Docker 的平台上运行，确保应用在不同环境中的一致性


---
---

## 3 <font color="#ffc000">镜像 image</font>

`Docker` 安装软件的过程，就是自动搜索下载镜像，然后创建并运行容器的过程

`Docker` 会根据命令中的镜像名称自动搜索并下载镜像，`Docker` 官方提供了一个专门管理、存储镜像的网站，也称之 **镜像仓库（DockerRegistry）** ，并对外开放了镜像上传、下载的权利

DockerRegistry：https://hub.docker.com/

---
---

## 4 <font color="#ffc000">入门案例</font>
### 4.1 <font color="#ffc000">通过 `Docker` 安装 `MySQL`</font>

```PowerShell
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  mysql
```

---

### 4.2 <font color="#ffc000">命令解读</font>

- <span style="background:#fff88f">`docker run -d`</span> ：创建并运行一个容器，`-d` 则是让容器以后台进程运行
- <span style="background:#fff88f">`--name mysql`</span> : 给容器起个名字叫 `mysql` ，名字唯一
- <span style="background:#fff88f">`-p 3306:3306`</span>：设置端口映射
- <span style="background:#fff88f">`-e TZ=Asia/Shanghai   -e MYSQL_ROOT_PASSWORD=123`</span>：配置容器内进程运行时的一些参数，**<font color="#ffc000">格式</font>**：`-e KEY=VALUE`，`KEY` 和 `VALUE` 都由容器内进程决定
- <span style="background:#fff88f">`mysql`</span>：设置 **镜像** 名称，`Docker` 会根据这个名字搜索并下载镜像，<font color="#ffc000">格式</font>：    `REPOSITORY:TAG`，例如`mysql:8.0`

> [!question] 
> Q：为什么要设置端口映射？
> 
> A：容器是一个隔离环境，外界是不可访问的
> 1. 但是可以通过 **<font color="#ffc000">宿主机端口映射容器内的端口</font>**，当访问宿主机指定端口时，就是在访问容器内的端口了
> 2. 容器内端口往往是由容器内的进程决定，例如 ``MySQL`` 进程默认端口是3306，因此容器内端口一定是3306；而宿主机端口则可以任意指定，一般与容器内保持一致
> 3. <font color="#ffc000">格式</font>：`-p 宿主机端口:容器内端口`
