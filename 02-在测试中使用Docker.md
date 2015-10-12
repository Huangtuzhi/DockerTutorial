前面主要是了解 Docker 的基础知识和使用方法，接下来看看怎么在实际的开发和测试中使用 Docker 这一利器。

#使用 Docker 测试静态网站

## 初始 Dockerfile
新建目录 sample 和 nginx，将 nginx 配置放在 nginx 目录下面。
```
├── sample
│   ├── Dockerfile
│   └── nginx
│       ├── global.conf
│       └── nginx.conf

```

Dockerfile

```
# Version: 0.01
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"
ENV REFRESHED_AT 2015-10-12
RUN apt-get update
RUN apt-get -y -q install nginx
RUN mkdir -p /var/www/html
ADD nginx/global.conf /etc/nginx/conf.d/
ADD nginx/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

## 构建 Sample 网站和 Nginx 镜像

`sudo docker build -t titushuang/nginx .`

目录下新建文件夹 website 存放网页文件 index.html
```
.
├── Dockerfile
├── nginx
│   ├── global.conf
│   └── nginx.conf
└── website
    └── index.html
```

网页文件

```
<head>

<title>Test website</title>

</head>

<body>

<h1>This is a test website</h1>

</body>
```

创建容器，在sample目录下执行命令

`sudo docker run -d -p 80 --name web -v $PWD/website:/var/www/html/website titushuang/nginx nginx`

`-v` 将宿主机的 `$PWD/website` 目录作为卷挂载到了容器的 `/var/www/html/website`下。这样可以直接在宿主机中对网站进行修改。