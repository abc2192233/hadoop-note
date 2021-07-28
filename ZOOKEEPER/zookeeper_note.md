## ZooKeeper(A Distributed Coordination Service for Distributed Applications)
ZooKeeper: 分布式应用程序的分布式协调服务

### 设计目标
1. 结构简单，允许分布式进程通过共享的分层命名空间（namespace）相互协调，该命名空间的组织类似于标准文件系统。命名空间由数据寄存器构成，在zookeeper中被称为znodes，类似于文件和目录。同时和典型的文件系统不同，zookeeper的数据保存在内存中，可以保证高吞吐量和低延迟
2. 副本设计，和协调的分布式进程相同，zookeeper本身也是通过一组机器进行复制，只要集群中的可用节点大于一半那么集群就是可用的

![image](frame_img/zkservice.jpg)
<p align="center">zkervice</p>

3. 