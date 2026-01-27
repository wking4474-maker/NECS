#### Redis Cluster集群

**简介**

##### 1.集群模式是实际使用最多的模式。

Redis Cluster是社区版推出的Redis分布式集群解决方案，主要解决Redis分布式方面的需求，比如，当遇到单机内存，并发和流量等瓶颈的时候，Redis Cluster能起到很好的的负载均衡的目的。

**为什么使用redis-cluster？**

```xml
为了在大流量访问下提供稳定的业务，集群化时存储的必然形态
未来的发展趋势可定是云计算和大数据的紧密结合
只有分布式架构能满足需求
```

##### 2.集群描述

Redis集群搭建方案：

```xml
(1)、Twitter开发的twemproxy
(2)、豌豆荚开发的codis
(3)、redis观法的redis-cluster
```

Redis集群搭建的方式有很多种，但从redis 3.0 之后变动表呢支持redis-cluster集群，志超需要3（master）+3（Slave）才能简历集群。

Redis Cluster集群几点最小配置6个节点以上（3主3从），其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用。

**Redis Cluster集群特点**

1、所有的redis节点彼此互联（PING-PONG)，内部使用二进制协议优化传输速度和带宽。

2、节点的fail是通过集群中超过半数的节点检测失效时才生效。

3、客户端与redis节点直连，不需要中间proxy层。客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。

4、redis-cluster把所有的物理节点映射到[0-16383]slot上（不一定是平均分配），cluster负责维护

5、redis集群预先分好16384个哈希槽，当需要在redis集群中放置一个key-value时，redis先对key使用crc16算法算出一个结果，然后把结果对16384求余数，这样对每个key都会对应一个编号在0-16383之间的哈希槽，redis会根据节点数量大致均等的将哈希槽映射到不同的节点。

##### 21.3 Redis Cluster集群搭建

集群搭建参考官网：https://redis.io/topic/cluster-tutorial

​	redis集群需要至少三个master节点，我们这里搭建三个master节点，并且给每个master在搭建一个slave节点，总共6个节点，这里用一台机器（可以多台机器部署，修改一下ip地址就可以了）部署6个redis实例，三主三从。搭建集群的步骤如下：

1. 创建Redis节点安装目录,在/opt目录，创建8001-8006   6个文件夹下

   ```shell
   cd /opt
   mkdir 8001 8002 8003 8004 8005 8006
   ```

2. 并将redis-conf分别拷贝到8001-8006文件夹下

   ```xml
   cp /etc/redis/redis.conf /opt/8001
   ```

3. 修改Redis配置文件

   ```c
   # 关闭保护模式 用于公网访问
       protected-mode  no
       port  8001
   # 开启集群模式
       cluster-enabled  yes
       cluster-config-file nodes-8001.config
       cluster-node-timeout  5000
   # 后台启动
       daemonize  yes
       pidfile  /var/run/redis_8001.pid
       logfile  "8001.log"
   # dir /redis/data
   # 此处绑定ip,可以是阿里内网ip和本地ip也可以直接注释掉该项
   # bind 127.0.0.1
   # 用于连接主节点密码
       masterauth redis
   #设置redis密码 各个几点请保持密码一致
       requirepass  redis
   ```

4. 依次复制并修改6个redis.conf

   ```c
   cp /opt/8001/redis.conf /opt/8002   :依次进行复制
   vim /opt/8002/redis.conf :将文件所有端口号替换成对应端口
   #这里需要做6个文件8001-8006
   ```

5. 依次启动6个节点

启动服务端

```xml
redis-server   /opt/8001/redis.conf
redis-server   /opt/8002/redis.conf
redis-server   /opt/8003/redis.conf
redis-server   /opt/8004/redis.conf
redis-server   /opt/8005/redis.conf
redis-server   /opt/8006/redis.conf
```

启动后，可以用PS查看进程：

```xml
ps -ef | grep -i redis
```

6. 创建集群 

   Redis 5版本后 通过redis-cli 客户端命令来创建集群。

   ```xml
redis-cli --cluster  create  -a redis 192.168.101.91:8001 192.168.101.91:8002 192.168.101.91:8003 192.168.101.91:8004 192.168.101.91:8005 192.168.101.91:8006 --cluster-replicas 1
   ```

7. Redis Cluster集群验证
   在某台机器上（或）连接集群的7001端口的几点：
   ```properties
   redis-cli  -h 192.168.101.91  -c -p 8001 -a redis
   #-c 表示连接到集群
   ```

   redis cluster在设计的时候，就考虑了去中心化，去中间件，也就是说集群中的每个节点都是平等的关系，都是对等的，每个几点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。

   基本命令

   **info replication** 通过Cluster Nodes 命令和Cluster Info命令来看看集群的效果

```properties
192.168.101.91:8001> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.101.91,port=8006,state=online,offset=1638,lag=0
master_failover_state:no-failover
master_replid:97b5ba596c25843424437bf8deeebbd70d94d581
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1638
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1638
```

**输入命令cluster nodes**

```properties
192.168.101.91:8001> cluster nodes
34af8322b38bd2e0a7e39e06d4bc63b927758b41 192.168.101.91:8003@18003 master - 0 1769477733000 3 connected 10923-16383
db07834ba56208c1c74d7ab25b6a5237e74fc23a 192.168.101.91:8001@18001 myself,master - 0 0 1 connected 0-5460
b6fcb5a68f96c535eeb96928152445a19a6e645a 192.168.101.91:8004@18004 slave 6d4602bf3bec2f4c2afe5cfc51d1067a6614bb10 0 1769477734000 2 connected
6d4602bf3bec2f4c2afe5cfc51d1067a6614bb10 192.168.101.91:8002@18002 master - 0 1769477735006 2 connected 5461-10922
6ad658ce27c920dc64ef6ca056dba2f131be642d 192.168.101.91:8005@18005 slave 34af8322b38bd2e0a7e39e06d4bc63b927758b41 0 1769477734100 3 connected
7dfdec779936bca659b5f1e39769f8f28f17b6aa 192.168.101.91:8006@18006 slave db07834ba56208c1c74d7ab25b6a5237e74fc23a 0 1769477734402 1 connected

```

每个Redis的节点都有一个ID值，此ID将被此特定redis实例永久使用，以便实例在集群上下文中具有唯一的名称。每个节点都都会记住使用此ID的每个其他节点，而不是通过IP或端口号。IP地址和端口可能会发生变化，但唯一的节点标识符在节点的整个生命周期内都不会改变。我们简单称这个标识符为节点ID。