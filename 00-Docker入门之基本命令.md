## 安装 Docker

Ubuntu/Mint 下安装 Docker

`sudo apt-get install lxc-docker`

如果 Docker 启动报错：
> Error loading docker apparmor profile: exec: "/sbin/apparmor_parser"

则需要先安装依赖包 

`sudo apt-get install apparmor`

运行Docker镜像

`sudo docker run -i -t ubuntu /bin/bash`

`-i： 保证容器中STDIN开启`

`-t： 为容器分配一个伪tty终端`

## 基本命令

查看当前系统中容器的列表

`docker ps -a`

重新打开容器会话shell 

`sudo docker attach titus_docker_container`

exit退出容器shell后，容器会随之停止运行。若需重新运行容器，需要执行

`sudo docker start titus_docker_container`
`sudo docker attach titus_docker_container`

创建守护式容器，不打开shell会话，让进程打印hello world

`huangyi@HP ~ % sudo docker run --name daemon_dave -d ubuntu /bin/sh -c "while true;do echo hello world; sleep 1; done"`

查看daemon容器内部日志

`sudo docker logs daemon_dave`

查看daemon容器内部进程

`sudo docker top daemon_dave`

查看daemon容器更多信息

`sudo docker inspect daemon_dave`

查看daemon容器固定字段信息

`sudo docker inspect --format='{{ .NetworkSettings.IPAddress }}' daemon_dave`

docker工作目录位于/var/lib/docker/containers，目录树如下

```
├── 0f2daad3a2347c06cea22cebd704d6d147380b4d7a14ac459a1a94c617aaf09d
│   ├── 0f2daad3a2347c06cea22cebd704d6d147380b4d7a14ac459a1a94c617aaf09d-json.log
│   ├── config.json
│   ├── hostconfig.json
│   ├── hostname
│   ├── hosts
│   └── resolv.conf
├── 6c2f88d0899b8041213eee0216689a4f0cd64ed31b41ca2b7d948af209e7613a
│   ├── 6c2f88d0899b8041213eee0216689a4f0cd64ed31b41ca2b7d948af209e7613a-json.log
│   ├── config.json
│   ├── hostconfig.json
│   ├── hostname
│   ├── hosts
│   └── resolv.conf
```

在a6b*-json.log中可以查看daemon容器的日志

```
{"log":"hello world\n","stream":"stdout","time":"2015-10-11T11:39:26.038047014Z"}
{"log":"hello world\n","stream":"stdout","time":"2015-10-11T11:39:27.039658441Z"}
{"log":"hello world\n","stream":"stdout","time":"2015-10-11T11:39:28.041660087Z"}
```

##参考

[http://www.open-open.com/lib/view/open1423703640748.html](http://www.open-open.com/lib/view/open1423703640748.html)

[第一本Docker书](http://www.amazon.cn/%E7%AC%AC%E4%B8%80%E6%9C%ACDocker%E4%B9%A6-%E8%A9%B9%E5%A7%86%E6%96%AF%C2%B7%E7%89%B9%E6%81%A9%E5%B8%83%E5%B0%94/dp/B00RBEIFMI/ref=sr_1_1_twi_pap_2?s=books&ie=UTF8&qid=1444566016&sr=1-1&keywords=%E7%AC%AC%E4%B8%80%E6%9C%ACdocker%E4%B9%A6)