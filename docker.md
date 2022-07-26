```shell
#容器其实是进程，容器中的进程被限制了对CPU内存等资源的访问，进程停止，容器退出
```



```shell
#创建一个nginx容器并进入交互模式（shell用的是sh，此时如果输入exit，会退出shell，并停止容器）
docker run -it nginx sh

#在一个已经运行的容器执行一个额外的sh命令行（以sh shell进入容器39e，此时输入exit会推出shell，但不会停止容器运行）
docker exec -it 39e sh
```

 

```shell
#docker container run 背后发生了什么？
docker container run -d --publish 80:80 --name webhost nginx
```

- 在本地查找是否有nginx这个image镜像，但是没有发现
- 去远程的image registry查找nginx镜像（默认的registry是Docker Hub)
- 下载最新版本的nginx镜像 （nginx:latest 默认)
- 基于nginx镜像来创建一个新的容器，并且准备运行
- docker engine分配给这个容器一个虚拟IP地址
- 在宿主机上打开80端口并把容器的80端口转发到宿主机上
- 启动容器，运行指定的命令（这里是一个shell脚本去启动nginx）

