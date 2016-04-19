如何构建一个每天数十亿次请求级别的web应用？

印度最大电商公司Snapdeal介绍了其Snapdeal Ads系统支持每天5B请求的经验分享。

Snapdeal是一家类似于京东和阿里巴巴结合体的电商平台。独立商户可以借助这个平台销售高质量的商品，在Snapdeal出售的商品均为全新，并且支持七天免费退换。商家进驻Snapdeal后，随后的事宜（交易、包装和物流）都将由Snapdeal完成，也就是商家都将成为Snapdeal的“供货商”，无需与用户直接进行交易。

对于只有不到10个工程师的团队构建一个可伸缩的大型Web系统(web-scale)是困难的，使用正确的技术也许比你的团队成员数量多少更加重要。

关键战略：

```
1. 从水平和垂直两个方面扩展
2.CAP定理中选择可用性和分区容错性(AP)，而不是一致性和可用性组合(CA)。因为初始目标是需要一个低延迟 高性能的拍卖服务平台。
3.没有厂商锁定保护或因为专利限制使用的情况，开源软件以前达到毫无疑问的稳定和易用程度，且低费用。因此决定不再使用软件供应厂商的专有软件。
4.基于机器同情Mechanical Sympathy法则建立系统，软件建立在深刻理解硬件工作机理上，通过软件最大发挥硬件潜能。
5.云技术的限制使用，因为亚马逊EC2比较昂贵，其次是网络不确定和磁盘虚拟化会提高延迟时间。
6.如果延迟存在就必须处理它，再试图消除它，所有的查询应该限制在1ms以下，使用RocksDB和各种其他解决方案作为初始缓存/嵌入式数据库。
7.尽可能使用SSD，也是为了降低延迟。
8.不虚拟化硬件，利用大规模硬件优点（256GB RAM, 24 core）并行化很多计算。
9.磁盘写操作，如果可能进行计时然后每隔几秒将一串数据flush写到到磁盘。
10.Nginx微调到支持keep-alive连接，Netty优化到支持大量并发负载支持模型。
11.关键数据对于广告服务器总是立即可用（微妙级），所有数据都是存储在内存in-memory的库或数据结构中。
12.架构应该总是share nothing，至少广告服务器和外部拍卖系统应该是share nothing，当我们拔掉广告服务器时，整个系统都不会眨眼受到影响。
13.所有关键数据结果必须是可复制的。
14.保持几天的原始记录备份。
15.如果数据有点过时和系统不一致，没有关系。
16.消息系统应该是失败容错，可以崩溃但是不能丢失数据。
```
当前基础设施：
1.跨3个数据中心的40–50节点。
2.其中是30台用于高计算(128–256G RAM, 24 cores, 当前顶级CPU，尽可能SSD)
3.其余小于32G RAM, Quadcore机器.
4.10G私有网络 + 10G 公共网络
5.小型 Cassandra, Hbase 和 Spark 集群.

关键性需求：
1.系统支持多个拍卖者发送基于HTTP(REST端口)的RTB 2.0请求。
2.系统应当能在拍卖中推出Yes/No 价格与广告的响应。
3.系统应当能处理每天数亿的事件，响应几百上千的QPS。
4.数据应该尽可能被处理，至少关键点是这样。

```
使用的关键技术：
1.HBase和Cassandra用于计数据和和管理用户或账户的传统数据集，选择HBase是因为高写入性能，能够几乎实时处理计数。
2.后端主要语言是Java，尽管过去有C++和Erlang经验，Java有成熟的应用技能，JVM也相当成熟。
3.Google Protobuf 用于数据传输
4.Netty作为后端主要服务器，简单高性能。
5.RocksDB作为用户资料读写服务，它是嵌入式数据库，使用Apache Kafka能够跨RocksDB同步数据。
6.Kafka是用于消息队列，流化数据处理
7.CQEngine用于主要的内存in-memory快速查询。
8.Nginx是主要的反向代理
9.Apache Spark是用户ML处理
10 Jenkins用于CI
11.Nagio和Newrelic 监视服务器
12.Zookeeper用于分布式同步
13.Dozens of third parties for audience segments, etc.
14.Bittorrent Sync用于同步跨节点和数据中心的关键数据
15.ustom built quota manger based on Yahoo white paper for budget control.
```

系统设计与结果：
ad服务器是使用简单非堵塞的netty构建，处理每个进来的HTTP请求，在内存的很多存储中寻找一个活动进行展示，这是使用CQ Engine查询，这种查询并不引发任何网络延迟，计算时间或堵塞过程比如磁盘写，将会整个在内存中运行，所有计算会发生在节点内存中，几乎是in process。

ad服务器和其他系统没有分享，共同组件是通过异步通讯。

ad服务器以5-15ms延迟的高性能传递结果，原始数据异步写入到Kafka处理。

原始数据被Hbase中多个Java过程消费，预算和活动状态在Cassandra集群中更新。

一些原始数据发往spark集群用于adhoc处理。

总结:

```
使用的关键技术：
1.HBase和Cassandra用于计数据和和管理用户或账户的传统数据集，选择HBase是因为高写入性能，能够几乎实时处理计数。
2.后端主要语言是Java，尽管过去有C++和Erlang经验，Java有成熟的应用技能，JVM也相当成熟。
3.Google Protobuf 用于数据传输
4.Netty作为后端主要服务器，简单高性能。
5.RocksDB作为用户资料读写服务，它是嵌入式数据库，使用Apache Kafka能够跨RocksDB同步数据。
6.Kafka是用于消息队列，流化数据处理
7.CQEngine用于主要的内存in-memory快速查询。
8.Nginx是主要的反向代理
9.Apache Spark是用户ML处理
10 Jenkins用于CI
11.Nagio和Newrelic 监视服务器
12.Zookeeper用于分布式同步
13.Dozens of third parties for audience segments, etc.
14.Bittorrent Sync用于同步跨节点和数据中心的关键数据
15.ustom built quota manger based on Yahoo white paper for budget control.

```