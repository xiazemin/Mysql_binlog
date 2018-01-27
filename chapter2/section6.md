# Apache Camel

Apache Camel是一个非常实用的规则引擎库，能够用来处理来自于不同源的事件和信息。你可以在使用不同的协议比如VM，HTTP，FTP，JMS甚至是文件系统中来传递消息，并且让你的操作逻辑和传递逻辑保持分离，这能够让你更专注于消息的内容。

首先创建一个Maven项目的pom.xml。



&lt;?xml version="1.0" encoding="UTF-8"?&gt; 

&lt;project xmlns="http://maven.apache.org/POM/4.0.0" 

xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 

xsi:schemaLocation="  

http://maven.apache.org/POM/4.0.0  

http://maven.apache.org/maven-v4\_0\_0.xsd"&gt; 

 

&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt; 

&lt;groupId&gt;camel-spring-demo&lt;/groupId&gt; 

&lt;artifactId&gt;camel-spring-demo&lt;/artifactId&gt; 

&lt;version&gt;1.0-SNAPSHOT&lt;/version&gt; 

&lt;packaging&gt;jar&lt;/packaging&gt; 

 

&lt;properties&gt; 

&lt;project.build.sourceEncoding&gt;UTF-8&lt;/project.build.sourceEncoding&gt; 

&lt;camel.version&gt;2.11.1&lt;/camel.version&gt; 

&lt;/properties&gt; 

 

&lt;dependencies&gt; 

&lt;dependency&gt; 

&lt;groupId&gt;org.apache.camel&lt;/groupId&gt; 

&lt;artifactId&gt;camel-core&lt;/artifactId&gt; 

&lt;version&gt;${camel.version}&lt;/version&gt; 

&lt;/dependency&gt; 

&lt;dependency&gt; 

&lt;groupId&gt;org.slf4j&lt;/groupId&gt; 

&lt;artifactId&gt;slf4j-simple&lt;/artifactId&gt; 

&lt;version&gt;1.7.5&lt;/version&gt; 

&lt;/dependency&gt; 

&lt;/dependencies&gt; 

 

&lt;/project&gt; 

在这里我们只用到了camel-core.jar包，实际上它提供了许多你可能用到的实用组件。出于日志记录的目的，我使用了slf4j-simple来作为日志记录的实现，从而我们可以从控制台上看到输出。



接下来我们只需要构造一个路由类。路由就好比是Camel中怎样将消息从一端传递到另一端的一个指令定义。我们将会创建src/main/java /camelcoredemo/TimerRouteBuilder.java文件，每隔一秒向处理器发送一个消息，简单打印出来。



package camelcoredemo;  

 

import org.slf4j.\*;  

import org.apache.camel.\*;  

import org.apache.camel.builder.\*;  

 

public class TimerRouteBuilder extends RouteBuilder {  

static Logger LOG = LoggerFactory.getLogger\(TimerRouteBuilder.class\);  

public void configure\(\) {  

from\("timer://timer1?period=1000"\)  

.process\(new Processor\(\) {  

public void process\(Exchange msg\) {  

LOG.info\("Processing {}", msg\);  

}  

}\);  

}  

} 

以上就是这个示例的全部所需，现在编译运行。



bash&gt; mvn compile  

bash&gt; mvn exec:java -Dexec.mainClass=org.apache.camel.main.Main -Dexec.args='-r camelcoredemo.TimerRouteBuilder' 

注意，这里我们并没有编写Java类的main入口，我们只是将RouteBuilder的类名当作参数简单传递给 org.apache.camel.main.Main,然后它将自动加载路由。



控制CamelContext



当启动Camel后，它会创建一个CamelContext对象，该对象拥有了很多关于如何运行Camel的信息，还包含我们所创建的Route的定义。现在如果你想通过CamelContext获得更多的控制，那么你需要编写自己的主类代码。我在这举个简单的例子。



package camelcoredemo;  

 

 

 

 

import org.slf4j.\*;  

import org.apache.camel.\*;  

import org.apache.camel.impl.\*;  

import org.apache.camel.builder.\*;  

 

 

 

 

public class TimerMain {  

static Logger LOG = LoggerFactory.getLogger\(TimerMain.class\);  

public static void main\(String\[\] args\) throws Exception {  

new TimerMain\(\).run\(\);  

}  

void run\(\) throws Exception {  

final CamelContext camelContext = new DefaultCamelContext\(\);  

camelContext.addRoutes\(createRouteBuilder\(\)\);  

camelContext.setTracing\(true\);  

camelContext.start\(\);  

 

 

 

 

Runtime.getRuntime\(\).addShutdownHook\(new Thread\(\) {  

public void run\(\) {  

try {  

camelContext.stop\(\);  

} catch \(Exception e\) {  

throw new RuntimeException\(e\);  

}  

}  

}\);  

 

 

 

 

waitForStop\(\);  

}  

RouteBuilder createRouteBuilder\(\) {  

return new TimerRouteBuilder\(\);  

}  

void waitForStop\(\) {  

while \(true\) {  

try {  

Thread.sleep\(Long.MAX\_VALUE\);  

} catch \(InterruptedException e\) {  

break;  

}  

}  

}  

} 

可以看到，我们在createRouteBuilder\(\)方法中重用了已有的TimerRouteBuilder类。现在我们的主类对在什么时候创建、启动、停止CamelContext有了完全的控制。context\(camelContext\)对象允许你全局性地控制如何配置Camel，而不是在 Route级。它的JavaDoc链接给出了所有setter方法，你可以研究下它都可以做些什么。



注意到一点，我们也需要在我们的主类中提供少量设置代码。首先我们需要处理优雅关闭的问题，所以我们增加了一个Java关闭回调函数去调用context 的stop\(\)方法。其次在context已经启动后，我们需要增加一个线程阻塞。如果在启动后你不阻塞你的主线程，那么它会在启动后就简单的退出了，那就没啥用了。你会把Camel一直作为一个服务\(就像一个服务器\)运行，直至你按下CTRL+C键去终止该进程。



改善启动CamelContext的主类



如果你不想像上面例子一样过多的处理主类设置代码，那么你可以简单地继承由camel-core提供的 org.apache.camel.main.Main类作为代替。通过利用这个类，你不仅可以让你的context自动设置，还可以获得所有附加的命令行特性，比如控制进程运行多久，启用追踪，加载自定义route类等等。



重构了下上一个例子，代码如下：



package camelcoredemo;  

 

import org.slf4j.\*;  

import org.apache.camel.builder.\*;  

import org.apache.camel.main.Main;  

 

public class TimerMain2 extends Main {  

static Logger LOG = LoggerFactory.getLogger\(TimerMain2.class\);  

public static void main\(String\[\] args\) throws Exception {  

TimerMain2 main = new TimerMain2\(\);  

main.enableHangupSupport\(\);  

main.addRouteBuilder\(createRouteBuilder\(\)\);  

main.run\(args\);  

}  

static RouteBuilder createRouteBuilder\(\) {  

return new TimerRouteBuilder\(\);  

}  

} 

现在TimerMain2类的代码比之前的更少了，你可以试试看，它应该和之前的功能一样。



bash&gt; mvn compile  

bash&gt; mvn exec:java -Dexec.mainClass=camelcoredemo.TimerMain2 -Dexec.args='-t' 

注意到我们给出-t选项后，会转储Route追踪。使用-h会看到所有可用的选项。



用Camel的注册机制添加bean



在之前的TimerRouteBuilder例子中，我们已经在代码中创建了一个匿名Processor。现在如果你想将几个不同的Processor放在一起，那么使用Camel的注册机制添加bean的方式将能更好的减少代码混乱。Camel允许你通过将processing当作bean注入到它的 registry space，然后你只要把它们当作bean组件来进行调用。如下是我的重构代码:



package camelcoredemo;  

 

import org.slf4j.\*;  

import org.apache.camel.\*;  

import org.apache.camel.builder.\*;  

import org.apache.camel.main.Main;  

 

public class TimerBeansMain extends Main {  

static Logger LOG = LoggerFactory.getLogger\(TimerBeansMain.class\);  

public static void main\(String\[\] args\) throws Exception {  

TimerBeansMain main = new TimerBeansMain\(\);  

main.enableHangupSupport\(\);  

main.bind\("processByBean1", new Bean1\(\)\);  

main.bind\("processAgainByBean2", new Bean2\(\)\);  

main.addRouteBuilder\(createRouteBuilder\(\)\);  

main.run\(args\);  

}  

static RouteBuilder createRouteBuilder\(\) {  

return new RouteBuilder\(\) {  

public void configure\(\) {  

from\("timer://timer1?period=1000"\)  

.to\("bean:processByBean1"\)  

.to\("bean:processAgainByBean2"\);  

}  

};  

}  

 

// Processor beans  

static class Bean1 implements Processor {  

public void process\(Exchange msg\) {  

LOG.info\("First process {}", msg\);  

}  

}  

static class Bean2 implements Processor {  

public void process\(Exchange msg\) {  

LOG.info\("Second process {}", msg\);  

}  

}  

} 

现在Route类更简洁明了，同时处理代码也被重构到了独立的类中。当你需要编写很复杂的Route来实现业务逻辑时，这种方式能够帮助你更好的组织和测试你的代码。它能够让你构建像”乐高“积木那样可复用的POJO bean。Camel的registry space同样可用于其他很多用途，比如你可以自定义许多具有附加功能的endpoint组件或者注册一些信息，更或者替换线程池实现策略之内的事情。



上述Route示例是用所谓的Java DSL来构成的，它的可读性较高，你可以用IDE提供的支持查看所有可用于Route的方法。



我希望这篇文章能够帮助你跳过Camel的摸索阶段。除了已经提到的事件组件之外，camel还提供了如下组件：



bean component

browse component

dataset component

direct component

file component

log component

mock component

properties component

seda component

test component

timer component

stub component

validator component

vm component

xslt component

