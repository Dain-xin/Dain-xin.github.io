---
title: Docke使用说明
date: 2022-05-24 14:47:17
categories:
- 环境搭建
- Docker
---

# Docker

Docker 官方网站：https://www.docker.com/

[Docker 教程 | 菜鸟教程 ](https://www.runoob.com/docker/docker-tutorial.html)

**使用场景**

- Web 应用的自动化打包和发布。
- 自动化测试和持续集成、发布。
- 在服务型环境中部署和调整数据库或其他的后台应用。

**Docker 的优点**

> Docker 是一个用于开发，交付和运行应用程序的开放平台。
>
> Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。
>
> 借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。
>
> 利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

1、快速，一致地交付您的应用程序

Docker 允许开发人员使用您提供的应用程序或服务的本地容器在标准化环境中工作，从而简化了开发的生命周期。

容器非常适合持续集成和持续交付（CI / CD）工作流程，请考虑以下示例方案：

- 您的开发人员在本地编写代码，并使用 Docker 容器与同事共享他们的工作。
- 他们使用 Docker 将其应用程序推送到测试环境中，并执行自动或手动测试。
- 当开发人员发现错误时，他们可以在开发环境中对其进行修复，然后将其重新部署到测试环境中，以进行测试和验证。
- 测试完成后，将修补程序推送给生产环境，就像将更新的镜像推送到生产环境一样简单。

2、响应式部署和扩展

Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。

Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。

3、在同一硬件上运行更多工作负载

Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## Docker 安装卸载

安装过程：

> CentOS 6 及以后的版本，Docker 并不是容器，它是一个管理容器的引擎

CentOS7 系统可以直接通过 yum 进行安装：

```
yum update
```

系统是否已经安装了 Docker:

```
yum list installed | grep docker
```

安装：

```
yum install docker -y
```

安装后，看 docker 是否安装成功:

```
docker –version
```

卸载：

```
yum remove docker.x86_64 -y
yum remove docker-client.x86_64 -y
yum remove docker-common.x86_64 -y
#或者用
rm -rf /var/lib/docker
```

## Docker 服务启动

[Docker 启动失败——Job for docker.service failed because the control process exited with error... ](http://www.javashuo.com/article/p-zotynuma-ox.html)

启动-停止-重启：

```
systemctl start docker 或者 service docker start

systemctl stop docker 或者 service docker stop

systemctl restart docker 或者 service docker restart
```

检查 docker 进程的运行状态：

```
systemctl status docker 或者 service docker status
```

查看 docker 进程：

```
ps -ef | grep docker
```

## Docker 服务信息

```
docker info
#查看 docker 系统信息
docker
#查看所有的帮助信息
docker commond –help
#查看某个 commond 命令的帮助信息
```

## Docker 运行机制

### 第一个 Docker 容器

根据 Docker 的运行机制，我们将按照如下步骤运行第一个 Docker 容器；

1、将 Docker 服务启动；

2、下载一个镜像，Docker 运行一个容器前需要本地存在有对应的镜像，如果镜像不存在本地，Docker 会从镜像仓库下载（默认是 Docker Hub 公共注册服务器中的仓库 [https://hub.docker.com](https://hub.docker.com/)）。

> **从 docker hub 官网搜索要使用的镜像，也可以在命令行使用命令搜索要使用的镜像，**
>
> **比如 docker search tomcat 进行搜索，然后下载所需要的镜像：**

下载镜像：

```
docker pull tomcat
```

运行镜像：

```
docker run tomcat

#前台运行， 要后台运行，加参数 -d
```

显示本地已有的镜像（仓库和 ID 等信息）：

```
docker images
```

在列出信息中，可以看到几个字段信息

> REPOSITORY：来自于哪个仓库，比如 docker.io/tomcat TAG：镜像的标记，比如 latest
> IMAGE ID：镜像的 ID 号（唯一）
> CREATED：创建时间
> SIZE：镜像大小

![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205241457281.png)

启动下载下来的镜像得到一个容器（可以通过）：

```
docker run -d docker.io/tomcat 或 者 docker run -d 41a54fe1f79d（镜像ID）
#默认是前台启动，如果需要后台启动，指定-d 参数；
ps -ef | grep tomcat
#查看，检查 tomcat 镜像是否启动容器成功；
docker ps
#查看容器ID
```

![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205241457281.png)

### 进入 Docker 容器

> 进入镜像可以修改对应容器的配置，没有 ll，有 ls 命令；
>
> i 表示交互式的，也就是保持标准输入流打开；
>
> t 表示虚拟控制台，分配到一个虚拟控制台；

```
#进入容器：
docker exec -it cef0d139bfd6（容器ID） bash

#退出容器：
exit
```

### 客户机访问容器

> 从客户机上访问容器，需要有端口映射，docker 容器默认采用桥接模式与宿主机通信，
>
> 需要将宿主机的 ip 端口映射到容器的 ip 端口上； （就是 Docker 容器和主机的端口映射）

```
#停止容器：使用docker ps 查看容器ID
docker stop 容器 ID/名称

#停止所有容器
docker stop ${docker ps -q}

#删除容器
docker stop ${docker ps -aq}

#启动容器：使用docker images 查看镜像信息
docker run -d -p 8080:8080 docker.io/tomcat（仓库位置）或者（镜像ID）

#查看状态和关闭防火墙
systemctl status firewalld
systemctl stop firewalld
```

## Docker 核心组件

### Docker 架构

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程 API 来管理和创建 Docker 容器。

Docker 容器通过 Docker 镜像来创建。

镜像与容器的关系类似于面向对象编程中的类与对象的关系。

| Docker | 面向对象 |
| ------ | -------- |
| 镜像   | 类       |
| 容器   | 对象     |

`通过镜像来创建容器；就是通过类创建对象`

![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205241458042.png)

| 概念                   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| :--------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板。                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。                                                                                                                                                                                                                                                                                                                                                          |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。 Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine 是一个简化 Docker 安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装 Docker，比如 VirtualBox、 Digital Ocean、Microsoft Azure。                                                                                                                                                                                                                                                                                                                      |

### Docker 核心要素

#### 三个核心要素

镜像（Image）、容器（Container）、仓库（Repository）

理解了这三个概念，就理解了 Docker 的整个生命周期。

`有人会误以为，Docker 就是容器，但 Docker 不是容器，而是管理容器的引擎。`

### Docker 镜像

#### 镜像的基本概念

Docker 镜像就是一个只读的模板，可以用来创建 Docker 容器。

> 例如：一个镜像可以包含一个完整的 centos 操作系统环境，里面仅安装了 mysql 或用户需要的其它应用程序。Docker 提供了一个非常简单的机制来创建镜像或者更新现有的镜像， 用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

#### 镜像的组成结构

镜像是由许多层的文件系统叠加构成的，最下面是一个引导文件系统

bootfs，第二层是一个 root 文件系统 rootfs，root 文件系统通常是某种操作系统，比如 centos、Ubuntu，在 root 文件系统之上又有很多层文件系统，这些文件系统叠加在一起，构成 docker 中的镜像；

#### 镜像的日常操作

1、下载镜像，比如下载 redis 镜像：

```
docker pull redis:latest
docker pull redis
```

reids 是查询到的镜像名称，latest 是镜像的标签 tag

获取一个镜像有两种方式，一种是从官方镜像仓库下载，一种是自己通过 Dockerfile 文件构建。

如果有官方镜像，我们就不必自己用 Dockerfile 文件构建了，除非官方没有才会自己去 Dockerfile 文件构建；

2、列出已经下载的镜像：

```
docker images

docker images redis
```

3、运行镜像：

```
docker run -d redis

#其中-d 表示在后台运行然后通过
#可以查到redis 进程

ps -ef | grep redis
```

4、查看容器镜像的状态：

```
docker ps
```

进入 redis 容器:

```
docker  exec  -it  a8584016f9b6(镜像ID)  bash
#退出容器
exit
```

5、删除镜像：（先删容器再删镜像）

```
docker rmi redis:latest
```

`注意是rmi，不是 rm，rm 是删除容器；`

停止所有容器

```
docker stop ${docker ps -q}
```

删除容器

```
docker stop ${docker ps -aq}
```

停止容器：使用 docker ps 查看容器 ID

```
docker stop 容器ID/名称
```

6、启动 redis：

```
docker run -d -p 6379:6379 redis 或者 镜像ID
```

### Docker 容器

#### **容器的基本概念**

> 容器是从镜像创建的运行实例。它可以被启动、停止、删除。
>
> 每个容器都是相互隔离的、保证安全平台。
>
> 可以把看做一个简易版的 Linux 环境，包括 root 用户权限、进程空间、用户空间和网络空间和运行在 其 中 的 应 用 程 序 。
>
> Docker 利用容器来运行应用，镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205241458042.png)

`注意：一个镜像可以创建多个容器，docker ps -a 可以查询多个容器，理解就是运行新的镜像会创建新的容器，但是，创建的容器，也可以通过docker start 容器ID启动创建的容器`

#### 容器的日常操作

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态的容器重新启动。

> **-p :** 是容器内部端口绑定到**指定**的主机端口。

通过镜像启动容器：

```
docker run -d redis
docker run -d -p 6379:6379 redis
```

查看运行中的容器：

```
docker ps
```

查看所有的容器：

```
docker ps -a
```

停止容器：

```
docker stop  容器id 或容器名称
```

已经停止的容器，我们可以使用命令 docker start 来启动。

开启容器：

```
docker start 容器id 或容器名称
```

> `因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。`
>
> `删除容器时，容器必须是停止状态，否则会报错； `

删除容器：

```
docker rm 容器id 或容器名称
```

进入容器：

```
docker exec -it 容器id 或容器名称 bash
```

查看容器的更多信息:

```
docker inspect + 容器id 或容器名称
```

停用全部运行中的容器：

```
docker stop $(docker ps -q)
```

删除全部容器：

```
docker rm $(docker ps -aq)
```

一条命令实现停用并删除容器：

```
docker stop $(docker ps -q) & docker rm -f $(docker ps -aq)
```

### Docker 仓库

#### **仓库的基本概念**

仓库是集中存放镜像文件的场所，有时候会把仓库和仓库注册服务器（Registry）看做同一事物，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）；

仓库分为公开仓库（Public）和私有仓库（Private）两种形式；

最大的公开仓库是 Docker Hub （https://hub.docker.com/），存放了数量庞大的镜像供用户下载；

用户也可以在本地网络内创建一个私有仓库；

当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，

这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 pull 下来即可；

`注：Docker 仓库的概念跟 Git 类似，注册服务器也类似于 GitHub 这样的托管服务；`

#### 仓库的日常操作

用户可通过 docker search 命令来查找官方仓库中的镜像：

```
docker search rabbitmq
```

可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、星级（表示该镜像的受欢迎程度）、是否官方创建、是否自动创建； 官方的镜像说明是官方项目组创建和维护的，**automated 资源允许用户验证镜像的来源和内容；**

根据是否是官方提供，可将镜像资源分为两类:

一种是类似 centos 这样的基础镜像，被称为基础或根镜像。这些基础镜像是由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字；

还有一种类型，比如 tianon/centos 镜像，它是由 Docker 的用户创建并维护的，往往带有用户名称前缀。可以通过前缀 user_name/ 来指定使用某个用户提供的镜像，比如 tianon 用户；

并利用 docker pull 命令来将它下载到本地：

```
docker pull rabbitmq

docker pull centos
```

## Docker 镜像加速

阿里云镜像获取地址：[容器镜像服务 (aliyun.com)](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

登陆后，左侧菜单选中镜像加速器就可以看到你的专属地址了：
![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205241458960.png)

```
#复制的代码
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://svye10a1.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## Docker 使用示例

### Docker 安 装 MySQL

[Docker 安装 MySQL | 菜鸟教程](https://www.runoob.com/docker/docker-install-mysql.html)

```
#下载MySQL 镜像：（其中-e 是指定环境变量）
docker pull mysql:latest	（安装的是mysql 8.0）
docker run -p 3306:3306 -e MYSQL_DATABASE=workdb -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

#进入容器：
docker exec -it 3e8bf7392b4e bash
#登录MySQL：
mysql -u root -p
#修改密码：
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
#授权远程登录访问：
CREATE USER 'wkcto'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO ' wkcto'@'%';
```

### Docker 安 装 Nginx

```
#下载 Nginx 镜像：
docker pull nginx
docker run -d -p 80:80 nginx

#进入容器：
docker exec -it 3e8bf7392b4e bash

#浏览器访问Nginx：
http://192.168.230.128:80

#Nginx 部署静态网站：
#将linux 的文件拷贝到 docker 容器某个目录下：
docker cp /root/test.html bf8a58328e18:/usr/share/nginx/html

#Nginx修改配置文件需要vi编辑器去etc/nginx 中修改，
```

### Dokcer 安 装 Zookeeper

```
#下载Zookeeper 镜像：
docker pull zookeeper
docker run -p 2181:2181 -d zookeeper

#进入容器：
docker exec -it 3e8bf7392b4e bash
#客户端工具访问 Zookeeper：
```

### Docker 安装 RabbitMQ

```
docker pull rabbitmq:management

docker run -di --name=peijie_rabbitmq -p 5672:5672 -p 5671:5671 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management
```

### 更多实例

[Docker 安装实例| 菜鸟教程](https://www.runoob.com/docker/docker-install-nginx.html)

查找安装实例的路径

```
find / -name tomcat
```

解决 Docker 容器没有 vi 编辑器

```
#1、更新软件包再下载
apt-get update
apt-get install vi
#2、可以使用cat命令实现docker容器内写入的目的
cat > hk.html << EOF, hk.html命令是希望使用cat写入的目标文件，
<< 后面指定的EOF是指定的交互式输入终止符

```

## Docker 自定义镜像

### Dockerfile 文件和基本结构

> Dockerfile 用于构建 Docker 镜像，Dockerfile 文件是由一行行命令语句组成， 基于这些命令即可以构建一个镜像；
>
> Dockerfile 分为四部分：
>
> 基础镜像信息； 维护者信息； 镜像操作指令；容器启动时执行指令；
>
> ```
> #例如
>
> FROM centos:latest
> MAINTAINER wkcto
> ADD jdk-8u121-linux-x64.tar.gz /usr/local
> ENV JAVA_HOME /usr/local/jdk1.8.0_121
> ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
> ENV PATH $PATH:$JAVA_HOME/bin
> CMD java -version
> ```

### **Dockerfile** **指令**

1、FROM

Dockerfile 文件的第一条指令必须为 FROM 指令。

并且，如果在同一个 Dockerfile 中创建多个镜像时，可以使用多个 FROM 指令（每个镜像一次）；

2、MAINTAINER

指定维护者信息；

3、ENV

指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持；

4、ADD

复制指定的<src>到容器中的<dest>；

5、EXPOSE

告诉 Docker 服务端容器暴露的端口号，供互联系统使用，在启动容器时需要通过 -p 映射端口，Docker 主机会自动分配一个端口转发到指定的端口；

6、RUN

RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像， 当命令较长时可以使用 \ 来换行；

7、CMD

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

### 自定义镜像

定义 Dockerfile 文件： (vim Dockerfile)

**JDK**

```
FROM centos:latest
MAINTAINER djx
ADD jdk-8u121-linux-x64.tar.gz /usr/local
ENV JAVA_HOME /usr/local/jdk1.8.0_121
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:$JAVA_HOME/bin
CMD java -version
```

**Tomcat**

```
FROM djx_jdk1.8.0_121
MAINTAINER djx
ADD apache-tomcat-8.5.24.tar.gz /usr/local/
ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.24
ENV PATH $PATH:$CATALINA_HOME/lib:$CATALINA_HOME/bin
EXPOSE 8080
CMD /usr/local/apache-tomcat-8.5.24/bin/catalina.sh run
```

**Mysql**

```
FROM centos:centos6

MAINTAINER djx

RUN yum install mysql-server mysql -y

RUN /etc/init.d/mysqld start &&\
mysql -e "grant all privileges on *.* to 'root'@'%' identified by '123456' WITH GRANT OPTION ;"&&\
mysql -e "grant all privileges on *.* to 'root'@'localhost' identified by '123456'
WITH GRANT OPTION ;"&&\
mysql -uroot -p123456 -e "show databases;"

EXPOSE 3306

CMD /usr/bin/mysqld_safe
```

**Redis**

```
FROM centos:latest
MAINTAINER djx
RUN yum install epel-release -y && yum install redis -y && yum install net-tools -y EXPOSE 6379
CMD /usr/bin/redis-server –protected-mode no
```

构建镜像：

```
docker build -t djx_jdk1.8.0_121（镜像名）

docker build -t djx-tomcat-8.5.24 （镜像名）

docker build -t djx-mysql

docker build -t djx-redis

```

运行镜像：(docker images：查看镜像名前面的镜像 ID)

```
docker run -d 镜像ID

docker run -d -p 8080:8080 镜像ID

docker run -d -p 3306:3306 镜像ID

docker run -d -p 6379:6379 镜像ID
```

## Docker 项目部署

#### Jar 包

1、上传 jar 文件到 /root/docker；定义 dockerfile 文件，用于创建项目镜像。

2、文件内容

```
FROM djx_jdk1.8.0_121
MAINTAINER djx
ADD springboot-web-1.0.0.jar（jar的名字） /opt
RUN chmod +x /opt/springboot-web-1.0.0.jar（jar名字） CMD java -jar /opt/springboot-web-1.0.0.jar（jar名字）
```

3、构建和运行

```
构建镜像：docker build -t 项目名（jar名字）
运行容器：docker run -d -p 项目需要的端口:映射到host的端口 容器ID
```

4、注意数据库或者其他你需要用的组件的启动

#### War 包

`必须放Tomcat服务器中`

1、同上上传，dockerfile 内容：

```
FROM djx-tomcat-8.5.24
MAINTAINER djx
ADD springboot-web-1.0.0.war（war名） /usr/local/apache-tomcat-8.5.24/webapps
EXPOSE 8080
CMD /usr/local/apache-tomcat-8.5.24/bin/catalina.sh run
```

2、构建

```
docker build -t springboot-web-war（war名）
docker run -d -p 8080:8080 项目名或容器ID
```

3、注意项目内嵌使用的端口，但是放入 tomcat 中就是使用 tomcat 的端口；打包 war 再 build 中添加<finalName>djx</finalName>这样打包会生成改名字的 war
