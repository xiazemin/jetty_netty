# section2

基于 NIO 方式工作

前面所描述的 Jetty 建立客户端连接到处理客户端的连接都是基于 BIO 的方式，它也支持另外一种 NIO 的处理方式，其中 Jetty 的默认 connector 就是 NIO 方式。

创建一个 Selector 相当于一个观察者，打开一个 Server 端通道，把这个 server 通道注册到观察者上并且指定监听的事件。然后遍历这个观察者观察到事件，取出感兴趣的事件再处理。这里有个最核心的地方就是，我们不需要为每个被观察者创建一个线程来监控它随时发生的事件。而是把这些被观察者都注册一个地方统一管理，然后由它把触发的事件统一发送给感兴趣的程序模块。这里的核心是能够统一的管理每个被观察者的事件，所以我们就可以把服务端上每个建立的连接传送和接受数据作为一个事件统一管理，这样就不必要每个连接需要一个线程来维护了。



这里需要注意的地方时，很多人认为监听 SelectionKey.OP\_ACCEPT 事件就已经是非阻塞方式了，其实 Jetty 仍然是用一个线程来监听客户端的连接请求，当接受到请求后，把这个请求再注册到 Selector 上，然后才是非阻塞方式执行。这个地方还有一个容易引起误解的地方是：认为 Jetty 以 NIO 方式工作只会有一个线程来处理所有的请求，甚至会认为不同用户会在服务端共享一个线程从而会导致基于 ThreadLocal 的程序会出现问题，其实从 Jetty 的源码中能够发现，真正共享一个线程的处理只是在监听不同连接的数据传送事件上，比如有多个连接已经建立，传统方式是当没有数据传输时，线程是阻塞的也就是一直在等待下一个数据的到来，而 NIO 的处理方式是只有一个线程在等待所有连接的数据的到来，而当某个连接数据到来时 Jetty 会把它分配给这个连接对应的处理线程去处理，所以不同连接的处理线程仍然是独立的。



Jetty 的 NIO 处理方式和 Tomcat 的几乎一样，唯一不同的地方是在如何把监听到事件分配给对应的连接的处理方式。从测试效果来看 Jetty 的 NIO 处理方式更加高效。

处理请求

下面看一下 Jetty 是如何处理一个 HTTP 请求的。



实际上 Jetty 的工作方式非常简单，当 Jetty 接受到一个请求时，Jetty 就把这个请求交给在 Server 中注册的代理 Handler 去执行，如何执行你注册的 Handler，同样由你去规定，Jetty 要做的就是调用你注册的第一个 Handler 的 handle\(String target, Request baseRequest, HttpServletRequest request, HttpServletResponse response\) 方法，接下去要怎么做，完全由你决定。



要能接受一个 web 请求访问，首先要创建一个 ContextHandler，如下代码所示：

Server server = new Server\(8080\); 

ContextHandler context = new ContextHandler\(\); 

context.setContextPath\("/"\); 

context.setResourceBase\("."\); 

context.setClassLoader\(Thread.currentThread\(\).getContextClassLoader\(\)\); 

server.setHandler\(context\); 

context.setHandler\(new HelloHandler\(\)\); 

server.start\(\); 

server.join\(\);

当我们在浏览器里敲入 http://localhost:8080 时的请求将会代理到 Server 类的 handle 方法，Server 的 handle 的方法将请求代理给 ContextHandler 的 handle 方法，ContextHandler 又调用 HelloHandler 的 handle 方法。这个调用方式是不是和 Servlet 的工作方式类似，在启动之前初始化，然后创建对象后调用 Servlet 的 service 方法。在 Servlet 的 API 中我通常也只实现它的一个包装好的类，在 Jetty 中也是如此，虽然 ContextHandler 也只是一个 Handler，但是这个 Handler 通常是由 Jetty 帮你实现了，我们一般只要实现一些我们具体要做的业务逻辑有关的 Handler 就好了，而一些流程性的或某些规范的 Handler，我们直接用就好了，如下面的关于 Jetty 支持 Servlet 的规范的 Handler 就有多种实现，下面是一个简单的 HTTP 请求的流程。

访问一个 Servlet 的代码：

Server server = new Server\(\); 

Connector connector = new SelectChannelConnector\(\); 

connector.setPort\(8080\); 

server.setConnectors\(new Connector\[\]{ connector }\); 

ServletContextHandler root = new 

ServletContextHandler\(null,"/",ServletContextHandler.SESSIONS\); 

server.setHandler\(root\); 

root.addServlet\(new ServletHolder\(new 

org.eclipse.jetty.embedded.HelloServlet\("Hello"\)\),"/"\); 

server.start\(\); 

server.join\(\);

创建一个 ServletContextHandler 并给这个 Handler 添加一个 Servlet，这里的 ServletHolder 是 Servlet 的一个装饰类，它十分类似于 Tomcat 中的 StandardWrapper。



