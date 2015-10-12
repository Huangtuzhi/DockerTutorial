## Docker镜像
* 由文件系统叠加而成
* 最底端第一层是引导文件系统bootfs，类似grub
* 镜像第二层是root文件系统rootfs

列出镜像

```
huangyi@HP ~ % sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              cdd474520b8c        2 days ago          188 MB
```

删除镜像

`sudo docker rmi ubuntu`

## 镜像与容器关系
一对多

镜像 ——> 程序

容器 ——> 进程

## 仓库
查看ubuntu仓库中其他镜像

```
HP docker # sudo docker pull ubuntu
Pulling repository ubuntu
c6a3582257ff: Pulling image (vivid-20150528) from ubuntu, endpoint: https://registry-1.docker.io/v1/ 
5ba9dab47459: Pulling image (14.04.1) from ubuntu, endpoint: https://registry-1.docker.io/v1/ 
```

## commit 构建新镜像

创建一个要进行修改的定制容器

`sudo docker run -i -t ubuntu /bin/bash`

在容器中安装vim

```
root@93a46591d393:/# sudo apt-get install vim
Reading package lists... Done
```

提交
```
HP huangyi # sudo docker commit 93a46591d393 ubuntu/myvim
3806f1faa5f007ccc756c96490d23c75fb8ede77775c3cd2b310617038157876
```

查看本机现在的Repo，可以看见多了一个`ubuntu/myvim`

```
HP huangyi # sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu/myvim        latest              3806f1faa5f0        2 minutes ago       231.4 MB
ubuntu              latest              cdd474520b8c        3 days ago          188 MB
```

##基于 Dockerfile 构建新镜像

```
HP Docker # tree
.
└── static_web
    └── Dockerfile
```

Dockerfile文件
```
# Version: 0.01
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hi, I am in your container' \
        > /usr/share/nginx/html/index.html
EXPOSE 80
```

构建镜像

`sudo docker build -t="titushuang/static_web" .`

参看Repo，可以看见多了一个`titushuang/static_web`
```
huangyi@HP ~ $ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
titushuang/static_web    latest              1f8ee6fd2bd6        5 minutes ago       227.7 MB
ubuntu/myvim        latest              3806f1faa5f0        29 minutes ago      231.4 MB
ubuntu              latest              cdd474520b8c        3 days ago          188 MB
```

上述每一条RUN语句都会生成新镜像，更新镜像ID，删除旧的镜像。

查看最终镜像

```
huangyi@HP ~ $ sudo docker run -t -i 1f8ee6fd2bd6 /bin/bash
root@778aa645f23f:/# cat /usr/share/nginx/html/index.html
Hi, I am in your containe
```

可见镜像构建成功。

也可以启动容器中的Nginx。

```
sudo docker run -i -t -p 80 titus/static_web
nginx -g "daemon off";
```

-p选项将宿主机的49153端口绑定到容器的80端口，在宿主机中

```
HP static_web # curl 192.168.1.154:49153
Hi, I am in your container
```

##镜像推送到 Docker Hub
登陆 Docker Hub

`sudo docker login`

推送

`docker push titushuang/web`

需要特别注意的是构建时的username一定是Docker Hub中的用户名，

`sudo docker build -t="titushuang/web" .`

在[Docker Hub](https://hub.docker.com/r/titushuang/web/)中可以看见推过去的repo。

##参考

[http://stackoverflow.com/questions/25388684/pushing-docker-image-to-dockerhub](http://stackoverflow.com/questions/25388684/pushing-docker-image-to-dockerhub)

