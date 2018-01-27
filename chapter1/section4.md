# 强制走主库

1 主从延迟问题

在读写分离的情况下，存在一个主库，多个从库。以mysql为例，主库中插入的数据通过binlog同步给从库，下图演示了这个过程：



Image.png



主库中将数据变更写到本地的二进制日志\(binary log\)中。从库通过一个I/O线程从主库拉取binlog，写到中继日志\(relay log\)中，然后通过一个SQL回放线程读取本地的中继日志，将数据变更应用到从库上，使得从库的数据库与主库保持一致。



由于binlog的拉取线程和sql回放线程是异步执行的，因此从库的数据在大部分情况下都是稍微落后于主库的。这就可能会造成以下情况，主库虽然插入了数据，但是从库中无法读取到数这条据，对于一些强一致性的场景，这种情况是无法忍受的。因此DragonHADataSource还提供了强制从主库查询的功能。



2 DragonHADataSource强制走主库

 DragonHADataSource提供了2种强制从主库查询的方式：hint和api



hint



所谓hint，就是提示的意思。我们通过在sql的前面加一些特殊的标记，来表示强制走主库。例如：



/\*master\*/ SELECT \* FROM user

 Api



所以Api指的是，就是通过java代码的方式来设置走主库。dragon提供了一个工具类DragonHAHintUtil来进行设置，使用方式如下：



DragonHAHintUtil.forceMaster\(\);

 

//此条查询sql会走主库

PreparedStatement preparedStatement = connection.prepareStatement\("SELECT \* FROM user "\);

ResultSet resultSet = preparedStatement.executeQuery\(\);

 

DragonHAHintUtil.clear\(\);

3 事务

在开启了事务的情况下，一个事务中的sql都会走主库，如



Connection connection = dragonHADatasource.getConnection\(\);

connection.setAutoCommit\(false\);

try {

    PreparedStatement ps1 = connection.prepareStatement\("INSERT into user\(name\) VALUES \('huhuamin'\)"\);

    ps1.execute\(\);

    PreparedStatement ps2 = connection.prepareStatement\("SELECT \* FROM user"\);

    ResultSet resultSet = ps2.executeQuery\(\);

    //...

    connection.commit\(\);

} catch \(Exception e\) {

    connection.rollback\(\);

}

这段代码中分别执行了insert和select语句，而由于在一个事务中，select语句也会走主库。



4 DragonHADataSource路由规则总结



