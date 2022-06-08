# 概览

> docker 是一个容器引擎

###### 名词解释

- image
  - 镜像。是一个只读模板，用来创建容器

- container
  - 容器。是一个可运行的==镜像实例==

- Dockerfile
  - 镜像构建的模板。描述镜像构建的步骤

上述三者之间的关系，==通过 Dockerfile 构建出镜像，然后通过镜像创建容器，而程序就运行在容器中==，需要注意的是，一个镜像可以任意创建多个容器，且各个容器之间互相隔离

# 基础概念

### 容器-container

它特别像一个虚拟机，容器中运行着一个完整的操作系统，可以在容器中安装 node，运行 npm 脚本，可以做一切当前操作系统能做的事

### 镜像-image

镜像是一个文件，用来创建容器。想想重装 windows 系统时，经常说`需要一个镜像`，那 image 跟这里的镜像特别类似

# 安装 Docker

### MacOS

docker 官网手动下载安装即可

安装成功启动后如下图

![](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/2022_05_27_2337242022_05_27_233650docker-installcYbpBjYgvV9S.png)

可以在终端里通过命令查看 Docker 是否成功安装

```shell
$ docker --version

Docker version 20.10.14, build a224086
```

或者

```shell
$ docker version
```

输出

```sh
Client:
 Cloud integration: v1.0.24
 Version:           20.10.14
 API version:       1.41
 Go version:        go1.16.15
 Git commit:        a224086
 Built:             Thu Mar 24 01:49:20 2022
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true

Server: Docker Desktop 4.8.2 (79419)
 Engine:
  Version:          20.10.14
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.15
  Git commit:       87a90dc
  Built:            Thu Mar 24 01:46:14 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.11
  GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc:
  Version:          1.0.3
  GitCommit:        v1.0.3-0-gf46b6ba
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

确认安装成功以后，需要配置镜像加速器

```json
{
  "registry-mirrors": [
    "http://f1361db2.m.daocloud.io",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

如图

![](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/2022_05_27_2347522022_05_27_234737Shx3bj4JGk0JMVCCYO.png)

重启完成后，运行如下命令

```shell
$ docker info
```

输出结果中可以看到

```sh
 Registry Mirrors:
  http://f1361db2.m.daocloud.io/
  https://hub-mirror.c.163.com/
  https://mirror.baidubce.com/
```

就说明配置成功

安装成功后，可以尝试运行一个 nginx 服务器

```shell
$ docker run -d -p 80:80 --name webserver nginx
```

服务正常运行后，浏览器里访问 `http://localhost`，如果能出现 Welcome to Nginx 的 web 页面，就说明 docker 可以正常运行了，第一个 docker 容器启动成功

停止刚刚启动的服务器使用如下命令
```shell
$ docker stop webserver
$ docker rm webserver
```

# 使用镜像

Docker 官方提供了中央镜像仓库 `Docker hub`，里面集中存放了大量镜像

如果用 Git 项目作类比，那么镜像仓库就相当于 GitHub、Gitlab 等代码托管平台，只不过 GitHub 托管的是各种代码项目，而 Docker hub 托管的是一个个镜像

我们可以使用 `docker pull`命令从镜像仓库里拉取镜像

![](https://assets-1253723501.cos.ap-beijing.myqcloud.com/uPic/2022_05_29_1020052022_05_29_101949fSXhTe5OcL8wUEGPyI.png)

### 镜像命名

镜像通过`<仓库名>:<标签>`来指定具体是哪个软件的哪个版本，仓库名包含用户名和软件名两部分，经常以两段路径的形式出现 `<用户名>/<软件名>`

以上面的 nginx 为例，我们忽略了标签，则以 latest 为默认标签，即相当于`nginx:latest`，如果不给出用户名，则表示 docker 的官方镜像，默认为 `library`

### 查看镜像

我们可以通过 `docker images`来查看当前已经存放和管理了哪些镜像，`docker image ls` 也可以

| REPOSITORY | TAG    | IMAGE ID     | CREATED      | SIZE   |
| ---------- | ------ | ------------ | ------------ | ------ |
| nginx      | latest | de2543b9436b | 21 hours ago | 142 MB |

查看某个镜像的详细信息，可以使用 `docker inspect` 命令

```shell
$ docker inspect nginx
```

### 删除镜像

```shell
$ docker rmi nginx:latest
```

# 容器

容器有自己的生命状态流转，可分为：

- Created
- Running
- Paused
- Stopped
- Deleted

### 创建容器

镜像拉取成功后，可以基于它创建一个容器，使用 --name 给容器命名

```shell
$ docker create --name webserver nginx
```

此时创建成功的容器状态为 Created，内部的应用程序还没有运行，可以使用 docker start 命令来启动

```shell
$ docker start webserver
```

也可以使用 `docker run` 直接启动

-d 参数代表 --detach，一般来说我们期待让容器运行在后台而非直接把执行命令输出到当前的宿主机下

### 查看容器

使用 `docker ps` 查看处于运行中的容器，要显示所有状态的容器，增加 -a 参数选项

### 停止容器

```shell
$ docker stop webserver
```

### 删除容器

```shell
$ docker rm webserver
```

默认删除的是一个处于终止状态的容器，如果要删除一个运行中的容器，可添加 -f 参数选项

### 外部访问容器

要让外部访问到容器中运行的网络应用，可以使用 -p 参数来指定端口映射

使用 `hostPort:containerPort`格式，将本地的 80 端口映射到容器到 80 端口

# 构建镜像





