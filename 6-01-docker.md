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

4. 

#### 构建镜像

```shell
#会寻找当前目录的Dockerfile文件
docker image build -t 名字:版本 .
```

#### 推送镜像

```shell
#先登录
docker login

#推送
docker image push 用户名/镜像名:版本
```

#### 通过commit容器创建镜像

```shell
docker container commit 容器id 用户名/镜像名:版本号
```

