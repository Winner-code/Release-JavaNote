# ----docker-----





# **一、Docker 安装与启动**

以下安装步骤基于 CentOS8，并确保该虚拟机可以连接外网。

## 1  安装

### 1.1 下载 docker-ce repo

为本地 yum 提供远程 repo 信息。

```
curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
```



### 1.2 安装依赖

```
yum   install   -y  https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm
```

### **1.3** **安装 docker-ce**

```
yum install -y docker-ce
```



## 2  启动

### 2.1 启动命令

systemctl start docker 

service docker start

### 2.2 关闭命令

systemctl stop docker 

service docker stop

### 2.3 查看 Docker 状态

docker info

结果如下：

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)



# 二、  镜像加速器配置

默认情况下 Docker 从 Docker Hub 上下载镜像资源，但速度很慢，可以通过配置国内的镜像加速器来解决。

本课程以阿里云镜像加速器为例讲解

**1**  **访问阿里云**

https://www.aliyun.com/

# 2  进入控制台

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

# 3  搜索镜像加速器

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image015.gif)

# 4 选择对应的 OS 并配置

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image017.jpg)

# 5  验证镜像加速器配置

通过 docker info 命令验证镜像加速器配置，结果如下：

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image019.jpg)

**四、  Docker 镜像操作**

# 1 什么是 Docker 镜像

Docker 镜像是由文件系统叠加而成（是一种文件的存储形式）。最底端是一个文件引导系统，即 bootfs，这很像典型的 Linux/Unix 的引导文件系统。Docker 用户几乎永远不会和引导系统有什么交互。实际上，当一个容器启动后，它将会被移动到内存中，而引导文件系统则会被卸载，以留出更多的内存供磁盘镜像使用。Docker 容器启动是需要一些文件

的，而这些文件就可以称为 Docker 镜像。

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image021.jpg)

**2**   **列出镜像**列出 docker 下的所有镜像，命令： docker images    或者 docker image ls

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image023.jpg)

结果解释：

REPOSITORY：镜像所在的仓库名称

TAG：镜像标签

IMAGE ID：镜像 ID

CREATED：镜像的创建日期（不是获取该镜像的日期）

SIZE：镜像大小

# 3  搜索镜像

可使用命令搜索需要的镜像，命令： docker search 镜像名称

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image025.jpg)

结果解释：

NAME：仓库名称

DESCRIPTION：镜像描述

STARS：用户评价，反应一个镜像的受欢迎程度

OFFICIAL：是否官方

AUTOMATED：自动构建，表示该镜像由 Docker Hub 自动构建流程创建的

# 4  拉取镜像

拉取镜像相当于从远程 Registry 中下载镜像到本地，命令： docker pull 镜像名称

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image027.jpg)

# 5  删除镜像

删除本地镜像使用命令：

docker rmi $IMAGE_ID docker rmi $REPOSITORY:$TAG

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image029.jpg)

# 五、  Docker 容器操作

可以把容器看成简易版的 Linux 环境（包括 root 用户权限，进程空间，用户空间和网络空间等）和运行在其中的应用程序。

# 1  新建容器

docker create [options] 镜像名字或者 ID [COMMAND] [ARG...]

docker create -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root mysql docker create -p 3306:3306 --name mysql_new -e MYSQL_ROOT_PASSWORD=root \

-v /usr/local/docker/mysql/conf:/etc/mysql \

-v /usr/local/docker/mysql/logs:/var/log/mysql \ -v /usr/local/docker/mysql/data:/var/lib/mysql \ mysql

## 1.1 options 常见参数说明

--name：给容器起一个新名字。为容器指定一个名称

-i：以交互模式运行容器，通常与-t 连用

-t：为容器重新分配一个伪终端，通常与-i 连用

-P：随机端口映射

-p：指定端口映射，hostPost：containerPort

-e：配置信息

-d：后台执行

-v：主机和容器的目录映射关系，":"前为主机目录，之后为容器目录

# 2  新建并启动容器

​      docker run [options] 镜像名字或者 ID  [COMMAND] [ARG...]

docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql docker run -p 3306:3306 --name mysql_new -e MYSQL_ROOT_PASSWORD=root \

-v /usr/local/docker/mysql/conf:/etc/mysql \

-v /usr/local/docker/mysql/logs:/var/log/mysql \

-v /usr/local/docker/mysql/data:/var/lib/mysql \

-d mysql

# 3  列出启动容器

列出正在运行的容器： docker container ls 查看 docker 容器进程信息： docker ps [options]

docker ps

## 3.1 options 参数说明

-l：显示最近创建的容器

-n 数字：显示最近 n 个创建的容器

-a：列出所有的运行过的容器

-q：列出容器 id

# 4  与运行中的容器交互

docker exec [options] 容器 ID [command] docker exec -it mysql /bin/bash

## 4.1 options 参数说明

-i：以交互模式运行容器，通常与-t 连用

-t：为容器重新分配一个伪终端，通常与-i 连用

# 5  停止容器

docker stop 容器 ID

docker stop mysql

# 6  启动容器

docker start 容器 ID

docker start mysql

# 7  强制停止容器

不推荐使用，容易造成容器服务不正常关闭，影响后续使用。

docker kill 容器 ID 或名称

docker kill mysql

# 8  暂停容器

docker pause 容器 ID 或名称

docker pause mysql

# 9  恢复容器

docker unpause 容器 ID 或名称

docker unpause mysql

# 10 删除容器

要删除的容器，必须是关闭状态的。

docker rm 容器 ID 或名称

docker rm mysql

# 11 查看容器日志

docker logs -f -t --tail 行数 容器 ID

docker logs -f -t --tail 5 mysql

-f 参数代表长期监控日志数据

-t 额外增加时间戳

--tail 行数 显示末尾多少行日志

# 12 查看容器中运行的进程

docker top 容器 ID 或名称

docker top mysql

# 13 查看容器内部详情

docker inspect 容器 ID 或名称

docker inspect mysql

# 14 复制容器数据到宿主机

docker cp 容器 ID 或名称:容器内路径 宿主机路径

​      复制 MySQL 配置到宿主机：  docker cp mysql:/etc/mysql ~/tmp/conf

# 六、  Docker File 管理

Docker File 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本。

案例构建一个 java 工程镜像。

# 1  使用本地命令构建镜像

## 1.1 下载 JDK 镜像

一般使用 openjdk 镜像。

docker search openjdk docker pull openjdk

## 1.2 创建构建文件

要构建到镜像中的 jar 文件需要和 buildFile 处于同一个目录。

vi ~/docker/buildFile

FROM openjdk:latest

VOLUME /var/mydatas

ADD cloudeureka-1.0-SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]

EXPOSE 8761

​     1.2.1  语法解释

\#指定基础镜像，这个需要根据自己配置的仓库上的版本写

FROM openjdk:latest

\#持久化目录

VOLUME /var/mydatas

\#指定源包，前者是你的 jar 包

ADD cloudeureka-1.0-SNAPSHOT.jar app.jar

\#指定容器启动时执行的命令

ENTRYPOINT ["java","-jar","/app.jar"]

\#对外端口

EXPOSE 8761

## 1.3 构建镜像

docker build -f 构建文件 -t 镜像名称:TAG 相对目录

docker build -f /root/docker/buildFile -t eureka:1.0 .

**1.4** **启动**

docker run --name eureka -p 8761:8761 -d eureka:1.0

# 2 使用 IDEA 构建镜像

## 2.1 修改 Docker 服务配置

vi /usr/lib/systemd/system/docker.service

在 ExecStart 变量末尾，增加下述配置：

-H unix:///var/run/docker.sock -H 0.0.0.0:2375

-H unix:///var/run/docker.sock : 开启一个对外主机服务，使用 docker.sock 文件管理。

-H 0.0.0.0:2375 : 允许什么客户端 IP 访问当前服务，当前服务对外暴露的端口号是什么。端口可自定义。

结果如下：

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image031.jpg)

## 2.2 重启 docker 服务

systemctl daemon-reload systemctl restart docker

## 2.3 IDEA 项目 POM 依赖

新增 plugin 插件配置：

<**plugin**>

<**groupId**>com.spotify</**groupId**>

<**artifactId**>docker-maven-plugin</**artifactId**>

<**version**>1.2.2</**version**>

<**configuration**>

<**imageName**>projects/eureka:1.0</**imageName**> <!--指定镜像名称仓库/镜像名:标签-->

<**baseImage**>openjdk:latest</**baseImage**> <!--指定基础镜像-->

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image032.gif)<**dockerHost**>http://192.168.89.128:2375</**dockerHost**> <!--指定服务器仓库地址 -->

<**entryPoint**>["java","-jar","/${project.build.finalName}.jar"]</**entryPoint**> <!--容器

启动执行的命令-->

<**exposes**>

<**expose**>8761</**expose**><!--发布端口-->

<**/esposes**>

<**resources**>

<**resource**>

<**targetPath**>/</**targetPath**> <!--指定要复制的目录路径，这里是当前目录-->

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image033.gif)![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image034.gif)<**directory**>${project.build.directory}</**directory**> <!--指定要复制的根目录，这里是target目录-->

<**include**>${project.build.finalName}.jar</**include**> <!--指定需要拷贝的文件，

这里指最后生成的jar包-->

</**resource**>

</**resources**>

</**configuration**> </**plugin**>

## 2.4 新增 IDEA 启动配置

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image039.gif)

最终结果如下：

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image041.jpg)

# 七、  阿里云镜像仓库

在阿里云容器镜像服务中，创建镜像仓库。并依据镜像仓库信息完成镜像的 push 和

pull。

# 1  push 镜像

docker login --username=jincg_ear registry.cn-hangzhou.aliyuncs.com docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:[ 镜像版本

号]

docker push registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:[镜像版本号]

具体命令如下：

docker login --username=jincg_ear registry.cn-hangzhou.aliyuncs.com

​      docker                        tag                        df689c674c72

registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:eureka_1.0 docker push registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:eureka_1.0

# 2  pull 镜像

docker pull registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:[镜像版本号]

具体命令如下：

docker pull registry.cn-hangzhou.aliyuncs.com/jincg_ear/jincg_repo:eureka_1.0

**八、  本地镜像仓库**

# 1  搭建本地仓库

**1.1** **下载本地仓库镜像**

docker pull registry

## 1.2 修改 Docker Service 配置

vi /usr/lib/systemd/system/docker.service

修改内容如下：

找到 Service 节点，在 ExecStart 属性末尾增加新参数，值为：

--insecure-registry ip:5000

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image043.jpg)

## 1.3 修改 Docker Daemon 配置

vi /etc/docker/daemon.json 新增配置内容：

{

"insecure-registries":["192.168.89.128:5000"]

}

具体如下：

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image045.jpg)

## 1.4 重启 Docker 服务

systemctl daemon-reload systemctl restart docker

**1.5** **启动容器**

docker run -p 5000:5000 -v /opt/registry:/var/lib/registry --name registry -d registry

**1.6** **容器启动状态**

docker ps -l

## 1.7 浏览器查看本地仓库

http://ip:5000/v2

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image047.jpg)

# 2  push 镜像

docker tag [ImageId] ip:5000/[镜像名称]:[镜像版本号] docker push ip:5000/[镜像名称]:[镜像版本号]

具体命令如下：

docker tag df689c674c72 192.168.89.128:5000/eureka:1.0 docker push 192.168.89.128:5000/eureka:1.0

查看 push 结果

http://ip:5000/v2/_catalog

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image049.jpg)

# 3  pull 镜像

docker pull ip:5000/[镜像名称]:[镜像版本号]

具体命令如下：

docker pull 192.168.89.128:5000/eureka:1.0

# 九、  Docker 容器的生命周期

![img](file:///C:/Users/WINNER/AppData/Local/Temp/msohtmlclip1/01/clip_image051.jpg)

# 1 状态介绍 1.1 圆形

代表容器的五种状态： created：初建状态 running：运行状态 stopped：停止状态 paused： 暂停状态 deleted： 删除状态

## 1.2 长方形

代表容器在执行某种命令后进入的过度状态： docker create ： 创建容器后，不立即启动运行，容器进入初建状态；

docker run ： 创建容器，并立即启动运行，进入运行状态； docker start ： 容器转为运行状态； docker stop ： 容器将转入停止状态；

docker kill ： 容器在故障（死机）时，执行 kill（断电），容器转入停止状态，这种操作容易丢失数据，除非必要，否则不建议使用；

docker restart ： 重启容器，容器转入运行状态； docker pause ： 容器进入暂停状态； docker unpause ： 取消暂停状态，容器进入运行状态；

docker rm ： 删除容器，容器转入删除状态（如果没有保存相应的数据库，则状态不可

见）。

## 1.3 菱形

需要根据实际情况选择的操作

### 1.3.1 killed by out-of-memory（因内存不足被终止）

宿主机内存被耗尽，也被称为 OOM：非计划终止这时需要杀死最吃内存的容器然后进行选择操作

1.3.2 container process exited（异常终止）

### 1.3.3 出现容器被终止后，将进入 Should restart?选择操作：

yes 需要重启，容器执行 start 命令，转为运行状态。

no 不需要重启，容器转为停止状态。

**十、  Docker 的数据管理**

# 1  数据卷管理

数据卷的作用是将宿主机的某个磁盘目录映射到容器的某个目录，从而实现宿主机和容器之间的数据共享。

docker run|create --name [容器名称] -v [宿主机目录]:[容器目录] [镜像名称]

参考新建容器章节或新增并启动容器章节

# 2  数据卷容器管理

数据卷容器的作用是实现多个容器之间的数据共享。其实，数据卷容器也是一个容器，

但是与其他 Docker 容器不一样的是，数据卷容器是专门用来提供数据卷给其他容器进行挂载操作。

## 2.1 创建数据卷容器

创建容器，并在容器中创建目录/datas。

docker create --name my_datas -v /datas eureka:1.0

## 2.2 创建容器并使用数据卷容器

命令格式：

docker run --volumes-from [数据卷容器名或 ID] [options] [镜像名或 ID]

具体如下：

docker run --volumes-from my_datas -tid --name eureka1 -p 8761:8761 eureka:1.0 docker run --volumes-from my_datas -tid --name eureka2 -p 8762:8761 eureka:1.0

## 2.3 测试

### 2.3.1 访问 eureka1 容器并在共享目录中写入数据

docker exec -it eureka1 bash

\> ls /

\> mkdir /datas/test

\> touch /datas/test/readme

\> echo this text from eureka1 >> /datas/test/readme

\> exit

### 2.3.2 访问 eureka2 容器并读取共享目录中的数据

docker exec -it eureka2 bash

\> ls /datas/test

\> cat /datas/test/readme

# 3  数据备份

将某容器中的数据目录备份。此过程需要一个备份容器作为中间过程辅助完成。

**3.1** **创建宿主机备份目录**

mkdir /backup/

## 3.2 通过容器备份数据

命令格式：

docker run --rm --volumes-from [数据卷容器 ID 或名称] -v [宿主机目录]:[容器目录] [镜

像名称] [备份命令] 其中--rm 是自动删除容器参数，代表此容器执行完备份命令后，自动删除。具体如下：

​      docker  run  --rm  --volumes-from  my_datas  -v  /backup/:/backup/  mysql  tar  -cf

/backup/data.tar.gz /data 备份成功后，查看宿主机中/backup 目录备份结果。

# 4  数据还原

将备份的数据文件还原到某容器，这个过程也需要一个容器辅助完成。

## 4.1 删除容器中的数据文件

用于模拟无数据的容器。

docker exec -it my_datas bash

\> rm -rf /datas

## 4.2 通过容器还原备份数据

命令格式：

docker run --rm -itd --volumes-from [数据要恢复到的容器] -v [宿主机备份目录]:[容器

备份目录] [镜像名称] [解压命令] 具体如下：

​      docker run  --rm -itd  --volumes-from my_datas -v /backup/:/backup/ mysql tar -xf

/backup/data.tar.gz -C /

## 4.3 查看还原结果

docker exec -it my_datas bash

\> ls /data/test

\> cat /datas/test/readme