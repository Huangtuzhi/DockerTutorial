Docker 生态系统中有三种 API

* Registry API：提供与存储 Docker 镜像的 Docker Registry 集成的功能
* Docker Hub API：提供与 Docker Hub 集成的功能 
* Docer Remote API：提供与 Docker 守护进程进行集成的功能

这三种 API 都是 [RESTful](http://www.ruanyifeng.com/blog/2011/09/restful) 风格的，需要着重了解的是 Remote API，主要是用来和 Docker 守护进行进行通信,进而可以远程控制镜像或者容器，就像在本机执行命令一样。

## 配置 Remote API 功能

默认情况下，Docker 进程会绑定到所在宿主机的套接字，位于 `unix:///var/run/docker.sock`。如果需要远程访问 Remote API，需要将 Docker 守护进程绑定到一个网络接口上去。

绑定到守护进程到 4243 端口

`sudo docker -d -H unix:///var/run/docker.sock -H 0.0.0.0:4243`

可以在其他机器测试连接

`curl http://*.*.*.*:4243/info`

使用 python 的 Json 工具格式化一下返回数据

`curl http://*.*.*.*:4243/info | python -mjson.tool`

会打印和本宿主机 `sudo docker info` 一样的信息。

```
{
    "Containers": 44,
    "Debug": 0,
    "Driver": "aufs",
    "DriverStatus": [
        [
            "Root Dir",
            "/var/lib/docker/aufs"
        ],
        [
            "Dirs",
            "151"
        ]
    ],
    "ExecutionDriver": "native-0.2",
    "IPv4Forwarding": 1,
    "Images": 63,
    "IndexServerAddress": "https://index.docker.io/v1/",
    "InitPath": "/usr/lib/docker.io/dockerinit",
    "InitSha1": "6578c4a98eb5aaa1db564782fd990839ebca1b4d",
    "KernelVersion": "3.13.0-24-generic",
    "MemoryLimit": 1,
    "NEventsListener": 0,
    "NFd": 11,
    "NGoroutines": 10,
    "SwapLimit": 1
}
```

## 通过 API 来管理 Docker 镜像

获取 Docker 守护进程中所有镜像的列表

` curl http://*.*.*.*:4243/images/json | python -mjson.tool`

```
[
    {
        "Created": 1444616406,
        "Id": "1f8ee6fd2bd681727f0218f0be1e63a22ca28e1ef37a918c6f7f7d379cdb5a8d",
        "ParentId": "c90d37781e07f71a49d084451b990fef4830487352f8a4a8acc075d817640d7f",
        "RepoTags": [
            "titushuang/web:mytag"
        ],
        "Size": 0,
        "VirtualSize": 227737551
    },
    {
        "Created": 1444340548,
        "Id": "cdd474520b8c2c0418260fc9dcd999c84fe35bd3a36304aeb47517a290e5a5c4",
        "ParentId": "f03f3645bde1a45143056ad6bc9d352199f13cdb01c6d59a1c972432c9fc7e97",
        "RepoTags": [
            "ubuntu:latest"
        ],
        "Size": 0,
        "VirtualSize": 187958250
    }
]
```

在 Docker Hub 中查找镜像

`curl "http://*.*.*.*:4243/images/search?term=redis" | python -mjson.tool`

```
[
    {
        "description": "Redis - current stable",
        "is_official": false,
        "is_trusted": true,
        "name": "ruo91/redis",
        "star_count": 0
    },
    {
        "description": "Redis Atomic App",
        "is_official": false,
        "is_trusted": true,
        "name": "projectatomic/redis-centos7-atomicapp",
        "star_count": 0
    }
]
```

## 通过 API 来管理 Docker 容器

显示所有正在运行的容器

`curl -s "http://*.*.*.*:4243/containers/json" | python -mjson.tool`

```
[
    {
        "Command": "/usr/sbin/apache2 -D FOREGROUND",
        "Created": 1445696729,
        "Id": "bd7acf1a0fad1dd6f8491ef5c556153493aa913922074a00c4dabaa594b33164",
        "Image": "titushuang/apache:latest",
        "Names": [
            "/focused_torvalds"
        ],
        "Ports": [
            {
                "IP": "0.0.0.0",
                "PrivatePort": 80,
                "PublicPort": 49153,
                "Type": "tcp"
            }
        ],
        "Status": "Up 2 minutes"
    }
]
```

创建一个新容器

```
curl -X POST -H "Content-Type: application/json" \
"http://202.201.13.*:4243/containers/create?name=test_remote_api" \
-d '{
"Image":"titushuang/jekyll",
"Hostname":"jekyll" 
}'
```

启动一个容器

```
curl -X POST -H "Content-Type: application/json" \
"http://202.201.13.45:4243/containers/d23657590b09df5c1671cd723fe2ff5cf4d89378c2f0d91a55c4e83bf6d832a6\
start" \
-d '{
    "PublishAllPorts":true
}'
```

创建和启动容器加起来就提供了与 Docker run 相同的功能。

显示容器详细信息

```
curl http://202.201.13.*:4243/containers/ \
d23657590b09df5c1671cd723fe2ff5cf4d89378c2f0d91a55c4e83bf6d832a6/json | \
python -mjson.tool
```

