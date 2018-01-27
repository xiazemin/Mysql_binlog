# 其他同款中间件

Altas, Vitess, Heisenberg, CDS, DDB， OneProxy等等。



Atlas



Qihoo 360.

Web平台部基础架构团队开发维护的一个基于MySQL协议的数据中间层项目，它是在mysql-proxy 0.8.2版本上对其进行优化，增加了一些新的功能特性。

Atlas是一个位于应用程序与MySQL之间，它实现了MySQL的客户端和服务端协议，作为服务端与应用程序通讯，同时作为客户端与MySQL通讯。它对应用程序屏蔽了DB的细节。

Altas不能实现分布式分表，所有的字表必须在同一台DB的同一个DataBase里且所有的字表必须实现建好，Altas没有自动建表的功能。



Heisenberg



Baidu.

其优点：分库分表与应用脱离，分库表如同使用单库表一样，减少db连接数压力，热重启配置，可水平扩容，遵守MySQL原生协议，读写分离，无语言限制，mysqlclient, c, Java都可以使用Heisenberg服务器通过管理命令可以查看，如连接数，线程池，结点等，并可以调整采用velocity的分库分表脚本进行自定义分库表，相当的灵活。

（开源版已停止维护）



CDS



JD. Completed Database Sharding.

CDS是一款基于客户端开发的分库分表中间件产品，实现了JDBC标准API，支持分库分表，读写分离和数据运维等诸多共，提供高性能，高并发和高可靠的海量数据路由存取服务，业务系统可近乎零成本进行介入，目前支持MySQL, Oracle和SQL Server.

\(架构上和Cobar，MyCAT相似，直接采用jdbc对接，没有实现类似MySQL协议，没有NIO,AIO，SQL Parser模块采用JSqlParser, Sql解析器有：druid&gt;JSqlParser&gt;fdbparser.\)



DDB



猪场. Distributed DataBase.

DDB经历了三次服务模式的重大更迭：Driver模式-&gt;Proxy模式-&gt;云模式。



Driver模式：基于JDBC驱动访问，提供一个db.jar, 和TDDL类似， 位于应用层和JDBC之间.



Proxy模式：在DDB中搭建了一组代理服务器来提供标准的MySQL服务，在代理服务器内部实现分库分表的逻辑。应用通过标准数据库驱动访问DDB Proxy, Proxy内部通过MySQL解码器将请求还原为SQL, 并由DDB Driver执行得到结果。



私有云模式：基于网易私有云开发的一套平台化管理工具Cloudadmin, 将DDB原先Master的功能打散，一部分分库相关功能集成到proxy中，如分库管理、表管理、用户管理等，一部分中心化功能集成到Cloudadmin中，如报警监控，此外，Cloudadmin中提供了一键部署、自动和手动备份，版本管理等平台化功能。

