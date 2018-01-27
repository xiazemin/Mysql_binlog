# MyCAT

从定义和分类看，它是一个开源的分布式数据库系统，是一个实现了MySQL协议的Server，前端用户可以把它看做是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL Native Protocol与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

MyCAT发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL, SQL Server, Oracle, DB2, PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。

MyCAT是一个强大的数据库中间件，不仅仅可以用作读写分离，以及分表分库、容灾管理，而且可以用于多租户应用开发、云平台基础设施，让你的架构具备很强的适应性和灵活性，借助于即将发布的MyCAT只能优化模块，系统的数据访问瓶颈和热点一目了然，根据这些统计分析数据，你可以自动或手工调整后端存储，将不同的表隐射到不同存储引擎上，而整个应用的代码一行也不用改变。

MyCAT是在Cobar基础上发展的版本，两个显著提高：

后端由BIO改为NIO，并发量有大幅提高；

增加了对Order By, Group By, Limit等聚合功能（虽然Cobar也可以支持Order By, Group By, Limit语法，但是结果没有进行聚合，只是简单返回给前端，聚合功能还是需要业务系统自己完成）

MyCAT架构

![](/mycat.png)

事务是弱XA

MyCAT的原理中最重要的一个动词是“拦截”，它拦截了用户发来的SQL语句，首先对SQL语句做了一些特定的分析：如分片分析，路由分析、读写分离分析、缓存分析等，然后将此SQL发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户。

MyCAT对自身不支持的SQL语句提供了一种解决方案——在要执行的SQL语句前添加额外的一段由注解SQL组织的代码，这样SQL就能正确执行，这段代码称之为“注解”。注解的使用相当于对MyCAT不支持的SQL语句做了一层透明代理转发，直接交给目标的数据节点进行SQL语句执行。

MyCAT自身有类似其他数据库的管理监控方式，可以通过MySQL命令行，登录管理端口（9066）执行相应的SQL进行管理，也可以通过jdbc的方式进行远程连接管理。

HA

![](/mycat_proxy.png)

MyCAT作为一个代理层中间件，MyCAT系统的高可用设计到MyCAT本身的高可用以及后端MySQL的高可用. 在多数情况下，建议采用MySQL主从复制高可用性配置并交付给MyCAT来完成后端MySQL节点的主从自动切换。

MySQL侧的HA

MySQL节点开启主从复制的配置方案，并将主节点配置为MyCAT的dataHost里的writeNode，从节点配置为readNode，同时MyCAT内部定期对一个dataHost里的所有writeHost与readHost节点发起心跳检测。

正常情况下，MyCAT将第一个writeHost作为写节点，所有的DML SQL会发送此节点。

若MyCAT开启了读写分离，则查询节点会根据读写分离的策略发往readHost\(+writeHost\)执行。

如果第一个writeHost宕机，MyCAT会在默认的三次心跳检测失败后，自动切换到下一个可用的writeHost执行DML SQL语句

当原来配置的MySQL写节点宕机恢复后，作为从节点，跟随新的主节点，重新配置主从同步。

MyCAT自身的HA

官方建议是采用基于硬件的负载聚亨或者软件方式的HAproxy等。

如果还担心HAproxy的稳定性和但节点问题，则可以用keepalived的VIP的浮动功能，加以强化。

MyCAT功能和特性

支持SQL 92标准

支持Mysql集群，可以作为Proxy使用

支持JDBC连接多数据库

支持NoSQL数据库

支持galera sfor mysql集群，percona-cluster或者mariadb cluster，提供高可用性分片集群

自动故障切换，高可用性

支持读写分离，支持MySQL双主多从，以及一主多从的模式

支持全局表，数据自动分片到多个节点，用于高效表关联查询

支持一致性Hash分片，有效解决分片扩容难题

多平台支持，部署和试试简单

支持Catelet开发，类似数据库存储过程，用于跨分片复杂SQL的人工智能编码实现

支持NIO与AIO两种网络通信机制，windows下建议AIO,Linux下目前建议NIO

支持MySQL存储过程调用

以插件的方式支持SQL拦截和改写

支持自增长逐渐、支持Oracle的Sequence机制

支持Mysql, MongoDB，Oracle, SQL Server, Hive, DB2, PostgreSQL等。

MyCAT目前的项目

MyCAT-Server:MyCAT核心服务

MyCAT-Spider:MyCAT爬虫技术

MyCAT-ConfigCenter:MyCAT配置中心

MyCAT-BigSQL:MyCAT大数据处理（暂未更细）

MyCAT-Web:MyCAT监控及web（新版开发中）

MyCAT-Balance:MyCAT负载均衡（暂未更细）

