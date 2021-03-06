# 数据库相关中间件介绍

数据库相关平台主要解决以下三个方面的问题：

为海量前台数据提供高性能、大容量、高可用性的访问

为数据变更的消费提供准实时的保障

高效的异地数据同步

应用层通过分表分库中间件访问数据库，包括读操作（Select）和写操作（update, insert和delete等，DDL, DCL）。写操作会在数据库上产生变更记录，MySQL的变更记录叫binlog, Oracle的称之为redolog, 增量数据订阅与消费中间件解析这些变更，并以统一的格式保存起来，下层应用根据这些数据进行消费应用。当然，在数据库与数据库本身之间也会有数据库迁移的操作，这种操作可以不需要增量数据订阅与消费中间件的数据，而可以自行处理。

数据库中间件有以下几种：

分布式数据库分表分库

数据增量订阅与消费

数据库同步（全量、增量、跨机房、复制）

跨数据库（数据源）迁移

整个产品族图如下：

![](/canel.png)

最上层的是分布式数据库分表分库中间件，负责和上层应用打交道，对应用可表现为一个独立的数据库，而屏蔽底层复杂的系统细节。分布式数据库中间件除了基本的分表分库功能，还可以丰富一下，比如讲读写分离或者水平扩容功能集成在一起，或者比如读写分离本身也可以作为一个独立的中间件。（Cobar, MyCAT, TDDL, DRDS, DDB）

增量数据订阅和消费，用户对数据库操作，比如DML, DCL, DDL等，这些操作会产生增量数据，下层应用可以通过监测这些增量数据进行相应的处理。典型代表Canal，根据MySQL的binlog实现。也有针对Oracle\(redolog\)的增量数据订阅与消费的中间件。（Canal, Erosa）

数据库同步中间件涉及数据库之间的同步操作，可以实现跨（同）机房同步以及异地容灾备份、分流等功能。可以涉及多种数据库，处理之后的数据也可以以多种形式存储。（Otter, JingoBus, DRC）

数据库与数据库之间会有数据迁移（同步）的动作，同款数据同步原理比较简单，比如MySQL主备同步，只要在数据库层进行相应的配置既可，但是跨数据库同步就比较复杂了，比如Oracle-&gt;MySQL. 数据迁移一般包括三个步骤：全量复制，将原数据库的数据全量迁移到新数据库，在这迁移的过程中也会有新的数据产生；增量同步，对新产生的数据进行同步，并持续一段时间以保证数据同步；原库停写，切换新库。将“跨数据库”这个含义扩大一下——“跨数据源”，比如HDFS, HBase, FTP等都可以相互同步。（yugong, DataX）

## 分布式数据库 {#分布式数据库}

随着互联网产品在体量和规模上日益膨胀，无论是Oracle还是MySQL，都会第一时间面临来自磁盘，CPU和内存等单机瓶颈，为此，产品方除了需要不断购买成本难以控制的高规格服务器，还要面临不断迭代的在线数据迁移。在这种情况下，无论是海量的结构化数据还是快速成长的业务规模，都迫切需要一种水平扩展的方法将存储成本分摊到成本可控的商用服务器上。同时，也希望通过线性扩容降低全量数据迁移对线上服务带来的影响，分库分表方案便应运而生。

分表分库类的中间件主要有两种形式向应用提供服务：

一种是以JDBC的jar包形式为Java应用提供直接依赖，Java应用通过提供的JDBC包实现透明访问分布式数据库集群中的各个分库分表，典型代表网易的DDB和阿里的TDDL.

另一种是为应用部署独立的服务来满足应用分库分表的需求，在这种方式下通过标准JDBC访问Proxy，而Proxy则根据MySQL标准通信协议对客户端请求解析，还原应用SQL请求，然后通过本地访问数据库集群，最后再将得到的结果根据MySQL标准通信协议编码返回给客户端。典型代表阿里的Cobar, Cobar变种MyCAT, 阿里的DRDS，网易的DDB proxy模式以及DDB的私有云模式。

Cobar

Cobar 是提供关系型数据库（MySQL）分布式服务的中间件，它可以让传统的数据库得到良好的线性扩展，并看上去还是一个数据库，对应用保持透明。

Cobar以Proxy的形式位于前台应用和实际数据库之间，对前台的开放的接口是MySQL通信协议。将前台SQL语句变更并按照数据分布规则发到合适的后台数据分库，再合并返回结果，模拟单库下的数据库行为。

Cobar属于阿里B2B事业群，始于2008年，在阿里服役3年多，接管3000+个MySQL数据库的schema,集群日处理在线SQL请求50亿次以上。由于Cobar发起人的离职，Cobar停止维护。后续的类似中间件，比如MyCAT建立于Cobar之上，包括现在阿里服役的RDRS其中也复用了Cobar-Proxy的相关代码。

Cobar结构

![](/cobar.png)

与应用之间通过MySQL protocol进行交互，是一个proxy的结构，对外暴露jdbc:mysql://CobarIP:port/schema。对应用透明。

无需引入新的jar包，从访问迁移到数据库访问Cobar可以复用原有的基于JDBC的DAO。

Cobar前后端都实现了MySQL协议，当接受到SQL请求时，会一次进行解释（SQL Parser）和路由（SQL Router）工作，然后使用SQL Executor去后端模块获取数据集（后端模块还负责心跳检查功能）；如果数据集来自多个数据源，Cobar则需要把数据集进行组合（Result Merge），最后返回响应。

数据库连接复用。Cobar使用连接词与后台真是数据库进行交互。（实际应用中，根据应用的不同，使用proxy结构后数据库连接数能够节约2-10倍不等。）

Cobar事务，Cobar在单库的情况下保持事务的强一致性，分库的情况下保持事务的弱一致性，分库事务采用2PC协议，包括执行阶段和提交阶段。

Cobar的前端是NIO的，而后端跟MySQL交互是阻塞模式，其NIO代码只给出了框架，还没有来得及实现。 据称未开源版的Cobar实现了后端的NIO。

Cobar会出现假死，假死以后Cobar会频繁进行主从切换（如果配置了的话），自动切换本身也存在隐患。

可以计算：Cobar的TPS=5,000,000,000/\(3000\*24\*60\*60\)=20。

与Cobar相关的还有一共Cobar-Client.

Cobar通过SQL语句转发的方式实现数据访问。用户发来的SQL语句，Cobar解析其内容，判断该语句所涉及的数据分布在哪个分库上，再将语句转发给此分库执行。当SQL语句中涉及的拆分字段有多值，如 IN, 或where条件中没有出现拆分字段时，该语句将会转发至后台所有分库执行，再将执行结果以MySQL协议包的形式送回应用端。

通信模块，负责从连续的网络数据流中识别出一个个MySQL协议包，再解析协议包识别出SQL语句输出给Parser模块，同时，把Result Merge模块输入的执行结果，编码成MySQL的协议包。它以NIO方式实现，有很高的执行效率。之后进行优化，引入了一个ByteBuffer池，将NIO的Buffer统一管理起来，减少了NIO数据交互时的垃圾回收。

Cobar前端使用的是优化后的NIO通信模块，为了让该模块在后端使用，Cobar去除了JDBC。与后端数据库交互，Cobar直接面向协议，目前实现了基于MySQL协议的后端交互。

水平拆分后，后台有多个数据源，对他们的管理分为两个层次：DataNode和replica\(HA Pool\)。

DataNode管理拆分，一个DataNode存放一个分片的数据，彼此无数据交集。每个分片的数据存多份以保证高可用，每一份叫做一个replica，由HA层管理。每一个replica表示一个具体的数据源，它是一个连接池，池内管理每一个具体的JDBC连接。路由运算只关注到DataNode层，之下的层次对其不可见。

每一份replica之间的数据复制和同步由MySQL本身的replication协议完成，同一时刻只有一个replica提供服务（称为Master，其余replica称为Slave）.Cobar会与之保持心跳，一旦发现它不可用，会切换至另一个replica，解决Oracle单点的第二个问题。

为了节省数据库的机器数量，可以采用下图中的方式部署：

![](/cobar_partition.png)HA

在用户配置了MySQL心跳的情况下，Cobar可以自动向后端连接的MySQL发生心跳，判断MySQL运行状况，一旦运行出现异常，Cobar可以自动切换到备机工作，但需要强调的是：



Cobar的主备切换有两种触发方式，一种是用户手动触发，一种是Cobar的心跳语句检测到异常后自动触发。那么，当心跳检测到主机异常，切换到备机，如果主机恢复了，需要用户手动切回主机工作，Cobar不会在主机恢复时自动切换回主机，除非备机的心跳也返回异常。



Cobar只检查MySQL主备异常，不关心主备之间的数据同步，因此用户需要在使用Cobar之前在MySQL主备上配置双向同步，详情可以参阅MySQL参考手册。



Cobar解决的问题

分布式：Cobar的分布式主要是通过将表放入不同的库来实现。



Cobar支持将一张表水平拆分成多份分别放入不同的库来实现表的水平拆分



Cobar也支持将不同的表放入不同的库



多数情况下，用户将以上两种方式混合使用



这里需要强调的是，Cobar不支持将一张表，例如test表拆分成test\_1, test\_2, test\_3….放在同一个库中，必须拆分后的表分别放入不同的库来实现分布式。



Cobar的约束

不支持跨库情况下的join、分页、排序、子查询操作



SET语句执行会被忽略，事务和字符集设置除外



分库情况下，insert语句必须包括拆分字段列名



分库情况下，update语句不能更新拆分字段的值



不支持SAVEPOINT操作



暂时只支持MySQL数据节点



使用JDBC时，不支持rewriteBatchedStatements=true参数设置（默认为false）



使用JDBC时，不支持useServerPrepStmts=true参数设置（默认为false\)



使用JDBC时，BLOB, BINARY, VARBINARY字段不能使用setBlob\(\)或setBinaryStream\(\)方法设置参数

