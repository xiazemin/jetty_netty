# section5

最近一直想做一个分布式服务的模型用来完成实验室项目的分流计算量的要求，于是上网查找资料，发现有以下几个开源框架或者方法可以采用（按LZ的个人经验分类，如有不妥，请大家指证）：



      1、Netty、Mina和Grizzly



      2、Jetty、Tomcat、Apache Server和Nginx



      3、Thrift、Spring MVC、Spring RESTful和SOA



      4、RMI、Hessian和Burlap



      5、RMI、JMS和RPC





        简单地查阅了资料和百度一下，发现大致可以有以下的联系：



      1、Netty、Mina和Grizzly：网络IO框架，性能各异但Netty适用性比较强，建议Netty。



            和2中的概念比较：



            1）网络IO框架直接封装传输层的TCP/UDP协议，而web容器时基于servlet的，servlet3.0是基于传输层，封装了应用层的协议，包括HTTP协议的；



            2）网络IO框架不需要web容器的支持，可以直接在程序中应用这些框架实现客户端和服务器通信，而Tomcat和Jetty都是Web容器，一般部署在服务器端，对servlet3.0的api提供了不同的实现



            3）客户端使用netty设计通信流程，并设计应用层协议与服务器端的Tomcat/Jetty等Web容器支持的Servlet通信



            4）Netty针对Socket，Jetty针对Servlet



            5）也可以这样理解：得即时通讯想要走http协议的可以用jetty， 否则还是用netty



      2、Jetty、Tomcat、Apache Server和Nginx：Web服务器，基于HTTP协议工作，Jetty可以与tomcat比较和划分为一类，其中Jetty的扩展性更好，但tomcat更稳定，可参考：Jetty的工作与设计原理和Tomcat比较；而Tomcat可以集合到Apache Server中，另外Nginx可以与Apache Server划分一类，可参考：apache vs tomcat vs nginx



