# Docker 文档

## 一、内网中的 Docker 服务器

### 1. 登录信息

| 服务器 IP | 192.168.3.222 |
| --------- | ------------- |
| 用户名    | docker        |
| 密码      | docker123     |

## 二、Ubuntu 系统中 Docker 的安装

在 Ubuntu 中安装 Docker 有两种方式：

一种是直接在软件源加入 Docker 官方的 PPA 仓库地址，之后直接使用软件包管理器 Apt 安装，这种方法会在安装时从仓库中下载最新版本的安装包，并且可以直接使用 `apt upgrade` 命令更新，但是 Docekr 官方PPA仓库的速度非常慢，需要下载很长时间。

另一种方式是直接使用下载好的安装包进行安装，这样更新会稍微麻烦一些。

### 1. 使用安装包安装

打开这个[下载连接](https://download.docker.com/linux/ubuntu/dists/)，选择使用的 Ubuntu 系统的版本，打开 `/pool/stable/amd64/` ，在该文件夹中下载 `containerd.io_*.deb, docker-ce_*.deb, docker-ce-cli_*.deb` 这三个安装包。

> Ubuntu 一般使用长期支持的 LTS 版本，即 Ubuntu 16.04 代号为 Xenial Xerus 或者 Ubuntu 18.04 代号为 Bionic Beaver 。
>

或直接使用 deb 文件夹下已经下载好的 Docker 安装包（适用于 Ubuntu 18.04）。

安装 deb 安装包的命令如下：

```bash
$ sudo dpkg -i ./deb/containerd.io_1.2.2-3_amd64.deb
$ sudo dpkg -i ./deb/docker-ce-cli_18.09.2~3-0~ubuntu-bionic_amd64.deb

// 可能会报错提示缺少需要的软件包，执行以下命令自动修复依赖关系
$ sudo apt --fix-broken install

$ sudo dpkg -i ./deb/docker-ce_18.09.2~3-0~ubuntu-bionic_amd64.deb
```

注意：docker-ce 依赖于之前的两个安装包，所以安装顺序不能颠倒。

执行以下命令可以测试 docker 是否安装成功

```bash
$ sudo docker run hello-world
```

出现以下内容表示安装成功：

> Hello from Docker!
> This message shows that your installation appears to be working correctly.

执行以下命令查看当前安装的 Docker 版本：

```bash
$ docker version
```

### 2. 使用 apt 仓库安装

1. 更新 `apt` 软件包索引：

   ```bash
   $ sudo apt-get update
   ```

2. 安装软件包，以允许 `apt` 通过 HTTPS 使用镜像仓库：

   ```bash
   $ sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
   ```

3. 添加 Docker 的官方 GPG 密钥：

   ```bash
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   $ sudo apt-key fingerprint 0EBFCD88
   ```

4. 使用下列命令设置 **stable** 镜像仓库：

   ```bash
   $ sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   ```

5. 再次更新 `apt` 软件包索引：

   ```bash
   $ sudo apt-get update
   ```

6. 安装最新版本的 Docker CE：

   ```bash
   $ sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

7. 验证是否正确安装了 Docker CE：

   ```bash
   $ sudo docker run hello-world
   ```

### 3. 使用 Docker 中国官方镜像加速

> 在中国大陆地区直接访问 Docker 的官方服务器会非常慢，所以往往需要使用国内的镜像服务器，比如使用 Docker 中国的镜像。
>
> 通过 Docker 官方镜像加速，中国区用户能够快速访问最流行的 Docker 镜像。该镜像托管于中国大陆，本地用户现在将会享受到更快的下载速度和更强的稳定性，从而能够更敏捷地开发和交付 Docker 化应用。

使用方法：修改（如果没有这个文件就新建一个） `/etc/docker/daemon.json` 文件并添加上 registry-mirrors 键值。

```json
{
  "registry-mirrors": [
      "https://registry.docker-cn.com"
  ]
}
```

修改保存后重启 Docker 以使配置生效。

## 三、Docker 概念

### 1. 什么是 Docker

> Docker是一个虚拟环境容器，可以将你的开发环境、代码、配置文件等一并打包到这个容器中，并发布和应用到任意平台中。比如，你在本地用 SpringBoot 开发网站后台，开发测试完成后，就可以将打包好的 Java 程序、JDK 及其依赖包、Mysql、Nginx等打包到一个容器中，然后部署到任意你想部署到的环境。

### 2. Docker 中的三个重要概念

1. 镜像（Image）

> 类似于虚拟机中的镜像，是一个包含有文件系统的面向Docker引擎的只读模板。任何应用程序运行都需要环境，而镜像就是用来提供这种运行环境的。例如一个Ubuntu镜像就是一个包含Ubuntu操作系统环境的模板，同理在该镜像上装上Apache软件，就可以称为Apache镜像。

在 Docker 的官方仓库 [Docker Hub](https://hub.docker.com/) 中有大量的镜像，其中包括官方提供的各种基础的系统镜像和常用的软件镜像，同时也有很多用户自己提交分享的镜像。我们在使用制作自己的镜像时一般也需要使用仓库中提供的镜像作为基础镜像，比如常用的 Ubuntu、Debian和体积非常小的 Alpine 这几种操作系统的镜像。以及一些常用的比如：Redis、Nginx、Maven、OpenJDK 的镜像等等

2. 容器（Container）

> 类似于一个轻量级的沙盒，可以将其看作一个极简的Linux系统环境（包括root权限、进程空间、用户空间和网络空间等），以及运行在其中的应用程序。Docker引擎利用容器来运行、隔离各个应用。容器是镜像创建的应用实例，可以创建、启动、停止、删除容器，各个容器之间是是相互隔离的，互不影响。注意：镜像本身是只读的，容器从镜像启动时，Docker在镜像的上层创建一个可写层，镜像本身不变。
>

每次使用 Docker 启动一个镜像，都会自动产生一个新的容器，容器包含了运行期间产生的各种数据和修改。

3. 仓库（Repository）

> 类似于代码仓库，这里是镜像仓库，是Docker用来集中存放镜像文件的地方。一般每个仓库存放一类镜像，每个镜像利用tag进行区分。

 最常用的仓库是 Docker 的官方仓库 [Docker Hub](https://hub.docker.com/) ，此外还有一些云平台提供的仓库，也可以自己搭建自己的私人仓库，官方提供了搭建私人仓库的 Docker 镜像，可以直接使用镜像来搭建。

## 四、Docker 的使用

### 1. 运行仓库中的镜像

仓库中的镜像包含很多不同的版本，默认使用的是标签为 latest 的版本，我们要根据需求选择需要的版本。Docker 通过镜像的标签（tag）来区分镜像版本，一个镜像往往有大量不同的标签，在拉取镜像时一般需要手动指定标签。以 Redis 仓库为例，包含以下这些标签：

```
5.0.3, 5.0, 5, latest, 5.0.3-stretch, 5.0-stretch, 5-stretch, stretch

5.0.3-32bit, 5.0-32bit, 5-32bit, 32bit, 5.0.3-32bit-stretch, 5.0-32bit-stretch, 5-32bit-stretch, 32bit-stretch

5.0.3-alpine, 5.0-alpine, 5-alpine, alpine, 5.0.3-alpine3.9, 5.0-alpine3.9, 5-alpine3.9, alpine3.9

4.0.13, 4.0, 4, 4.0.13-stretch, 4.0-stretch, 4-stretch

4.0.13-32bit, 4.0-32bit, 4-32bit, 4.0.13-32bit-stretch, 4.0-32bit-stretch, 4-32bit-stretch

4.0.13-alpine, 4.0-alpine, 4-alpine, 4.0.13-alpine3.9, 4.0-alpine3.9, 4-alpine3.9
```

可以使用如下命令将镜像下载到本地：

```bash
# 不加标签时默认为latest，会获取当前最新版的默认镜像
$ docker pull redis
$ docker pull redis:latest

# 指定标签 5，会下载版本号5开头的最新版本，如5.2.1、5.3.4等，Redis 默认版本使用 Debian 系统作为基础镜像构建
$ docker pull redis:5

# 指定标签 5.0，会下载最新的5.0版本，如5.0.1、5.0.3，这样指定版本号可以保证不会因为升级导致一些无法预料的兼容性问题
$ docker pull redis:5.0

# 指定标签 5-alpine，会下载使用 Alpine 系统作为基础镜像构建的版本，体积更小
$ docker pull redis:5-alpine
```

我们可以直接运行并使用仓库中的镜像

#### 1. 在后台运行镜像

```bash
# 比如如下命令会在后台启动一个名叫 jwkq-redis-server 的 Redis 容器
# Docker 会先在本地查找是否有 Redis 的镜像，如果有的话直接用该镜像启动容器，否则会去仓库中下载
# --name 表示给该容器取一个名字，-d 表示在后台启动这个容器，并且把容器 id 显示出来
$ docker run --name jwkq-redis-server -d redis:5.0-alpine
```

#### 2. 直接在终端中运行镜像

```bash
# 该命令会启动一个 Ubuntu 的容器，可以直接通过终端在其中执行命令
# -i 表示交互式操作，-t 表示终端
# /bin/bash 表示在其中执行这个命令（即启动bash）
$ docker run -it ubuntu:18.04 /bin/bash
```

#### 3. 常用命令

```bash
# 后台启动一个 Nginx 容器，使用 alpine 版本的镜像，命名为 frontend，并将容器中的80端口映射到本地的8080端口上
$ docker run -d --name frontend -p 8080:80 nginx:alpine
```

```bash
# 在运行中的 frontend 容器中执行 /bin/sh 命令，并使用终端交互
# 可以通过这种方式在容器中执行命令，比如修改配置文件等，使用 Ctrl + D 退出该模式
$ docker exec -it frontend /bin/sh
```

```bash
# 列出所有本地的镜像
$ docker image ls

# 列出所有容器，-a 表示包括已经停止运行的容器，-s 显示容器的大小
$ docker container ls -as

# 列出所有容器，-a 表示包括已经停止运行的容器
$ docker ps -a

# 显示容器的运行情况
$ docker stats

# 显示某个容器中运行的进程，jwkq-nginx 是容器的名字
$ docker top jwkq-nginx
```

```bash
# 杀死一个正在运行的容器
$ docker kill jwkq-nginx

# 停止一个正在运行的容器
$ docker stop jwkq-nginx

# 启动一个停止的容器
$ docker start jwkq-nginx
```

```bash
# 删除一个本地的镜像
$ docker rmi nginx:alpine

# 删除一个停止的容器
$ docker rm jwkq-nginx
```

### 2. 将项目文件构建在镜像中

上面介绍的是使用已有的镜像，在实际部署中需要自己构建镜像。Docker 镜像的构建需要用到 Dockerfile。

#### 1. 使用编译好的文件打包镜像

比如我们想要构建一个运行 java 项目的镜像，可以这样做：

```dockerfile
# 使用一个包含了 OracleJDK 的 alpine 系统镜像作为基础
FROM anapsix/alpine-java:8
# 将打包目录下的 app.jar 复制到容器中的 /root 目录下
COPY ./app.jar /root/
# 声明要暴露的端口
EXPOSE 8080
# 在容器启动后执行的命令：java -jar /root/app.jar
CMD ["java", "-jar", "/root/app.jar"]
```

需要新建一个文件夹，将上面的内容保存成 Dockerfile 文件，将编译好的 app.jar 文件放在也放在当前文件夹中，执行打包命令：

```bash
# -t 表示镜像名为 backend，镜像标签为 0.1
# 命令结尾的 . 表示以当前目录作为打包目录
$ docker build -t backend:0.1 .
```

同样也可以把前端项目打包为镜像。

```dockerfile
# 使用 nginx 作为基础镜像
FROM nginx:alpine
# 将 vue 编译好的文件复制到 nginx 的目录下
COPY ./dist /usr/share/nginx/html
EXPOSE 80
# 使用命令在前台启动 nginx
CMD ["nginx", "-g", "daemon off;"]
```

#### 2. 直接将源代码文件编译并打包为镜像

除了打包编译好的文件，还可以直接通过源码打包项目。

我们可以像上面一样直接在一个镜像中下载源码，编译，然后打包成一个镜像。但是在编译时我们需要 maven，JDK 等体积比较大的包，直接打包成镜像会导致镜像体积很大。这时候可以通过多阶段构建的方式打包，可以减小镜像的体积。具体的 Dockerfile 如下所示：

编译并打包 java 项目

先根据需求写一个 maven 的配置文件，指定本地仓库地址和镜像地址等。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/data/maven/repository</localRepository>
  <mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
</settings>
```

```dockerfile
# 使用包含 jdk8 的 maven 镜像作为编译用的镜像，并命名为 builder
FROM maven:3-jdk-8 as builder
# 将自定义的 maven 配置文件复制到镜像中
COPY ./files /root/files
# 复制 maven 的配置文件到正确的目录，用 git 拉取项目的源码，用 maven 打包，并将打包好的jar文件移动到 /root 目录
RUN mkdir /root/.m2 \
    && cp /root/files/settings.xml /root/.m2/ \
    && cd /root \
    && git clone http://192.168.3.213:3000/walter/springboot.git \
    && cd /root/springboot \
    && mvn package -Dmaven.test.skip=true \
    && mv /root/files/project/target/*.jar /root/app.jar
# 第二阶段，使用一个包含了 OracleJDK 的 alpine 系统镜像作为基础
FROM anapsix/alpine-java:8
# 将上一步编译好的文件复制到 /root/app/ 目录
COPY --from=builder /root/app.jar /root/app/
# 修改当前的工作目录，项目执行时生成的日志一般会保存在工作目录下，可以在运行容器时将工作目录挂在到本地
WORKDIR /root/workdir/
EXPOSE 8080
# 启动 java 项目
CMD ["java", "-jar", "/root/app/app.jar"]
```

编译并打包 vue 项目

```dockerfile
# 使用包含 node 环境的镜像作为编译项目的镜像，并命名为 builder
FROM node:latest as builder
# 执行以下命令，用 git 拉取前端项目的源码，安装相关依赖，编译打包项目
# 此处使用淘宝的 npm 镜像，否则会比较慢
RUN cd /root/ \
    && git clone http://192.168.3.213:3000/walter/vue-admin.git \
    && cd vue-admin/ \
    && yarn config set cache ~/.my-yarn-cache \
    && yarn --registry=https://registry.npm.taobao.org \
    && yarn build
# 第二阶段，以 nginx 作为基础镜像
FROM nginx:alpine
# 将上一步编译好的文件复制到 nginx 目录
COPY --from=builder /root/vue-admin/dist /usr/share/nginx/html
EXPOSE 80
# 启动 nginx
CMD ["nginx", "-g", "daemon off;"]
```

写好 Dockerfile 之后一样执行下面的命令就可以打包成镜像

```bash
$ docker build -t image_name:tag .
```

