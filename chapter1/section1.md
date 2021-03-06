# section1

Jetty 的基本架构

Jetty 目前的是一个比较被看好的 Servlet 引擎，它的架构比较简单，也是一个可扩展性和非常灵活的应用服务器，它有一个基本数据模型，这个数据模型就是 Handler，所有可以被扩展的组件都可以作为一个 Handler，添加到 Server 中，Jetty 就是帮你管理这些 Handler。

Jetty 的基本架构

下图是 Jetty 的基本架构图，整个 Jetty 的核心组件由 Server 和 Connector 两个组件构成，整个 Server 组件是基于 Handler 容器工作的，它类似与 Tomcat 的 Container 容器，Jetty 与 Tomcat 的比较在后面详细介绍。Jetty 中另外一个比不可少的组件是 Connector，它负责接受客户端的连接请求，并将请求分配给一个处理队列去执行

Jetty 中还有一些可有可无的组件，我们可以在它上做扩展。如 JMX，我们可以定义一些 Mbean 把它加到 Server 中，当 Server 启动的时候，这些 Bean 就会一起工作。

整个 Jetty 的核心是围绕着 Server 类来构建，Server 类继承了 Handler，关联了 Connector 和 Container。Container 是管理 Mbean 的容器。Jetty 的 Server 的扩展主要是实现一个个 Handler 并将 Handler 加到 Server 中，Server 中提供了调用这些 Handler 的访问规则。

整个 Jetty 的所有组件的生命周期管理是基于观察者模板设计，它和 Tomcat 的管理是类似的。

Jetty 主要提供了两种 Handler 类型，一种是 HandlerWrapper，它可以将一个 Handler 委托给另外一个类去执行，如我们要将一个 Handler 加到 Jetty 中，那么就必须将这个 Handler 委托给 Server 去调用。配合 ScopeHandler 类我们可以拦截 Handler 的执行，在调用 Handler 之前或之后，可以做一些另外的事情，类似于 Tomcat 中的 Valve；另外一个 Handler 类型是 HandlerCollection，这个 Handler 类可以将多个 Handler 组装在一起，构成一个 Handler 链，方便我们做扩展。

Jetty 的启动过程

Jetty 的入口是 Server 类，Server 类启动完成了，就代表 Jetty 能为你提供服务了。它到底能提供哪些服务，就要看 Server 类启动时都调用了其它组件的 start 方法。从 Jetty 的配置文件我们可以发现，配置 Jetty 的过程就是将那些类配置到 Server 的过程

因为 Jetty 中所有的组件都会继承 LifeCycle，所以 Server 的 start 方法调用就会调用所有已经注册到 Server 的组件，Server 启动其它组件的顺序是：首先启动设置到 Server 的 Handler，通常这个 Handler 会有很多子 Handler，这些 Handler 将组成一个 Handler 链。Server 会依次启动这个链上的所有 Handler。接着会启动注册在 Server 上 JMX 的 Mbean，让 Mbean 也一起工作起来，最后会启动 Connector，打开端口，接受客户端请求，启动逻辑非常简单。

接受请求

Jetty 作为一个独立的 Servlet 引擎可以独立提供 Web 服务，但是它也可以与其他 Web 应用服务器集成，所以它可以提供基于两种协议工作，一个是 HTTP，一个是 AJP 协议。如果将 Jetty 集成到 Jboss 或者 Apache，那么就可以让 Jetty 基于 AJP 模式工作。下面分别介绍 Jetty 如何基于这两种协议工作，并且它们如何建立连接和接受请求的。

基于 HTTP 协议工作

如果前端没有其它 web 服务器，那么 Jetty 应该是基于 HTTP 协议工作。也就是当 Jetty 接收到一个请求时，必须要按照 HTTP 协议解析请求和封装返回的数据。那么 Jetty 是如何接受一个连接又如何处理这个连接呢？

我们设置 Jetty 的 Connector 实现类为 org.eclipse.jetty.server.bi.SocketConnector 让 Jetty 以 BIO 的方式工作，Jetty 在启动时将会创建 BIO 的工作环境，它会创建 HttpConnection 类用来解析和封装 HTTP1.1 的协议，ConnectorEndPoint 类是以 BIO 的处理方式处理连接请求，ServerSocket 是建立 socket 连接接受和传送数据，Executor 是处理连接的线程池，它负责处理每一个请求队列中任务。acceptorThread 是监听连接请求，一有 socket 连接，它将进入下面的处理流程。

当 socket 被真正执行时，HttpConnection 将被调用，这里定义了如何将请求传递到 servlet 容器里，有如何将请求最终路由到目的 servlet，关于这个细节可以参考《 servlet 工作原理解析》一文。

Jetty 创建接受连接环境需要三个步骤：



创建一个队列线程池，用于处理每个建立连接产生的任务，这个线程池可以由用户来指定，这个和 Tomcat 是类似的。

创建 ServerSocket，用于准备接受客户端的 socket 请求，以及客户端用来包装这个 socket 的一些辅助类。

创建一个或多个监听线程，用来监听访问端口是否有连接进来。

相比 Tomcat 创建建立连接的环境，Jetty 的逻辑更加简单，牵涉到的类更少，执行的代码量也更少了。



当建立连接的环境已经准备好了，就可以接受 HTTP 请求了

Accetptor 线程将会为这个请求创建 ConnectorEndPoint。HttpConnection 用来表示这个连接是一个 HTTP 协议的连接，它会创建 HttpParse 类解析 HTTP 协议，并且会创建符合 HTTP 协议的 Request 和 Response 对象。接下去就是将这个线程交给队列线程池去执行了。



基于 AJP 工作

通常一个 web 服务站点的后端服务器不是将 Java 的应用服务器直接暴露给服务访问者，而是在应用服务器，如 Jboss 的前面在加一个 web 服务器，如 Apache 或者 nginx，我想这个原因大家应该很容易理解，如做日志分析、负载均衡、权限控制、防止恶意请求以及静态资源预加载等等。

这种架构下 servlet 引擎就不需要解析和封装返回的 HTTP 协议，因为 HTTP 协议的解析工作已经在 Apache 或 Nginx 服务器上完成了，Jboss 只要基于更加简单的 AJP 协议工作就行了，这样能加快请求的响应速度。



对比 HTTP 协议的时序图可以发现，它们的逻辑几乎是相同的，不同的是替换了一个类 Ajp13Parserer 而不是 HttpParser，它定义了如何处理 AJP 协议以及需要哪些类来配合。



实际上在 AJP 处理请求相比较 HTTP 时唯一的不同就是在读取到 socket 数据包时，如何来转换这个数据包，是按照 HTTP 协议的包格式来解析就是 HttpParser，按照 AJP 协议来解析就是 Ajp13Parserer。封装返回的数据也是如此。



让 Jetty 工作在 AJP 协议下，需要配置 connector 的实现类为 Ajp13SocketConnector，这个类继承了 SocketConnector 类，覆盖了父类的 newConnection 方法，为的是创建 Ajp13Connection 对象而不是 HttpConnection



