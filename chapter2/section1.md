# 数据增量订阅与消费

基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了mysql.

有关数据增量订阅与消费的中间件回顾一下：

![](/binlog_zookeeper.png)

增量订阅和消费模块应当包括binlog日志抓取，binlog日志解析，事件分发过滤（EventSink），存储（EventStore）等主要模块。

如果需要确保HA可以采用Zookeeper保存各个子模块的状态，让整个增量订阅和消费模块实现无状态化，当然作为consumer\(客户端\)的状态也可以保存在zk之中。

整体上通过一个Manager System进行集中管理，分配资源。

Canal

Canal架构图：

![](/canal_mananger.png)

说明：

server代表一个canal运行实例，对应于一个jvm

instance对应于一个数据队列 （1个server对应1..n个instance\)

instance模块：

eventParser \(数据源接入，模拟slave协议和master进行交互，协议解析\)

eventSink \(Parser和Store链接器，进行数据过滤，加工，分发的工作\)

eventStore \(数据存储\)

metaManager \(增量订阅&消费信息管理器\)

说明：一台机器下部署一个canal，一个canal可以运行多个instance\(通过配置destinations等\), 一般情况下一个client连接一个instance（每个instance可以配置standby功能）, 可以多个client连接同一个instance，但是同一时刻只能有一个client消费instance的数据，这个通过zookeeper控制。

