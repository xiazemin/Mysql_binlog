# 数据库同步Otter

背景：alibaba B2B因为业务的特性，卖家主要集中在国内，买家主要集中在国外，所以衍生出了杭州和美国异地机房的需求，同时为了提升用户体验，整个机房的架构为双A，两边均可写，由此诞生了otter这样一个产品。

otter第一版本可追溯到04~05年，此次外部开源的版本为第4版，开发时间从2011年7月份一直持续到现在，目前阿里巴巴B2B内部的本地/异地机房的同步需求基本全上了otter4。

基于数据库增量日志解析，准实时同步到本地机房或异地机房的mysql/oracle数据库，一个分布式数据库同步系统。

工作原理

![](/otter.png)

原理描述：

基于Canal开源产品，获取数据库增量日志数据。

典型管理系统架构，manager\(Web管理\)+node\(工作节点\)

manager运行时推送同步配置到node节点

node节点将同步状态反馈到manager上

基于zookeeper，解决分布式状态调度的，允许多node节点之间协同工作。

Otter的作用

异构库

mysql-&gt;mysql、oracle. \(目前开原版只支持mysql增量，目标库可以是mysql或者oracle,取决于canal的功能\)

单机房同步（数据库之间RTT\(Round-Trip Time\)&lt;1ms）

数据库版本升级

数据表迁移

异步二级索引

跨机房同步（比如阿里巴巴国际站就是杭州和美国机房的数据库同步，RTT&gt;200ms）

机房容灾

双向同步

避免回环算法（通用的解决方案，支持大部分关系型数据库）

数据一致性算法（保证双A机房模式下，数据保证最终一直性）

文件同步

站点镜像（进行数据复制的同时，复制关联的图片，比如复制产品数据，同事复制产品图片）

单机房复制示意图

![](/otter_syn.png)

说明：

* 数据On-Fly, 尽可能不落地，更快的进行数据同步。（开启node load balance算法, 如果Node节点S+ETL落在不同的Node上，数据会有个网络传输过程）

* node节点可以有failover/loadBalancer.

SETL

S: Select

为解决数据来源的差异性，比如接入canal获取增量数据，也可以接入其他系统获取其他数据等。

E: Extract

T: Transform

L: Load

类似于数据仓库的ETL模型，具体可为数据join，数据转化，数据加载。

跨机房复制示意图

数据涉及网络传输，S/E/T/L几个阶段会分散在2个或者更多Node节点上，多个Node之间通过zookeeper进行协同工作（一般是Select和Extract在一个机房的Node, Transform/Load落在另一个机房的Node）

node节点可以有failover/loadBalancer。\(每个机房的Node节点，都可以是集群，一台或者多台机器\)

More:

Otter调度模型：batch处理+双节点部署。

Otter数据入库算法

Otter双向回环控制

Otter数据一致性

Otter高可用性

Otter扩展性

