# ZooKeeper

# 一、 ZooKeeper 简介

 		顾名思义 zookeeper 就是动物园管理员，他是用来管 hadoop（大象）、Hive(蜜蜂)、pig(小猪)的管理员， Apache Hbase 和 Apache Solr 的分布式集群都用到了 zookeeper；Zookeeper: 是一个分布式的、开源的程序协调服务，是 hadoop 项目下的一个子项目。他提供的主要功能包括：配置管理、名字服务、分布式锁、集群管理。



# 二、 ZooKeeper 的作用

 

### 1.1 配置管理

​		在我们的应用中除了代码外，还有一些就是各种配置。比如数据库连接等。一般我们都是使用配置文件的方式，在代码中引入这些配置文件。当我们只有一种配置，只有一台服务器，并且不经常修改的时候，使用配置文件是一个很好的做法，但是如果我们配置非常多， 有很多服务器都需要这个配置，这时使用配置文件就不是个好主意了。这个时候往往需要寻找一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的都可以获得变更。Zookeeper 就是这种服务，它使用Zab 这种一致性协议来提供一致性。现在有很多开源项目使用 Zookeeper 来维护配置，比如在 HBase 中，客户端就是连接一个Zookeeper，获得必要的 HBase 集群的配置信息，然后才可以进一步操作。还有在开源的消息队列Kafka 中，也使用 Zookeeper 来维护broker 的信息。在 Alibaba 开源的SOA 框架Dubbo 中也广泛的使用Zookeeper 管理一些配置来实现服务治理。

### 1.2 名字服务

 		名字服务这个就很好理解了。比如为了通过网络访问一个系统，我们得知道对方的 IP 地址，但是 IP 地址对人非常不友好，这个时候我们就需要使用域名来访问。但是计算机是不能是域名的。 怎么办呢？如果我们每台机器里都备有一份域名到 IP 地址的映射，这个倒是能解决一部分问题，但是如果域名对应的 IP 发生变化了又该怎么办呢？于是我们有了DNS 这个东西。我们只需要访问一个大家熟知的(known)的点，它就会告诉你这个域名对应的 IP 是什么。在我们的应用中也会存在很多这类问题，特别是在我们的服务特别多的时候， 如果我们在本地保存服务的地址的时候将非常不方便，但是如果我们只需要访问一个大家都熟知的访问点，这里提供统一的入口，那么维护起来将方便得多了。

 ### 1.3 分布式锁

 其实在第一篇文章中已经介绍了 Zookeeper 是一个分布式协调服务。这样我们就可以利用 Zookeeper 来协调多个分布式进程之间的活动。比如在一个分布式环境中，为了提高可靠性，我们的集群的每台服务器上都部署着同样的服务。但是，一件事情如果集群中的每个服务器都进行的话，那相互之间就要协调，编程起来将非常复杂。而如果我们只让一个服务进行操作，那又存在单点。通常还有一种做法就是使用分布式锁，在某个时刻只让一个服务去干活，当这台服务出问题的时候锁释放，立即 fail over 到另外的服务。这在很多分布式系统中都是这么做，这种设计有一个更好听的名字叫 Leader Election(leader 选举)。比如 HBase 的 Master 就是采用这种机制。但要注意的是分布式锁跟同一个进程的锁还是有区别的，所以使用的时候要比同一个进程里的锁更谨慎的使用。

 

### 1.4 集群管理

​		在分布式的集群中，经常会由于各种原因，比如硬件故障，软件故障，网络问题，有些节点会进进出出。有新的节点加入进来，也有老的节点退出集群。这个时候，集群中其他机器需要感知到这种变化，然后根据这种变化做出对应的决策。比如我们是一个分布式存储系统，有一个中央控制节点负责存储的分配，当有新的存储进来的时候我们要根据现在集群目前的状态来分配存储节点。这个时候我们就需要动态感知到集群目前的状态。还有，比如一个分布式的 SOA 架构中，服务是一个集群提供的，当消费者访问某个服务时，就需要采用某种机制发现现在有哪些节点可以提供该服务(这也称之为服务发现，比如 Alibaba 开源的SOA 框架Dubbo 就采用了 Zookeeper 作为服务发现的底层机制)。还有开源的 Kafka 队列就采用了 Zookeeper 作为 Cosnumer 的上下线管理。

 

# 三、 Zookeeper 的存储结构

![](D:/Github/Note-Repository/01_JavaNote/images/clip_image001.jpg)

  

**1**    **Znode**

 

在 Zookeeper 中，znode 是一个跟 Unix 文件系统路径相似的节点，可以往这个节点存储或获取数据。

Zookeeper 底层是一套数据结构。这个存储结构是一个树形结构，其上的每一个节点， 我们称之为“znode”

zookeeper 中的数据是按照“树”结构进行存储的。而且 znode 节点还分为 4 中不同的类型。

每一个 znode 默认能够存储 1MB 的数据（对于记录状态性质的数据来说，够了）

可以使用 zkCli 命令，登录到 zookeeper 上，并通过 ls、create、delete、get、set 等命令操作这些 znode 节点



 

 

**2    Znode 节点类型**

 

(1) PERSISTENT 持久化节点: 所谓持久节点，是指在节点创建后，就一直存在，直到有删除操作来主动清除这个节点。否则不会因为创建该节点的客户端会话失效而消失。

 

(2) PERSISTENT_SEQUENTIAL 持久顺序节点：这类节点的基本特性和上面的节点类型是一致的。额外的特性是，在 ZK 中，每个父节点会为他的第一级子节点维护一份时序， 会记录每个子节点创建的先后顺序。基于这个特性，在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK 会自动为给定节点名加上一个数字后缀，作为新的节点名。这个数字后缀的范围是整型的最大值。在创建节点的时候只需要传入节点 “/test_”，这样之后，zookeeper 自动会给”test_”后面补充数字。

 

(3) EPHEMERAL 临时节点：和持久节点不同的是，临时节点的生命周期和客户端会话绑定。也就是说，如果客户端会话失效，那么这个节点就会自动被清除掉。注意，这里提到的是会话失效，而非连接断开。另外，在临时节点下面不能创建子节点。

这里还要注意一件事，就是当你客户端会话失效后，所产生的节点也不是一下子就消失了，也要过一段时间，大概是 10 秒以内，可以试一下，本机操作生成节点，在服务器端用命令来查看当前的节点数目，你会发现客户端已经 stop，但是产生的节点还在。

 

(4)  EPHEMERAL_SEQUENTIAL 临时自动编号节点：此节点是属于临时节点，不过带有顺序，客户端会话结束节点就消失。

 

# 四、 安装 zookeeper

 

### **1**    **安装单机版**

 

**1.1** **安装** **Linux**

 

**1.2** **安装** **JDK**

 

配置环境变量

  

```
export  JAVA_HOME=/usr/local/jdk  
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
export PATH=$JAVA_HOME/bin:$PATH  
```

 

**1.3 上传 Zookeeper**

 

官方资源包可在 zookeeper.apache.com 站点中下载。最新发布版本为：3.4.12。



 

**1.4 解压 Zookeeper 压缩包**

 

 

```
 [root@localhost  temp]# tar -zxf zookeeper-3.4.6.tar.gz  
 [root@localhost  temp]# cp zookeeper-3.4.6 /usr/local/zookeeper -r  
```

 

**1.5** **Zookeeper** **目录结构**

 

    1. bin：放置运行脚本和工具脚本，如果是 Linux 环境还会有有 zookeeper 的运行日志 zookeeper.out  
       2. conf：zookeeper 默认读取配置的目录，里面会有默认的配置文件 
          3. contrib：zookeeper 的拓展功能  
             4. dist-maven：zookeeper 的maven 打包目录  
                5. docs：zookeeper 相关的文档 
                   6. lib：zookeeper 核心的 jar  
                      7. recipes：zookeeper 分布式相关的jar 包 
                         8.  src：zookeeper 源码  

 






















**1.6** **配置** **Zookeeper**

 

Zookeeper 在启动时默认的去 conf 目录下查找一个名称为 zoo.cfg 的配置文件。在 zookeeper 应用目录中有子目录 conf。其中有配置文件模板：zoo_sample.cfg 

cp zoo_sample.cfg zoo.cfg。zookeeper 应用中的配置文件为 conf/zoo.cfg。

修改配置文件 zoo.cfg - 设置数据缓存路径



 

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image003-1588661007670.jpg)

 1.7 启动 Zookeeper



默认加载配置文件：./zkServer.sh start：默认的会去 conf 目录下加载 zoo.cfg 配置文件。

指定加载配置文件：./zkServer.sh start 配置文件的路径。

 1.8停止 Zookeeper 

./zkServer.sh stop 

1.9查看 Zookeeper 状态

 ./zkServer.sh status 

1.10使用客户端连接单机版 Zookeeper 

1.10.1 连接方式一

 bin/zkCli.sh 默认连接地址为本机地址，默认连接端口为 2181 6 

1.10.2 连接方式二 

bin/zkCli.sh -server ip:port 连接指定 IP 地址与端口

### 2    安装集群版

 

**2.1** **Zookeeper** **集群中的角色**

 Zookeeper 集群中的角色主要有以下三类

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image005-1588661007671.jpg)

 

![](D:/Github/Note-Repository/01_JavaNote/images/clip_image006-1588661007671.jpg)



 

**2.2 设计目的**

 

1.最终一致性：client 不论连接到哪个Server，展示给它都是同一个视图，这是 zookeeper最重要的性能。

2  .可靠性：具有简单、健壮、良好的性能，如果消息 m 被到一台服务器接受，那么它将被所有的服务器接受。

3 .实时性：Zookeeper 保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，Zookeeper 不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用 sync()接口。

4 .等待无关（wait-free）：慢的或者失效的 client 不得干预快速的client 的请求，使得每个 client 都能有效的等待。

5.原子性：更新只能成功或者失败，没有中间状态。

6 .顺序性：包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息 a 在消息b 前发布，则在所有 Server 上消息 a 都将在消息b 前被发布；偏序是指如果一个消息 b 在消息 a 后被同一个发送者发布，a 必将排在b 前面。



 

**2.3 集群安装**

 

使用 3 个 Zookeeper 应用搭建一个伪集群。应用部署位置是：192.168.70.143。服务监听端口分别为：2181、2182、2183。投票选举端口分别为 2881/3881、2882/3882、2883/3883。

tar -zxf zookeeper-3.4.6.tar.gz

将解压后的 Zookeeper 应用目录重命名，便于管理

mv zookeeper-3.4.12 zookeeper01

 

**2.3.1 提供数据缓存目录**

 

在 zookeeper01 应用目录中，创建 data 目录，用于缓存应用运行数据

cd zookeeper01 mkdir data

 

**2.3.2 复制应用**

 

复制两份 Zookeeper 应用。用于模拟集群中的 3 个节点。

cp -r zookeeper01 zookeeper02 

cp -r zookeeper01 zookeeper03

 

**2.3.3 提供配置文件**

 

在 zookeeper 应用目录中有子目录 conf。其中有配置文件模板：zoo_sample.cfg 

cp zoo_sample.cfg zoo.cfg

zookeeper 应用中的配置文件为 conf/zoo.cfg。

 

**2.3.4 修改配置文件 zoo.cfg - 设置数据缓存路径**

 

dataDir 参数值为应用运行缓存数据保存目录。是 3.2.3 步骤中创建的目录。使用绝对路径赋值。不同的应用路径不同。

 

**2.3.5 提供应用唯一标识**

在 Zookeeper 集群中，每个节点需要一个唯一标识。这个唯一标识要求是自然数。且唯一标识保存位置是录(dataDir=/usr/local/zookeeper/data)的 myid 文件中。：$dataDir/myid。其中 dataDir 为配置文件 zoo.cfg 中的配置参数

在 data 目录中创建文件 myid ： touch myid

为应用提供唯一标识。本环境中使用 1、2、3 作为每个节点的唯一标识。

vi myid

简化方式为： echo [唯一标识] >> myid。 echo 命令为回声命令，系统会将命令发送的数据返回。 '>>'为定位，代表系统回声数据指定发送到什么位置。 此命令代表系统回声数据发送到 myid 文件中。 如果没有文件则创建文件。

echo 1 >> zookeeper01/data/myid

 ![image-20200602080617763](D:/Github/Note-Repository/01_JavaNote/images/image-20200602080617763.png)

**2.3.6 修改配置文件 zoo.cfg - 设置服务、投票、选举端口**

 

vi zoo.cfg

 每个配置文件都需要修改

```
 clientPort=2181  #服务端口根据应用做对应修改,zk01-2181,zk02-2182,zk03-2183  
server.1=192.168.0.105:2881:3881  

server.2=192.168.0.105:2882:3882  

server.3=192.168.0.105:2883:3883  
```

 

 

**2.3.7 启动 ZooKeeper 应用**

 

```
zookeeper01/bin/zkServer.sh start
zookeeper02/bin/zkServer.sh start
zookeeper03/bin/zkServer.sh start
```

bin/zkServer.sh start

ZooKeeper 集群搭建后，至少需要启动两个应用才能提供服务。因需要选举出主服务节点。启动所有 ZooKeeper 节点后，可使用命令 bin/zkServer.sh status 来查看节点状态。如下：

Mode: leader   - 主机


Mode: follower   - 备份机

![](D:/Github/Note-Repository/01_JavaNote/images/clip_image008-1588661007671.jpg)

 

**2.3.8 关闭 ZooKeeper 应用**

 

```
zookeeper01/bin/zkServer.sh stop
zookeeper02/bin/zkServer.sh stop
zookeeper03/bin/zkServer.sh stop
```

bin/zkServer.sh stop 命令为关闭 ZooKeeper 应用的命令。

 

**2.3.9 控制台访问 ZooKeeper 应用**

 

bin/zkCli.sh -server 192.168.199.175:2181

命令格式为： zkCli.sh -server host:port。默认连接 localhost:2181。



# 五、  Zookeeper 常用命令

### 1  ls 命令

ls /path

使用 ls 命令查看 zookeeper 中的内容。在 ZooKeeper 控制台客户端中，没有默认列表功能，必须指定要列表资源的位置。 如： ls / 或者 ls /path

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image002-1591057833963.jpg)

### 2  create 命令

create [-e] [-s] /path [data] 默认不带参数为持久化节点

e 临时节点

s 序列

使用 create 命令创建一个新的 Znode。

*create [-e] [-s] path data* - 创建节点，如： create /test 123 创建一个/test 节点，节点携带数据信息 123。 

create -e /test 123 创建一个临时节点/test，携带数据为 123，临时节点只在当前会话生命周期中有效，会话结束节点自动删除。

create -s /test 123 创建一个顺序节点/test，携带数据 123，创建的顺序节点由 ZooKeeper 自动为节点增加后缀信息，如-/test00000001 等。

-e 和-s 参数可以联合使用。

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image004-1591057833964.jpg)

### 3  get 命令

get [-s] /path

get 命令获取 Znode 中的数据。

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image006-1591057833964.jpg)

get -s /path

-s 查看 Znode 详细信息

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image008-1591057833964.jpg)

oldlu:存放的数据 

cZxid:创建时 zxid(znode 每次改变时递增的事务 id) 

ctime:创建时间戳 mZxid:最近一次更近的 zxid 

mtime:最近一次更新的时间戳

pZxid:子节点的 zxid cversion:子节点更新次数

dataversion:节点数据更新次数 

aclVersion:节点 ACL(授权信息)的更新次数 

ephemeralOwner:如果该节点为 ephemeral 节点(临时，生命周期与 session 一样),ephemeralOwner 值表示与该节点绑定的 session id. 如果该节点不是ephemeral 节点, ephemeralOwner 值为 0.

dataLength:节点数据字节数

numChildren:子节点数量

### 4  set 命令

set /path [data]

添加或修改 Znode 中的值

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image010-1591057833964.jpg)

### 5  delete 命令

delete /path 删除 Znode。

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image012-1591057833964.jpg)



# 六、 使用 Java API 操作 Zookeeper

## **1**  **创建** **Znode**

### 1.1创建项目

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image014-1591057833964.jpg)

### 1.2修改 POM 文件添加依赖

该依赖为基于 Java 语言连接 Zookeeper 的客户端工具两端版本要保持一致

```xml
<dependency>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
<version>3.6.0</version>
</dependency>
```



### 1.3创建 Znode 并添加数据

```java
/**
* 操作 Zookeeper 的 Znode
*/
public class ZnodeDemo implements Watcher {
public static void main(String[] args) throws KeeperException,InterruptedException, IOException {
    //创建连接 Zookeeper 对象
    ZooKeeper zooKeeper = new
    ZooKeeper("192.168.233.130:2181,192.168.233.130:2182,192.168.233.130:2183",150000,new ZnodeDemo());
    //创建一个 Znode
    String path =zooKeeper.create("/bjsxt/test","oldlu".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
    System.out.println(path);
    }
/**
* 事件通知回调方法
* @param event
*/
    @Override
    public void process(WatchedEvent event) {
    	//获取连接事件
        if(event.getState() == Event.KeeperState.SyncConnected){
        System.out.println("连接成功");
        }
    }
}
```

 

## **2**  **获取** **Znode** **中的数据**

### 2.1获取指定节点中的数据

```java
byte[] data= zooKeeper.getData("/bjsxt/test0000000001",new ZnodeDemo(),new Stat());
System.out.println(new String(data));
```



### 2.2获取所有子节点中的数据

```java
List<String> list = zooKeeper.getChildren("/bjsxt",new ZnodeDemo());
for(String path :list){
byte[] data= zooKeeper.getData("/bjsxt/"+path,new ZnodeDemo(),null);
System.out.println(new String(data));
}
```



## 3  设置 Znode 中的值

```java
//设置 Znode 中的值
Stat stat =
zooKeeper.setData("/bjsxt/test0000000001","bjsxt".getBytes(),-1);
System.out.println(stat);
```



## **4**  **删除** **Znode**

```java
zooKeeper.delete("/bjsxt/test0000000001",-1);
```



# 七、  Zookeeper 实战

实战案例介绍：使用 Zookeeper 与 RMI 技术实现一个 RPC 框架。

RPC：RPC（Remote Procedure Call）远程过程调用。

**1**  **基于** **RMI** **实现远程方法调用**

### 1.1RMI 简 介

RMI(Remote Method Invocation) 远程方法调用。

RMI 是从 JDK1.2 推出的功能，它可以实现在一个 Java 应用中可以像调用本地方法一样调用另一个服务器中 Java 应用（JVM）中的内容。

RMI 是 Java 语言的远程调用，无法实现跨语言。

### 1.2执行流程

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image016-1591057833964.jpg)

Registry(注册表)是放置所有服务器对象的命名空间。 每次服务端创建一个对象时，它都会使用 bind()或 rebind()方法注册该对象。 这些是使用称为绑定名称的唯一名称注册的。

要调用远程对象，客户端需要该对象的引用。即通过服务端绑定的名称从注册表中获取对象(lookup()方法)。

### **1.3RMI** **的** **API** **介绍** 

**1.3.1    Remote** **接口** 

java.rmi.Remote 定义了此接口为远程调用接口。如果接口被外部调用，需要继承此接口。

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image018-1591057833964.jpg)

**1.3.2  RemoteException 类**

java.rmi.RemoteException 继承了 Remote 接口的接口，如果方法是允许被远程调用的，需要抛出此异常。

**1.3.3  UnicastRemoteObject 类**

java.rmi.server.UnicastRemoteObject 此类实现了 Remote 接口和 Serializable 接口。

自定义接口实现类除了实现自定义接口还需要继承此类。

**1.3.4  LocateRegistry 类**

java.rmi.registry.LocateRegistry

可以通过 LocateRegistry 在本机上创建 Registry，通过特定的端口就可以访问这个Registry。

**1.3.5  Naming 类**

java.rmi.Naming

Naming 定义了发布内容可访问 RMI 名称。也是通过 Naming 获取到指定的远程方法。

### 1.4创建 Server 端--实例化、注册、绑定

####      1.4.1   创建项目

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image020-1591057833964.jpg)

####      1.4.2   创建接口

  

```java
/**
* 定义允许远程调用接口，该接口必须要实现 Remote 接口
* 允许被远程调用的方法必须要抛出 RemoteException
*/
public interface DemoService extends Remote {
	String demo(String str) throws RemoteException;
}
```



####      1.4.3   创建接口实现类

 

```java
/**
* 接口实现类必须要继承 UnicastRemoteObject。
* 会自动添加构造方法，需要修改为 public
*/
public class DemoServiceImpl extends UnicastRemoteObject implements DemoService {
    //protect类型要修改为public
    public DemoServiceImpl() throws RemoteException {
    }
    @Override
    public String demo(String str) throws RemoteException {
    	return "Hello RMI "+str;
    }
}
```



#### 1.4.4   编写主方法

 

```java
public class DemoServer {
public static void main(String[] args) throws RemoteException,AlreadyBoundException, MalformedURLException {
    //将对象实例化
    DemoService demoService = new DemoServiceImpl();
    //创建本地注册表
    LocateRegistry.createRegistry(8888);
    //将对象绑定到注册表中
    Naming.bind("rmi://localhost:8888/demoService",demoService);
    }
}
```



### 1.5创建 Client 端--lookup、执行方法

####      1.5.1   创建项目

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image022-1591057833964.jpg)

####      1.5.2   复制服务端接口

```java
/**
* 定义允许远程调用接口，该接口必须要实现 Remote 接口
* 允许被远程调用的方法必须要抛出 RemoteException
*/
public interface DemoService extends Remote {
String demo(String str) throws RemoteException;
}
```



####      1.5.3   创建主方法

```java
public class ClientDemo {
public static void main(String[] args) throws RemoteException,NotBoundException, MalformedURLException {
    DemoService demoService = (DemoService)
        Naming.lookup("rmi://localhost:8888/demoService");
    String result = demoService.demo("bjsxt");
    System.out.println(result);
	}
}
```





## **2**  **使用** **Zookeeper** **作为注册中心实现** **RPC**

### 2.1创建服务端

**2.1.1   创建项目**

![img](D:/Github/Note-Repository/01_JavaNote/images/clip_image024-1591057833965.jpg)

**2.1.2  修改 POM 文件添加依赖**

```xml
<dependency>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
<version>3.6.0</version>
</dependency>
```

**2.1.3   创建接口**

```java
public interface UsersService extends Remote {
String findUsers(String str) throws RemoteException;
}
```

**2.1.4   创建接口实现类**

  

```java
public class UsersServiceImpl extends UnicastRemoteObject implements UsersService {
    public UsersServiceImpl() throws RemoteException {
    }
    @Override
    public String findUsers(String str) throws RemoteException {
    return "Hello Zookeeper "+str;
    }
}
```

**2.1.5   编写主方法**

  

```java
public class ServerDemo implements Watcher {
public static void main(String[] args) throws IOException,AlreadyBoundException, KeeperException, InterruptedException {
    UsersService usersService = new UsersServiceImpl();
    LocateRegistry.createRegistry(8888);
    String url ="rmi://localhost:8888/user";
    Naming.bind(url,usersService);
    //将 url 信息放到 zookeeper 的节点中
    ZooKeeper zooKeeper = new ZooKeeper("192.168.233.130:2181,192.168.233.130:2182,192.168.233.
    130:2183",150000,new ServerDemo());
    //创建 Znode
    zooKeeper.create("/bjsxt/service",url.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    System.out.println("服务发布成功");
    }
    @Override
    public void process(WatchedEvent event) {
    	//获取连接事件
        if(event.getState() == Event.KeeperState.SyncConnected){
        System.out.println("连接成功");
        }
    }
}                                        
```



### 2.2创建客户端

**2.2.1**   **创建项目**



**2.2.2  修改 POM 文件添加依赖**

```xml
<dependency>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
<version>3.6.0</version>
</dependency>
```

**2.2.3   创建接口**

```java 
public interface UsersService {
String findUsers(String str);
}
```

**2.2.4   编写主方法**

 

```java 
public class ClientDemo implements Watcher {
public static void main(String[] args) throws IOException,KeeperException, InterruptedException, NotBoundException {
    ZooKeeper zooKeeper = new
    ZooKeeper("192.168.233.130:2181,192.168.233.130:2182,192.168.233.130:2183",150000,new ClientDemo());
    byte[] bytes = zooKeeper.getData("/bjsxt/service",new ClientDemo(),null);
    String url = new String(bytes);
    UsersService usersService = (UsersService)Naming.lookup(url);
    String result = usersService.findUsers("Bjsxt");
    System.out.println(result);
    }
    @Override
    public void process(WatchedEvent event) {
        if(event.getState() == Event.KeeperState.SyncConnected){
        System.out.println("连接成功");
        }
    }
}
```

 