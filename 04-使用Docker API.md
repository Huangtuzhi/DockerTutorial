Docker 生态系统中有三种 API

* Registry API：提供与存储 Docker 镜像的 Docker Registry 集成的功能
* Docker Hub API：提供与 Docker Hub 集成的功能 
* Docer Remote API：提供与 Docker 守护进程进行集成的功能

需要着重了解的是 Remote API，主要是用来和 Docker 守护进行进行通信。

## 配置 Remote API 功能

默认情况下，Docker 进程会绑定到所在宿主机的套接字，位于 `unix:///var/run/docker.sock`。如果需要远程访问 Remote API，需要将 Docker 守护进程绑定到一个网络接口上去。

绑定到守护进程到 4243 端口

`sudo docker -d -H unix:///var/run/docker.sock -H 0.0.0.0:4243`

可以在其他机器测试连接

`curl http://127.0.0.1:4243/info`

会打印和本宿主机 `sudo docker info` 一样的信息。

```
{
    "Containers":44,
    "Debug":0,
    "Driver":"aufs",
    "DriverStatus":
    [["Root Dir","/var/lib/docker/aufs"],["Dirs","151"]],
    "ExecutionDriver":"native-0.2",
    "IPv4Forwarding":1,
    "Images":63,
    "IndexServerAddress":"https://index.docker.io/v1/",
    "InitPath":"/usr/lib/docker.io/dockerinit",
    "InitSha1":"6578c4a98eb5aaa1db564782fd990839ebca1b4d",
    "KernelVersion":"3.13.0-24-generic",
    "MemoryLimit":1,"NEventsListener":0,
    "NFd":11,
    "NGoroutines":10,"SwapLimit":1}
```