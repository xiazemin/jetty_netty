# Introduction

Jetty 是一个开源的servlet容器，它为基于Java的web容器，例如JSP和servlet提供运行环境。Jetty是使用Java语言编写的，它的API以一组JAR包的形式发布。开发人员可以将Jetty容器实例化成一个对象，可以迅速为一些独立运行（stand-alone）的Java应用提供网络和web连接。

易用性

易用性是 Jetty 设计的基本原则，易用性主要体现在以下几个方面：

通过 XML 或者 API 来对Jetty进行配置；默认配置可以满足大部分的需求；将 Jetty 嵌入到应用程序当中只需要非常少的代码；

可扩展性

在使用了 Ajax 的 Web 2.0 的应用程序中，每个连接需要保持更长的时间，这样线程和内存的消耗量会急剧的增加。这就使得我们担心整个程序会因为单个组件陷入瓶颈而影响整个程序的性能。但是有了 Jetty：

即使在有大量服务请求的情况下，系统的性能也能保持在一个可以接受的状态。利用 Continuation 机制来处理大量的用户请求以及时间比较长的连接。 另外 Jetty 设计了非常良好的接口，因此在 Jetty 的某种实现无法满足用户的需要时，用户可以非常方便地对 Jetty 的某些实现进行修改，使得 Jetty 适用于特殊的应用程序的需求。

易嵌入性

Jetty 设计之初就是作为一个优秀的组件来设计的，这也就意味着 Jetty 可以非常容易的嵌入到应用程序当中而不需要程序为了使用 Jetty 做修改。从某种程度上，你也可以把 Jetty 理解为一个嵌入式的Web服务器。

Jetty 可以作为嵌入式服务器使用，Jetty的运行速度较快，而且是轻量级的，可以在Java中可以从test case中控制其运行。从而可以使自动化测试不再依赖外部环境，顺利实现自动化测试。

和Tomcat比较编辑

原文地址:Jetty和Tomcat的选择：按场景而定\[1\]

1）Jetty更轻量级。这是相对Tomcat而言的。

由于Tomcat除了遵循Java Servlet规范之外，自身还扩展了大量JEE特性以满足企业级应用的需求，所以Tomcat是较重量级的，而且配置较Jetty亦复杂许多。但对于大量普通互联网应用而言，并不需要用到Tomcat其他高级特性，所以在这种情况下，使用Tomcat是很浪费资源的。这种劣势放在分布式环境下，更是明显。换成Jetty，每个应用服务器省下那几兆内存，对于大的分布式环境则是节省大量资源。而且，Jetty的轻量级也使其在处理高并发细粒度请求的场景下显得更快速高效。

2）Jetty更灵活，体现在其可插拔性和可扩展性，更易于开发者对Jetty本身进行二次开发，定制一个适合自身需求的Web Server。

相比之下，重量级的Tomcat原本便支持过多特性，要对其瘦身的成本远大于丰富Jetty的成本。用自己的理解，即增肥容易减肥难。

3）然而，当支持大规模企业级应用时，Jetty也许便需要扩展，在这场景下Tomcat便是更优的。

总结：Jetty更满足公有云的分布式环境的需求，而Tomcat更符合企业级环境。

代码实例编辑

作为嵌入式服务器使用代码实例

Java代码

//代码：以嵌入模式启动Jetty

import org.mortbay.http.HttpContext;

import org.mortbay.http.HttpServer;

import org.mortbay.http.SocketListener;

import org.mortbay.http.handler.ResourceHandler;

public class JettySample{

public static void main\(String\[\]args\)throws Exception{

//创建JettyHttpServer对象

HttpServer server=new HttpServer\(\);

//在端口8080上给HttpServer对象绑上一个listener，使之能够接收HTTP请求

SocketListener listener=new SocketListener\(\);

listener.setPort\(8080\);

server.addListener\(listener\);

//创建一个HttpContext，处理HTTP请求。

HttpContext context=new HttpContext\(\);

//用setContextPath把Context映射到（/web）URL上。

context.setContextPath\("/web"\);

//setResourceBase方法设置文档目录以提供资源

context.setResourceBase\("C:\j2sdk1.4.1\_05"\);

//添加资源处理器到HttpContext，使之能够提供文件系统中的文件

context.addHandler\(new ResourceHandler\(\)\);

server.addContext\(context\);

//启动服务器

server.start\(\);

}

}

需要的jar包：

commons-logging.jar

javax.servlet.jar

org.mortbay.jetty.jar

org.mortbay.jmx.jar

jetty还有对应maven插件

maven pom文件的设置:

&lt;?xmlversion="1.0"encoding="utf-8"?&gt;

&lt;plugin&gt;

&lt;groupId&gt;org.mortbay.jetty&lt;/groupId&gt;

&lt;artifactId&gt;maven-jetty-plugin&lt;/artifactId&gt;

&lt;version&gt;6.1.10&lt;/version&gt;

&lt;configuration&gt;

&lt;scanIntervalSeconds&gt;10&lt;/scanIntervalSeconds&gt;

&lt;stopKey&gt;foo&lt;/stopKey&gt;

&lt;stopPort&gt;9999&lt;/stopPort&gt;

&lt;/configuration&gt;

&lt;executions&gt;

&lt;execution&gt;

&lt;id&gt;start-jetty&lt;/id&gt;

&lt;phase&gt;pre-integration-test&lt;/phase&gt;

&lt;goals&gt;

&lt;goal&gt;run&lt;/goal&gt;

&lt;/goals&gt;

&lt;configuration&gt;

&lt;scanIntervalSeconds&gt;0&lt;/scanIntervalSeconds&gt;

&lt;daemon&gt;true&lt;/daemon&gt;

&lt;/configuration&gt;

&lt;/execution&gt;

&lt;execution&gt;

&lt;id&gt;stop-jetty&lt;/id&gt;

&lt;phase&gt;post-integration-test&lt;/phase&gt;

&lt;goals&gt;

&lt;goal&gt;stop&lt;/goal&gt;

&lt;/goals&gt;

&lt;/execution&gt;

&lt;/executions&gt;

&lt;/plugin&gt;

然后直接通过mvn jetty:run命令就能直接启动

---

注：

在maven中，用plugin的方式使用jetty，需要改动maven的setting.xml文件，才可以使用命令mvn jetty:run.

setting.xml中找到标签&lt;pluginGroups&gt;，增加：

1

&lt;pluginGroup&gt;org.mortbay.jetty&lt;/pluginGroup&gt;

