# 构建 Jekyll 站点
Jekyll 是基于 ruby 的博客生成工具，基于 Ubuntu 镜像进行构建。

安装 jekell 的 Dockerfile
```
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"
ENV REFRESHED_AT 2015-10-12

RUN apt-get -yqq update
RUN apt-get -yqq install ruby ruby-dev make nodejs
RUN gem install --no-rdoc --no-ri jekyll

VOLUME /data
VOLUME /var/www/html
WORKDIR /data

ENTRYPOINT [ "jekyll", "build", "--destination=/var/www/html" ]

```

安装 apache 的 Dockerfile
```
FROM ubuntu
MAINTAINER titushuang "ituzhi@163.com"
ENV REFRESHED_AT 2015-10-12

RUN apt-get -yqq update
RUN apt-get -yqq install apache2

VOLUME [ "/var/www/html" ]
WORKDIR /var/www/html

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2

RUN mkdir -p $APAHCE_RUN_DIR $APACHE_LOCK_DIR $APACHE_LOG_DIR

EXPOSE 80

ENTRYPOINT [ "/usr/sbin/apache2" ]
CMD ["-D", "FOREGROUND"]
```

构建 Jeyll 容器

`sudo docker run -v /home/huangyi/Practice/Docker/apache/james_blog:/data/ --name titusblog titushuang/jekyll`

构建 Apache 容器，共享使用 Jeyll 的卷

`sudo docker run -d -P --volumes-from titusblog titushuang/apache`

更新博客内容后使用下列命令重新编译 Jekyll

`sudo docker start titusblog`
