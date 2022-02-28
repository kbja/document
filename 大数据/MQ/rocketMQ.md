# 											Rocket MQ

## 一、部署角色

Producer、Consumer、NameServer、Broker

NameServer无状态，Broker将自身topic路由注册到NameServer，Producer、Consumer自己从NameServer拉取Broker信息。



## 二、储存结构

CommitLog落盘记录消息，MessageQueue记录CommitLog中的offset，消费者消费MessageQueue。

每个CommitLog大小为1G，文件名通过20为数字标识，为1G的偏移量。

搜索消息通过IndexFile，时间+key做检索，查出offset，再去CommitLog查



## 三、高可用

### 消息发送高可用：Broker挂了后，NameServer不会通知客户端，客户端通过故障规避（切换节点），避免发送失败

### 消息储存高可用：通过刷盘、主从同步、读写分离保证

### 消息消费高可用：客户端回复ACK才算消费完成、增加重试队列

### 集群管理高可用：NameServer全部挂了，正在运行的其他节点不受影响



### 刷盘：

同步：没10ms刷一次

异步：写满一页16K，或者10S刷一次

#### 主从复制：

同步：所有slave都写成功，才返回给客户端

异步：master写成功，就返回给客户端。

建议采取***异步刷盘，同步复制***

#### 读写分离

当master积压消息超过内存的40%，会告诉consumer去从服务器消费。



## 四、高性能



## 五、事务消息

简单可以理解为2PC，没有submit时，mq会进行回查

将一阶段的消息放在Half队列，定期对Half队列遍历



## 六、顺序发送/消费

发送时同步发送；

消费时对ConsumerQueue上锁，需避免消息重试导致堆积。
