## 													ZAB协议

### 一、名词解释

1、**zxid**：Zookeeper Transaction id，zk全局事务id。是一个64位整型，由两部分组成，高32位为epoch，低32位为从0开始的递增数字。

2、**epoch**：zk的leader周期，每选举出一个新的leader，epoch+1。

3、**角色**：Leader：用于协调其他节点的写请求，进行广播

​				 Observer：处理客户端的读请求，将写请求转发给Leader

​				 Follower：在Observer基础上，还参与Leader选举。

### 二、简介

ZAB协议ZookeeperAtomicBroadcastProtocol，Zookeeper通过ZAB协议来保障数据一致性。在ZAB协议中，存在两种状态，第一种为故障恢复状态，第二种为消息广播状态。当leader收不到超过半数的Follower的心跳时，会广播进入故障恢复状态。当新的Leader选举完成，和超过半数Follower通信成功，并完成数据同步，之后进入消息广播状态。



### 三、具体步骤

故障恢复状态可以细分为**发现**和**同步**两个小状态。

#### 发现：

1、<font color="red">选举出准leader</font>

2、所有follower将自己最后处理的zxid的epoch发送给准leader，当准leader收到超过半数请求后，选出其中最大的epoch，对其+1，并将结果广播给所有follower。

3、follower收到广播后，将自身的epoch更新为新的epoch，并发送给准leader一个【最后一个历史处理事务epoch，历史处理事务集合】做为回复消息。

4、当准leader收到超过半数的follower回复，选一个follower的历史处理事务集合作为初始化集合。选取条件为不存在任意一个follower的历史处理事务id大于选中follower的历史处理事务id。

***实际处理逻辑是先对比请求中的epoch，再对比zxid，应该是为了优化对比。***



#### 同步：

1、准leader将 ***发现*** 中选中的的【epoch，历史处理事务集合】发送给所有follower作为初始化数据，如果follower的epoch‘ 小于请求中的epoch（说明follow没有参与发现过程），就直接跳过等待下轮同步；否则follower将历史处理事务集合应用到自身，并返回给准leader一个回复，表明自身以处理事务集合。

2、准leader收到半数follower的回复后，向所有follower发送commit请求，要求follower提交事务。

**至此，完成同步过程，准leader成为leader，开始广播**

#### 广播：

1、leader将收到的所有消息，按照生成的zxid顺序进行排序，并以此发送给follower。follower收到后，将其加到历史处理事务集合中，并回复leader处理成功。

2、收到半数以上follower的回复后，leader发送commit请求，要求所有follower提交事务。follower收到后就会开始提交。



#### leader选举：

vote ： 【sid,zxid,version】集合

sid：服务器id，在myid里面

zxid：事务id

Version: 选举版本

有两种情况，1是一台新的zk服务器加入到集群中，2是所有服务器都处于初始化状态。

1、新的zk服务器会处于发现状态，向其他服务器发送投票；其他服务器会把当前的leader信息返回给新加入的服务器，使得新加入的服务器变更为广播状态

2、每台Zookeeper服务器在开始的时候都处于looking状态，将第一张选票投给自己，并将投票结果发送给其他服务器。当当前服务器收到别的服务器的投票结果时，会将投票结果和自身投票对比。

- 当自身version和选票version相同时，先对比zxid，再对比sid，优先选取大的；

- 当自身version大于选票version时，忽略；

- 当自身version小于选票version时，清空自身选票，将自己的sid和zxid与选票进行对比。

当超过半数服务器达成一致时，完成选举。