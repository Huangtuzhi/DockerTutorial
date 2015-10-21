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

#使用 Docker 构建并测试 Web 程序

```
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"

RUN adduser web --home /home/web --shell /bin/bash --disabled-password --gecos ""

RUN apt-get update --fix-missing
RUN apt-get -y install ruby ruby-dev build-essential redis-tools
RUN gem install --no-rdoc --no-ri sinatra json redis

RUN mkdir -p /var/www/webapp
RUN chmod -R 777 /var/www/webapp
RUN chown -R web:web /var/www/webapp

USER web

EXPOSE 4567

CMD [ "/var/www/webapp/bin/webapp" ]
```

启动一个新的容器
`sudo docker run -p 4567 -d --name titusapp9 -v $PWD/webapp:/var/www/webapp titushuang/sinatra`

查看容器映射到宿主机的端口

`sudo docker port titusapp9 4567`

> 0.0.0.0:49154

访问容器中的webapp
```
huangyi@HP ~/Practice/Docker/sinatra $ curl -i -H 'Accept: application/json' \
-d 'name=titushuang&status=bar' http://localhost:49154/json
HTTP/1.1 200 OK 
Content-Type: text/html;charset=utf-8
Content-Length: 36
X-Xss-Protection: 1; mode=block
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Server: WEBrick/1.3.1 (Ruby/1.9.3/2013-11-22)
Date: Wed, 21 Oct 2015 13:48:41 GMT
Connection: Keep-Alive

{"name":"titushuang","status":"bar"}
```

curl参数含义

```
Usage: curl [options...] <url>
 -i, --include       Include protocol headers in the output (H/F)
 -H, --header LINE   Custom header to pass to server (H)
 这两个参数将指定的HTTP头传给webapp
 -d, --data DATA     HTTP POST data (H)
```

相当于在浏览器中输入，只不过curl调用的是post方法，而浏览器调用get方法。

`http://localhost:49154/json?name=titushuang&status=bar`

返回数据由CGI构造

```
require "rubygems"
require "sinatra"
require "json"

class App < Sinatra::Application

  set :bind, '0.0.0.0'
  get '/' do
    "<h1>DockerBook Test Sinatra app</h1>"
  end

  post '/json/?' do
    params.to_json
  end

end
```
即echo请求数据。

#构建 Redis 镜像和容器

Dockerfile
```
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"

RUN apt-get update
RUN apt-get -y install redis-server redis-tools

EXPOSE 6379
ENTRYPOINT ["/usr/bin/redis-server"]
CMD []
```
ENTRYPOINT和CMD作用类似，相当于容器的自启动程序。

构建镜像

`sudo docker build -t titushuang/redis .`

从镜像构建容器

`sudo docker run -d -p 6379 --name redis titushuang/redis`

查看Redis的端口映射到宿主机哪个端口

```
huangyi@HP ~/Practice/Docker/redis $ sudo docker port redis 6379
0.0.0.0:49155
```

在宿主机上运行 redis 客户端连接到容器中的 redis 服务器端。

`redis-cli -h 127.0.0.1 -p 49155`

#让 Docker 容器互联
