# Introduction

数据库（分库分表）中间件对比

分区：对业务透明，分区只不过把存放数据的文件分成了许多小块，例如mysql中的一张表对应三个文件.MYD,MYI,frm。



根据一定的规则把数据文件\(MYD\)和索引文件（MYI）进行了分割，分区后的表呢，还是一张表。分区可以把表分到不同的硬盘上，但不能分配到不同服务器上。



优点：数据不存在多个副本，不必进行数据复制，性能更高。

缺点：分区策略必须经过充分考虑，避免多个分区之间的数据存在关联关系，每个分区都是单点，如果某个分区宕机，就会影响到系统的使用。

 



分片：对业务透明，在物理实现上分成多个服务器，不同的分片在不同服务器上



个人感觉跟分库没啥区别，只是叫法不一样而已，值得一提的是关系型数据库和nosql数据库分片的概念以及处理方式是一样的吗？



请各位看官自行查找相关资料予以解答



 



分表：当数据量大到一定程度的时候，都会导致处理性能的不足，这个时候就没有办法了，只能进行分表处理。也就是把数据库当中数据根据按照分库原则分到多个数据表当中，



这样，就可以把大表变成多个小表，不同的分表中数据不重复，从而提高处理效率。



分表也有两种方案：



1. 同库分表：所有的分表都在一个数据库中，由于数据库中表名不能重复，因此需要把数据表名起成不同的名字。



优点：由于都在一个数据库中，公共表，不必进行复制，处理更简单

缺点：由于还在一个数据库中，CPU、内存、文件IO、网络IO等瓶颈还是无法解决，只能降低单表中的数据记录数。

　　　　　　表名不一致，会导后续的处理复杂（参照mysql meage存储引擎来处理）



2. 不同库分表：由于分表在不同的数据库中，这个时候就可以使用同样的表名。



优点：CPU、内存、文件IO、网络IO等瓶颈可以得到有效解决，表名相同，处理起来相对简单

缺点：公共表由于在所有的分表都要使用，因此要进行复制、同步。

　　　　一些聚合的操作，join,group by,order等难以顺利进行



