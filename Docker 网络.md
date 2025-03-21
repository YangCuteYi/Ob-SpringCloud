
## 1 `Docker` <font color="#ffc000">初始网络分配</font>

在查看 `Docker` 创建的容器时，我们发现容器内部也有属于自己的 `IP` 地址

```PowerShell
# 查看 mysql 容器
docker inspect mysql
```

![[Docker mysql容器IP.png]]

同理，查看另外一个容器，`docker inspect nginx`

![[Docker nginx容器IP.png]]

> [!question]
> Q：我们可以发现两个容器都有一个相同的 <font color="#ffc000">网关</font>（`172.17.0.1`），且处在同一个 <font color="#ffc000">网段</font>（`172.17.0.x`），那么为什么会这样呢？
> 
> A：相同的网关是 `Docker` 通过桥接模式创建的虚拟网卡，默认名为 `docker0` ，并通过网桥方式，为 `Docker` 创建的容器分配 IP 
> 
> ![[Docker 宿主机与容器IP关系.png]]
> 
> <font color="#ffc000">⚠️</font> 因此容器之间虽然相互独立，但通过网桥建立了相互连接，实现相互访问


---
---
## 2 <font color="#ffc000">为什么使用自定义网络</font>

容器内的 IP 是一个虚拟 IP ，如果 `Docker` 停止了某个容器，这个容器的 IP 就有可能被其他容器所占用，从而发生变化，不利于使用。

我们可以使用 `Docker` 自定义网络功能来解决

<font color="#ffc000">在同一个自定义网络中的容器，可以通过别名互相访问</font>

---
---

## 3 <font color="#ffc000">常用命令</font>

|   |   |   |
|---|---|---|
|docker network create|创建一个网络|[docker network create](https://docs.docker.com/engine/reference/commandline/network_create/)|
|docker network ls|查看所有网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_ls/)|
|docker network rm|删除指定网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_rm/)|
|docker network prune|清除未使用的网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_prune/)|
|docker network connect|使指定容器连接加入某网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_connect/)|
|docker network disconnect|使指定容器连接离开某网络|[docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/)|
|docker network inspect|查看网络详细信息|[docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/)|

---
---

## 4 <font color="#ffc000">案例</font>
将容器 `mysql` 与 `dd` 加入自定义网络，并设置别名，互相访问

```Bash
# 1.首先通过命令创建一个网络
docker network create test

# 2.然后查看网络
docker network ls
# 结果：
NETWORK ID     NAME      DRIVER    SCOPE
639bc44d0a87   bridge    bridge    local
403f16ec62a2   test     bridge    local
0dc0f72a0fbb   host      host      local
cd8d3e8df47b   none      null      local
# 其中，除了test以外，其它都是默认的网络

# 3.让dd和mysql都加入该网络，注意，在加入网络时可以通过--alias给容器起别名
# 这样该网络内的其它容器可以用别名互相访问！
# 3.1.mysql容器，指定别名为db，另外每一个容器都有一个别名是容器名
docker network connect test mysql --alias db
# 3.2.db容器，也就是我们的java项目
docker network connect test dd

# 4.进入dd容器，尝试利用别名访问db
# 4.1.进入容器
docker exec -it dd bash
# 4.2.用db别名访问
ping db
# 结果
PING db (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=1 ttl=64 time=0.070 ms
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=2 ttl=64 time=0.056 ms
# 4.3.用容器名访问
ping mysql
# 结果：
PING mysql (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=2 ttl=64 time=0.054 ms
```

- 在自定义网络中，可以给容器起多个别名，默认的别名是容器名本身
- 在同一个自定义网络中的容器，可以通过别名互相访问
- <font color="#ffc000">⚠️</font> 也可以在运行容器时指定：
```bash
docker run -d --name 容器自定义名 -p 宿主机端口:容器内端口 --network 自定义网络名 -v 数据卷名:容器内目录 镜像名
```