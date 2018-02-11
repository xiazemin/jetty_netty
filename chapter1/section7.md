# Spark Netty与Jetty

　　spark呢，**对Netty API又做了一层封装**，那么Netty是什么呢~是个鬼。它基于NIO的服务端客户端框架，具体不再说了，下面开始。

　　创建了一个线程工厂，生成的线程都给定一个前缀名。

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208231201929-145505098.jpg)

　　像一般的netty框架一样，**创建Netty的EventLoopGroup**:

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208231318272-418801942.jpg)

　　在常用的netty框架中呢，会创建客户端辅助类，设置SocketChannel：

```
  Bootstrap b = new Bootstrap();
  b.group(group).channel(NioSocketChannel.class)
```

　　spark中呢**根据参数IOMode**，返回正确的客户端SocketChannel:

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208232633351-1392075356.jpg)　　返回正确的服务端socketChannel:

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208232717366-593371430.jpg)

　　返回远端的Channel地址：

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208232059991-2020805438.jpg)

　　创建一个ByteBuf对**本地线程缓存禁用的分配器**。ByteBuf是**由事件循环线程分配**，所以线程本地缓存**对于TransportClient是禁用的**，ByteBuf释放是由Executor线程，不是事件循环线程完成。本地线程缓存经常会延迟ByteBuf的回收，导致巨大的内存消耗。



![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208232423429-391309626.jpg)

　　Spark这个禽兽，对Jetty也进行了封装，什么是Jetty呢，它是以java作为开发语言的servlet容器，它的API以一组jar包的形式发布，**提供网络和web服务**.在我理解，Netty是用socket~Jetty呢 就是Http~那么下来，我们看一下JettyUtils：

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208233352835-622643064.jpg)

　　createServlet,**生成HttpServlet匿名内部类**，此Responder类型**发生隐式转换**，转换为用户传入的函数参数。

　　要为Jetty创建servlet,就涉及**ServletContextHandler**的API的使用，**生成ServletContextHandler**:

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208233707288-1464117050.jpg)

　　创建给定路径为前缀的请求的响应处理，将SparkUI中的全部handler加入**ContextHandlerCollection**.，如果使用配置**spark.ui.filters**指定了filter,则给所有handler添加filter.然后调用**startServiceOnPort**,最终回调函数connect:

![](https://images2015.cnblogs.com/blog/820234/201612/820234-20161208234130132-1657833433.jpg)

  


