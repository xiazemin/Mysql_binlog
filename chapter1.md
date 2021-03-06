# chapter1

1. 什么是中间件

传统的架构模式就是 应用连接数据库直接对数据进行访问，这种架构特点就是简单方便。

但是随着目前数据量不断的增大我们就遇到了问题:

单个表数据量太大

单个库数据量太大

单台数据量服务器压力很大

读写速度遇到瓶颈

当面临以上问题时，我们会想到的第一种解决方式就是 向上扩展\(scale up\) 简单来说就是不断增加硬件性能。这种方式只能暂时解决问题，当业务量不断增长时还是解决不了问题。特别是淘宝，facebook，youtube这种业务成线性，甚至指数级上升的情况

此时我们不得不依赖于第二种方式： 水平扩展 。 直接增加机器，把数据库放到不同服务器上，在应用到数据库之间加一个proxy进行路由，这样就可以解决上面的问题了。

1. 中间件与读写分离

很多人都会把中间件认为是读写分离，其实读写分离只是中间件可以提供的一种功能，最主要的功能还是在于他可以 分库分表

Cobar:

阿里巴巴B2B开发的关系型分布式系统，管理将近3000个MySQL实例。 在阿里经受住了考验，后面由于作者的走开的原因cobar没有人维护 了，阿里也开发了tddl替代cobar。

MyCAT:

社区爱好者在阿里cobar基础上进行二次开发，解决了cobar当时存 在的一些问题，并且加入了许多新的功能在其中。目前MyCAT社区活 跃度很高，目前已经有一些公司在使用MyCAT。总体来说支持度比 较高，也会一直维护下去，

OneProxy:

数据库界大牛，前支付宝数据库团队领导楼总开发，基于mysql官方 的proxy思想利用c进行开发的，OneProxy是一款商业收费的中间件， 楼总舍去了一些功能点，专注在性能和稳定性上。有朋友测试过说在 高并发下很稳定。

Vitess:

这个中间件是Youtube生产在使用的，但是架构很复杂。 与以往中间件不同，使用Vitess应用改动比较大要 使用他提供语言的API接口，我们可以借鉴他其中的一些设计思想。

Kingshard:

Kingshard是前360Atlas中间件开发团队的陈菲利用业务时间 用go语言开发的，目前参与开发的人员有3个左右， 目前来看还不是成熟可以使用的产品，需要在不断完善。

Atlas:

360团队基于mysql proxy 把lua用C改写。原有版本是支持分表， 目前已经放出了分库分表版本。在网上看到一些朋友经常说在高并 发下会经常挂掉，如果大家要使用需要提前做好测试。

MaxScale与MySQL Route:

这两个中间件都算是官方的吧，MaxScale是mariadb \(MySQL原作者维护的一个版本\)研发的，目前版本不支持分库分表。

MySQL Route是现在MySQL 官方Oracle公司发布出来的一个中间件。

