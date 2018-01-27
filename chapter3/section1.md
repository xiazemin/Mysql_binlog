# Canal记录mysql的binlog日志监听



定位： 基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了mysql



工作原理

mysql主备复制实现：

从上层来看，复制分成三步：



master将改变记录到二进制日志\(binary log\)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；

slave将master的binary log events拷贝到它的中继日志\(relay log\)；

slave重做中继日志中的事件，将改变反映它自己的数据。

canal的工作原理：

canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议

mysql master收到dump请求，开始推送binary log给slave\(也就是canal\)

canal解析binary log对象\(原始为byte流\)

 



几点说明：

a:canal的原理是基于mysql binlog技术，所以一定要需要开启mysql的binlog写入功能，并且配置binlog模式为row.



b:canal的原理是模拟自己为mysql salave，所以这里一定需要做为mysql slave的相关权限。

