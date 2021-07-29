## ZooKeeper(A Distributed Coordination Service for Distributed Applications)
ZooKeeper: 分布式应用程序的分布式协调服务

### 设计目标
1. 结构简单，允许分布式进程通过共享的分层命名空间（namespace）相互协调，该命名空间的组织类似于标准文件系统。命名空间由数据寄存器构成，在zookeeper中被称为znodes，类似于文件和目录。同时和典型的文件系统不同，zookeeper的数据保存在内存中，可以保证高吞吐量和低延迟
2. 副本设计，和协调的分布式进程相同，zookeeper本身也是通过一组机器进行复制，只要集群中的可用节点大于一半那么集群就是可用的

![image](frame_img/zkservice.jpg)
<p align="center">zkervice</p>

3. 提供服务，zookeeper对事物顺序标记并且记录每个更新，后续才做可以使用顺序标记来实现更高级别的抽象操作，如同步原语
4. 高速调度，在“读取主导”的工作任务中性能表现会更好，支持在数千台机器运行，并且读取比写入效率更高，比率约为10:1

### 数据结构和namespace分层
zookeeper的namespace类似标准文件系统的命名空间，逻辑层次由（/）分隔的一系列路径元素
![image](frame_img/zknamespace.jpg)
<p align="center">zknamespace</p>

### 节点和临时节点
与标准文件系统不同，zookeeper的namespace中每个节点都拥有与其关联的数据以及子节点，类似于文件也可以成为目录的文件系统，通常这些数据节点被称为znode，znode所维护的数据结构包括数据的更改、ACL更改和时间戳的版本号，允许缓存验证和协调更新，每个znode的数据更改时，版本号都会增加
存储在namespace的znode的数据都是原子读写的。每个节点都有一个访问控制列表（ACL），限制了谁可以做什么，同时zookeeper也有临时节点的定义，只要创建的znode处于活动状态，这些znode就存在，当会话结束时znode就被删除

### 限制性更新条件和监控
zookeeper支持watch概念，客户端可以对znode进行watch监控，当znode发生变化时，会触发并移除watch，当watch被触发时，客户端会接收一个数据包声明znode发生了变化，如果客户端和zookeeper服务器某个节点连接中断，客户端本地将会受到通知

### 可靠性
zookeeper为保证构建复杂服务的基础，提供了一组保证：
1. 顺序性一致，保证客户端发出的指令顺序与znode指令执行顺序一致
2. 原子性，当指令被执行时，只存在成功和失败两种结果，不存在局部成功
3. 单一系统映像，客户端在任一节点所看到的服务视图都是一致的，客户端不会看到旧的历史视图
4. 可靠性，应用更新后，会持续直到有新的更新覆盖
5. 实时性，保证客户端看到的系统映像在一定时间内是最新的
   
### 简单的API接口
zookeeper的设计初衷在于提供一个简单的编程接口，所以目前仅支持以下操作：
1. create：在树中的某个位置创建一个节点
2. delete：删除某个节点
3. exist：测试节点是否存在
4. get data：从znode获取数据
5. set data：写入数据到znode
6. get children：获取节点的所有子节点列表
7. sync：同步数据

### 实现

![image](frame_img/zkcomponents.jpg)
<p align="center">zkcomponents</p>

组成zookeeper服务的每个服务器都复制每个组件自己的副本，复制的数据块包含整个数据树的内存数据库，更新会记录到磁盘以保证可复现性，并且写入会在应用到内存数据库之前执行。作为协议的一部分，所有来自客户端的请求都会被转发到leader节点，其余为follow节点，接收来自leader的消息并传递保证一致性，消息传递层负责在失败时替换leader节点并保证follow与leader节点一致。
zookeeper使用自定义的原子消息传递协议，由于消息传递层是原子的，所以zookeeper可以保证本地副本一致

### 用途
同步原语，用户组权限，所有权等

