[详细笔记](https://dockertips.readthedocs.io/en/latest/)

#### 容器是啥？

```shell
#容器其实是进程，容器中的进程被限制了对CPU内存等资源的访问，进程停止，容器退出
```

#### 创建容器并进入容器

```shell
#创建一个nginx容器并进入交互模式（shell用的是sh，此时如果输入exit，会退出shell，并停止容器）
docker run -it nginx sh

#在一个已经运行的容器执行一个额外的sh命令行（以sh shell进入容器39e，此时输入exit会推出shell，但不会停止容器运行）
docker exec -it 39e sh
```

 

#### docker container run 背后发生了什么？

```shell
docker container run -d --publish 80:80 --name webhost nginx
#-d 在后台运行容器，并且打印容器id。
#加上--rm参数，那么容器会在退出后自动删除
```

- 在本地查找是否有nginx这个image镜像，但是没有发现
- 去远程的image registry查找nginx镜像（默认的registry是Docker Hub)
- 下载最新版本的nginx镜像 （nginx:latest 默认)
- 基于nginx镜像来创建一个新的容器，并且准备运行
- docker engine分配给这个容器一个虚拟IP地址
- 在宿主机上打开80端口并把容器的80端口转发到宿主机上
- 启动容器，运行指定的命令（这里是一个shell脚本去启动nginx）



#### 导入导出镜像

```shell
#导出镜像
docker image save nginx:1.20.0 -o nginx.image

#导入镜像
docker image load -i .\nginx.image
```



#### 清理镜像/容器

```shell
#清理没有启动的容器
docker system prune -f

#清理没有被使用的镜像
docker image prune -a
```



#### Dockerfile

- Dockerfile是用于构建docker镜像的文件
- Dockerfile里包含了构建镜像所需的“指令”
- Dockerfile有其特定的语法规则

##### RUN

​	一个run会多一层，建议多条语句用`&&`拼接

​	查看分层命令`docker image history 镜像名/id`

##### ADD和COPY

​	都是把文件/文件夹拷贝到镜像里，如果指定的路径不存在，会创建。区别是ADD遇到常见的压缩格式会解压并复制，比如`.tar.gz`格式。普通文件用copy，压缩文件用add。

##### WORKDIR

​	指定工作目录，相当于`cd`，如果目录不存在会创建。ADD/COPY的文件会拷贝到WORKDIR下

##### ARG和ENV

​	都可以用来设置一个“变量”。区别ARG声明变量只存在build容器的过程中。而ENV声明的变量讲作为环境变量存在容器中。

##### CMD

​	可以用来设置容器启动时默认会执行的命令

​	如果docker container run 启动容器时指定了其他命令，则CMD命令会被忽略

​	如果定义了多个CMD，只有最后一个会被执行

​	`echo hello $NAME` 正确写法应该是 `CMD ["sh", "-c", "echo hello $NAME"]`而不是`CMD ["echo", "hello $NAME"]`

##### ENTRYPOINT

​	跟CMD一样，用来执行命令，区别是，ENTRYPOINT所执行的命令无法被覆盖，容器启动后默认执行。

​	常常与`CMD []`空的CMD配合使用，ENTRYPOINT声明要执行的命令，空CMD可以在RUN时表示传递的参数。

​	**CMD和ENTRYPOINT同时支持shell格式和exec格式**

##### 技巧

1. 编写Dockerfile时，尽量把下载、不引起变化等语句放在前面，copy等经常变化的命令放在后面，这样可以充分利用缓存。因为当某条命令比如copy，拷贝的文件发生了变化，那么copy及其后的所有语句不会走缓存。

2. 通过.dockerignore忽略多余文件，保护隐私文件

3. 两阶段式构建，减少镜像大小，比如下面一个go demo

   ```dockerfile
   FROM golang:1.17.13-alpine3.16 AS builder
   
   WORKDIR /data/gin_blog/
   COPY . /data/gin_blog
   
   ENV GO111MODULE=on
   ENV GOPROXY="https://goproxy.cn"
   
   RUN go mod download && \
       go build -o main
   
   
   FROM loads/alpine:3.8
   WORKDIR /data/gin_blog/
   COPY --from=builder /data/gin_blog /data/gin_blog
   
   EXPOSE 8000
   ENTRYPOINT ["./main", "-h", "0.0.0.0"]
   ```

4. 尽量使用非root账户运行容器，比如

   ```dockerfile
   FROM golang:1.17.13-alpine3.16 AS builder
   
   WORKDIR /data/gin_blog/
   COPY . /data/gin_blog
   
   ENV GO111MODULE=on
   ENV GOPROXY="https://goproxy.cn"
   
   RUN go mod download && \
       go build -o main
   
   
   FROM loads/alpine:3.8
   WORKDIR /data/gin_blog/
   #alpine是用以下的方式创建用户组以及用户
   RUN addgroup -S www && adduser -S www && \
       chown www:www /data/gin_blog
   
   #通常linux是以下面这种方式创建用户组
   #RUN groupadd -r www && useradd -r -g www www && \
   #    chown -R www:www /data/gin_blog \
   
   #window无法指定用户运行，需要屏蔽下面这行
   USER www
   COPY --from=builder /data/gin_blog /data/gin_blog
   
   EXPOSE 8000
   ENTRYPOINT ["./main", "-h", "0.0.0.0"]
   ```

   此时用`docker container top 容器名/id`可以查看容器是以什么用户运行。

#### 构建镜像

```shell
#会寻找当前目录的Dockerfile文件
docker image build -t 名字:版本 .

#从当前名录指定Dockerfile文件
docker image build -t 名字:版本 -f 指定的Dockerfile文件名 .
```

#### 推送镜像

```shell
#先登录
docker login

#提交前可以打个tag
docker tag 容器id tag名(通常为 用户名/镜像名:版本号)
#推送
docker image push 用户名/镜像名:版本
```

#### 通过commit容器创建镜像

```shell
docker container commit 容器id 用户名/镜像名:版本号
```

#### 持久化

##### volume持久化

构建镜像时指定需要用volume持久化的文件夹

```dockerfile
#dockerfile文件
...
VOLUME ["/app"]		#该镜像构建出的容器里/app文件夹下的所有数据将被持久化到宿主机中/var/lib/docker/volumes/目录下
...
```

```dockerfile
#跑起容器时，要给volume起个名字，并指定好volume的路径，也就是上面的/app，这样便于管理和复用volume（不-v且dockerfile定义了VOLUME会持久化到默认路径且随机起文件名）
docker run -v test:/app 镜像名...		#这里test是指定的volume名   volume名:路径

#删除容器后，容器数据删除,指定的volume文件夹会保留

#重新跑起容器，并指定回原来的volume文件路径，容器会复用volume文件数据，实现持久化
docker run -v test:/app 镜像名...	
```

```dockerfile
#查看volume列表
docker volume ls

#查看volume文件保存在哪个路径，window中docker运行在虚拟环境路径不方便访问
docker volume inspect volume全名

#删除没有被使用的volume
docker volume prune
```

##### Bind Mount持久化

volume持久化方式在window使用不方便，这时可以用Bind Mount

```dockerfile
docker run -d -v 本地路径:容器路径 镜像名...
```

#### 使用远程主机做volume

- 本地和远程主机可以ssh通信
- 安装插件`vieux/sshfs`
- 剩下百度吧

#### 网络

##### 容器网络涉及哪些问题?

1. 容器为什么能获取IP地址？

2. 为什么宿主机可以ping通容器?

3. 为什么容器之间的ip是互通的?

   每个容器默认连接1个叫docker0的bridge，bridg会为每个容器划分ip，起到类似网关的作用，帮助连接上同个bridge的容器间通讯

4. 为什么容器能ping通外网？

   ```shell
   [root@localhost tang]# ip route
   default via 192.168.25.2 dev ens33 proto static metric 100 
   172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
   192.168.25.0/24 dev ens33 proto kernel scope link src 192.168.25.101 metric 100 
   192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 
   ```

   容器docker0对外访问时，会做网络地址转发(nat)，把172.17.0.0网段的地址转换为192.168.25.2出去

   net主要用于解决ip地址不够用的问题。能把一个私有地址转化为公有地址用于网络访问。

5. 容器的端口转发(-p)是怎么回事？

##### 自定义bridge

```dockerfile
#创建bridge
docker network create -d bridge 名字

#创建时可以指定参数
docker network create -d bridge \
--subnet=192.168.25.0/24 \			#子网掩码，会分配192.168.25.2开始的ip
--ip-range=192.168.25.0/24 \		#ip网段
--gateway=192.168.25.1 \			#网关，通常是路由器地址，后者就你的ipv4，最后一段为1
mynetwork							#bridge的名字

#查看创建的网桥（bridge）
docker network ls

#之后创建容器一个叫test的容器，指定连接这个名为mynetwork的bridge，他会为连上来的容器划分ip
docker run -d --network mynetwork --name test 镜像名
#可以查看test容器的Ip和连接到哪个bridge
docker container inspect test
#关注Networks字段

#test容器连接bridge网桥
docker network connect bridge test
docker network connect 要连接的网桥名 容器名
#网络名可以通过docker network ls 查看
#这样test容器就可以连接bridge网桥下的所有容器

#关闭连接
docker network disconnect bridge test
#断开test连接bridge网桥
```

- 自定义的网桥下的容器可以互相ping通，通过ip或者容器名

- 自定义的网桥提供类似DNS的功能，让我们可以ping通容器名，但默认的不行

  默认的网桥docker0下的容器不能通过容器名ping通，但通过ip还是可以的。

##### 端口号映射

- 容器构建时`-p`参数，比如`-p 8000:8000`，宿主机8000端口映射到容器内8000端口

- 即使dockerfile文件不声明`EXPOSE`构建镜像时依旧用`-p`实现与宿主机端口映射，配置`EXPOSE`主要是会出现在配置项，可以用`docker image inspect 镜像名`查看到配置项，提醒使用者使用端口转发。

- 还是nat做的转发

##### host网桥

```shell
C:\Users\Yeah>docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
73071c4828cd   bridge      bridge    local	#默认网桥
c083c6ffccaf   host        host      local	#host网桥
9ec36224ea7c   none        null      local	#用这个网桥创建出来的容器没有网络，外网访问不到
```

- host网桥就是我们宿主机的本地网络，如果构建容器时使用了host网络

```shell
docker run -d --name test --network host 镜像名
```

- 那么容器就是使用我们宿主机的网络，容器占据了80端口，宿主机80端口也会被占据
- 走bridge利用nat的转发会消耗一定性能，如果走host就不会经过转发，损耗性能。

#### docker-compose

一个python写的帮助配置docker的工具

##### 语法结构

```dockerfile
version: "3.8"	#指定版本，因为语法一直在更新

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
    image: # 镜像的名字
    command: # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
    environment: # 可选，相当于 docker run里的 --env
    volumes: # 可选，相当于docker run里的 -v
    networks: # 可选，相当于 docker run里的 --network
    ports: # 可选，相当于 docker run里的 -p
  servicename2:

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
```

以 Python Flask + Redis练习：为例子

```shell
docker image pull redis
docker image build -t flask-demo .

# create network
docker network create -d bridge demo-network

# create container
docker container run -d --name redis-server --network demo-network redis
docker container run -d --network demo-network --name flask-demo --env REDIS_HOST=redis-server -p 5000:5000 flask-demo
```

改造成一个docker-compose文件，如下

```shell
version: "3.8"

services:
  flask-demo:
    image: flask-demo:latest  #1.该镜像本地不存在会从docker仓库拉取，如果是自定义的镜像需要在本地提前构建
    #build:./flask			  #2.执行docker-compose build 会从.flask文件寻找dockerfile文件进行构建
    #build:					  #3.也可以通过这种方式指定build的文件	
      #context:./flask
      #dockerfile:Dockerfile.env
    environment:
      - REDIS_HOST=redis-server
    networks:
      - demo-network
    ports:
      - 8080:5000

  redis-server:
    image: redis:latest
    networks:
      - demo-network
      #- demo-network2  #可以指定多个网络

networks:
  demo-network:
  #demo-network2:	#可以指定多个网络
  #  ipam:
  #    driver:default	#可以给demo-network2指定driver，默认就是default
```

##### 命令集

```shell
#前台运行docker-compose文件定义的容器（在docker-compose.yml所在目录执行，也可以通过-f指定）
docker-compose up
#ctrl+c，退出运行

#后台运行docker-compose文件定义的容器
docker-compose up -d 

#查看后台通过docker-compose运行的容器情况
docker-compose ps

#停止
docker-compose stop

#清理后台停止的，通过docker-compose创建的容器
docker-compose rm 

#-p指定名字前缀，运行
docker-compose -p 名字 up -d

#拉取用到的镜像，这样up的时候会快些
docker-compose pull

#有时文件修改了，需要重新Build，跑起最新的镜像
docker-compose up -d --build

#重启已经存在的容器
docker-compose restart

#如果composer文件移除了一些配置，可以这样重启
docker-compose up -d --remove-orphans

#水平扩展
docker-compose up -d --scale 服务名，比如 flask-demo=3	#这样就会启动3个flask-demo
#如果此时在执行
docker-compose up -d --scale flask-demo=1  	#会关掉多余的flask-demo，只保留1个
#如果启动3个服务后
ping flask-demo		#这3个示例会轮流的返回，实现一个简单的负载均衡

#配置预览
docker-compose config
```

- [添加Nginx](https://dockertips.readthedocs.io/en/latest/_downloads/15ef8ef4c424aefda9ce24c71698051d/compose-scale-example-2.zip)
- [环境变量](https://dockertips.readthedocs.io/en/latest/_downloads/15ef8ef4c424aefda9ce24c71698051d/compose-scale-example-2.zip)
- [健康检查](https://dockertips.readthedocs.io/en/latest/_downloads/529c888c2faf46a0906548ed7510d12b/compose-healthcheck-redis.zip)

#### demo

[这里有很多demo](https://github.com/docker/awesome-compose)
