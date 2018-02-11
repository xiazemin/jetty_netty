# Jetty与tomcat的比较

Google 应用系统引擎最初是以 Apache Tomcat 作为其 webserver/servlet 容器的，但最终将切换到 Jetty 上。 这个决定让许多开发人员都诧异的想问：为什么要做这样的改变？Tomcat 有什么问题吗？ 我们获得的一次访问 Webtide ——Jetty 背后的公司——里的这个团队的机会，得到了关于这个决定背后更详细的信息。



 



记者： 为什么Google选择Jetty作为其应用系统的引擎，而不是 Tomcat 或其他的？



 



Google选择Jetty的关键原因是它的体积和灵活性。 在云计算里，体积的因素是很重要，如果你运行几万个Jetty的实例（Google就是这样干的），每个server省1兆，那就会省10几个G的内存（或能够给其他应用提供更多的内存）。



Jetty 被设计成了可插拔和可扩展的特性，这样Google就可以高度的自定义它。 他们在其中替换了他们自己的HTTP connector，Google认证，以及他们自己的session集群。也真是奇怪，这个特性对于云计算来说是非常出色的，但同时也让Jetty非常 适合嵌入小的设备中，例如手机和机顶盒。



 



记者： 是什么促使Jetty成为Java里出色的servlet容器？



 



我们在开发Jetty时，并没有想着要把它开发成一个全功能的应用server（尽管它是的）。每一项功能都考虑了可插拔性，所以，如果你不需要 他，你就可以不把它加载到内存里，把它从request 处理调用链中去掉。如果你不需要sessons，你可以把session处理器拿掉，这样你就不要浪费资源去来回寻找session cookie了。当你每秒钟都有出来上千个请求时，这些微小的查找动作的开销是非常的大的。



 



我们也并没有想当然的企图通过设计就可以得到最优化的代码，我们是如同收集沙粒般，每次得到一些人告诉我们如何才能有好的JVMs优化和垃圾回收办 法。这是真的，已经很小心的代码仍然能被优化，最后的效果就是避免创建新的对象。例如，我们在Jetty里使用并行处理技术，但我们并没有使用很多标准的 并行处理数据结构，因为这需要创建太多的对象。所以，只是作为个例子，我们使用了双并行锁循环 arrays，而不是采用并行链式 lists，这样我们就能够在不创建对象的情况下，获得了非阻塞并行效果。



 



记者： 是什么使Jetty成为开发人员的一个有用的server平台的（例如：testing）？



 



Jetty 已经在一些流行框架中内置了，例如GWT，scala/lift，grails，Jruby等等，还有很多。如果你使用了这些技术，你就直接可以用 Jetty了。 Jetty-maven 插件是另外一个非常优秀的开发工具，它能让web应用在不打包成war文件的情况下运行。源文件可以直接编辑，在不需要把它重新放进war文件的情况下获 得测试结果。 Jetty嵌入式的特征让我们不再需要写通过写那些main方法、通过你的IDE，调试器或 profiler 来运行之类的无聊的事情。



 



记者： Jetty在处理 client-server 请求时有什么独特的地方吗？



 



Jetty 现在是一个第二代的异步处理server。 过去的两年里，我们让Jetty实现了处理异步请求的功能，这成了它核心架构的一部分。就像其他的支持异步serlets容器一样，我想，他们会发现这个 东西并不是看起来的那么简单和容易。 我们的异步HTTP引擎被我们复用在了HTTP client 上，所以我们可以大量的降低request 和 responses 消耗。



同时，就像我之前提到过的，我们的请求处理器是可扩展和可插拔的，这让web application可以被单独省略掉，或者是单独使用，或者是进一步扩展的application。



 



记者： 有没有其他Jetty使用的案例，大的或小的？



 



使用Jetty的公司有像Zimbra/Yahoo，这意味着Jetty正作为web mail 服务器，为百万级的用户提供服务。 Eclipse IDE把它内置了进去，这意味着有成百万的开发者在桌面运行Jetty。 Jetty被 hadoop map/reduce cluster使用，在其上有几千个点的集群，处理着世界最大的TB级别的数据分类排序工作。 我们也有 J2ME 的接口，有本地编译器，所以我们可以在手机上，家用路由器和 Java cards 上运行。 更多的Jetty使用的例子可以参考http://docs.codehaus.org/display/JETTY/Jetty+Powered



 



记者： Jetty的将来或蓝图是怎样的？



 



Jetty 最近的计划是发布 7.0.0 版本，这将会完全的迁移到eclipse foundation 下。 Jetty 7 将会支持很多 servlet 3.0 的特征，但是并不会使用新的API 和 不会依赖Java 1.6 。 Jetty 7后，很快我们会发布Jetty 8，这将会完全支持 servlet 3.0 和 Java 1.6，Jetty 会继续的创新 和跟踪各种web 2.0 里的其他的新成果。 我们现在已经能支持 Firefox 3.5 里的跨域Ajax功能，我们可以在cometd版本里使用这个。 我们很快就会增加对 WebSocket 和 BWTP 的支持。 对 Google wave 以及相关协议的支持的问题已经优先排到了我们的议事日程上了。



 



记者： Google/Jetty 还有其他的计划吗？



 



Google有他们自己下棋的棋局，我们并不清楚。 我们在JavaOne大会上曾经和App Engine开发者们有个简单的对话，我们愿意听他们任何的反馈和意见，用来改进Jetty的可嵌入性和可扩展性。



下面的跟Webtide团队的讨论中，我们询问了SpringSource 从Jetty转换到Tomcat的事情。



 



记者： 你们如何看待 SpringSource 把 Grails 从本来作为缺省容器的Jetty换成了Tomcat的事情？



 



原因是grails开发的领导感觉使用Tomcat能从内部的Tomcat开发人员哪里获得更好的”服务“。我猜测，他们把Grails的用户驱赶 到某一个平台，以让SpringSource能更好的销售他们的技术支持服务。几年前我们看到了相同的事情，JBoss 雇佣了一下tomcat开发人员后把Jetty提出成了Tomcat，并最终和Mort Bay达成了商业合同。 很遗憾，这些商业协议对技术选择有如此大的影响，当相同的是，一些基础结构的工程也正聚集到也application server 为中心的队伍里来。



 rails将会继续同时支持对Jetty和Tomcat的集成，但会改成Tomcat为缺省服务。

