简单的读书笔记，主要参考etcd技术内幕第二章：

[etcd技术内幕](https://github.com/copbint/blogs/blob/main/99_books/books.md#etcd%E6%8A%80%E6%9C%AF%E5%86%85%E5%B9%95)

[分布式一致性算法-Paxos、Raft、ZAB、Gossip](https://zhuanlan.zhihu.com/p/130332285)  
[寻找一种易于理解的一致性算法（扩展版）](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)
[raft算法详解](https://zhuanlan.zhihu.com/p/32052223)


# raft协议要解决什么问题

raft协议是为了解决集群多节点之间数据保持一致的问题。

为什么需要多节点，是因为数据不能只存到单个节点上，否则容易出现单点故障的问题。

# raft的工作原理

主要可以分为几个模块：

leader选举

日志同步

下面分别介绍

## leader选举

### 正常选举流程

节点有3种状态：follower，candidate，leader。

所有节点初始化都为follower，term为0，并且有一个election timer，election timer的时间是在一个设置的范围内随机的值，这样能够尽可能使多个节点不同时超时。

当某个节点的election timer超时之后，便认为此term中的leader出现了故障，需要重新选举，所以便转变为candidate，进入下一个term，变为term1，投给自己一票，然后向集群中的其他节点发送选举请示，其他节点都还处于term 0，在term 1还没投过票，所以都将票投给了这个节点，这个节点就成为了leader。

follower在投票的同时，会重置自己的election timer。以防多个节点同时成为candidate

leader还有一个心跳超时器，定时向所有follower发送心跳，follower收到心跳知道leader存活，便重置election timer。所以只要leader一直正常发心跳，所有的follower就会一直当follower。

### 多个节点election timer同时到期

那么多个节点会同时转变为candidate，如果在这轮选举中，某个节点获得了超过半数节点的票，那么这个节点成为了leader，其他candidate自动降为follower。

如果没有一个节点获得超过半数节点的选票，那么这次选举就是失败的。

election timer最早过期的节点，会将term增加1，开始下一个term的选举。由于election timer的时间值是某个范围内的随机值，所以这种冲突的概率并不大。

### Leader 节点宕机之后重新选举的场景  

leader节点宕机之后，follower节点无法接收到心跳重置election timer，election timer最先超时的节点，将term加1，发起新一轮的选举。

接下来的流程和正常选举流程一样。虽然leader崩溃，但是只要集群中超过半数的节点依然正常工作，选举便能够正常进行，集群也就能正常工作。

当宕机的节点恢复之后，接收到新的leader节点的心跳包，发现term已经变大了，便万变follower继续工作。



## 日志同步

在集群中，对数据的更改都是以日志的形式保存，并且日志是编号有序的，便于在多节点之间进行同步。

在集群中，只有leader负责处理请求，follower收到请示，可以转发给leader处理，也可以重定向客户端到master节点。

leader收到请求之后，会将客户端的更新操作以Append Entries 消息的形式发送到所有follower节点，当收到超过半数的节点的回复之后，leader会提交客户端的更新操作，并通知follower节点，此操作可以提交。同时会对客户端进行应答。（先提交还是先响应客户端？如果先响应，没有真提交上不就不一致了？）

由于日志是编号且有序的，那么就可以保证每个follower都接收到了连续的消息，中间不会存在漏消息的可能。因为如果真漏了消息，follower会响应错误，leader会重新从follower需要的数据开始发送。所以leader要维护每个follower的消息接收状态。

### 既然每次更新操作都只需要同步到半数节点，怎么保证新选举的leader拥有最新的数据?

follower在投票时会有一个判断，判断candidate的最新日志是否比自己本地的新或相等，如果不满足要求则拒绝投票，由于一定要超过半数的节点投票才行，这就保证了一定是拥有最新数据的节点才可能成为leader。



## 网络分区

在一个集群中，如果有部分节点的网络发生故障，与集群中另 一部分节点的连接中断，就会出现网络分区。

如果leader所在分区有超过半数节点，那么无需要重新选举，集群正常工作。不到半数节点的分区，由于无法获得超过半数结点的选票，所以无法选出leader。

如果leader所在分区不超过半数节点，那么leader节点由于无法联系到超过半数的节点，所有更新操作都无法执行。其余节点所在分区将选举出新的leader并对外提供服务。待分区恢复后，原leader节点的term值较小，将会自动降为follower，并从新的leader处同步数据。



# QA

## 为什么一般集群是奇数个节点

因为raft中定义，超过半数节点可以正常工作的话，集群就还能够正常工作。

那么：

3个节点能够容忍1个节点失效

4个节点还是只能够容忍1个节点失效（超过半数节点，所以必须有3个）

5个节点能够容忍2个节点失效

6个节点还是只能够容忍2个节点失效（超过半数节点，所以必须有4个）

所以，一般集群是奇数个节点，就是因为奇数个节点更划算



