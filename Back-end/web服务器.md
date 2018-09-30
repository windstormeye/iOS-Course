# 浅析 Web 服务器的工作原理（Java）
## 什么是 Web 服务器，应用服务器和 web 容器？
### web 服务器
在过去很长的一段时间中，它们是有区别的，但是这两个分类慢慢的合并了，而如今在大多情况下可以把它们看成一个整体。在早期，引发出“ web 服务器”的概念是因为通过了 HTTP 协议来提供静态页面内容和图片的服务，当时大部分内容都是静态的，并且 HTTP 1.0 只是一种传送文件的方式，但在不久后 web 服务器提供了 CGI 功能，意味着我们能够为每一个 web 请求启动一个进程来产生动态内容。现在 HTTP 协议已经非常成熟了并且 web 服务器变得更加复杂，拥有了例如缓存、安全和 session 管理等这些附加功能。

![WX20180930-155912@2x.png](https://i.loli.net/2018/09/30/5bb082c0d9a2e.png)

### 应用服务器
在同一时期，应用服务器已经存在并发展了很长一段时间了，大部分产品都指定了“封闭的”产品专用通信协议来互连胖客户端和服务器，在 90 年代，传统的应用服务器产品开始嵌入了 HTTP 通信功能，准备利用网关来实现。不久之后这两者的界限开始变得模糊。同时，web 服务器变得越来越成熟，可以处理更高的负载、更多的并发和拥有更好的特性；应用服务器开始添加越来越多的机遇 HTTP 的通信功能。所有的这些导致了 web 服务器与应用服务器的界线变得更窄了。

目前，应用服务器和 web 服务器之间的界线已经变得模糊不清了，但是人们还把这两个术语分开来。当有人说到 web 服务器时，我们通常要把它认为是以 HTTP 为核心、web UI 为向导的应用。当有人说到应用服务器时，你可能想到“高负载、企业级特性、事物和队列、多通道通行（HTTP 和更多的协议）”，但现在提供这些需求的基本上都是同一个产品。

### web 容器
在 java 中，web 容器一般就是指 Servlet 容器。Servlet 容器是与 Java Servlet 交互的 web 容器的组件。web 容器负责管理 Servlet 的生命周期、把 URL 映射到特定的 Servlet 、确保 URL 请求拥有正确的访问权限和更多类似的服务。综合来看，Servlet 容器就是用来运行你的 Servlet 和维护它的生命周期的运行环境。

![20180930161346.png](https://i.loli.net/2018/09/30/5bb085e59d02c.png)

### 什么是 Servlet？它们有什么用？
在 java 中，Servlet 使你能够编写根据请求动态生成内容的服务端组件。事实上，Servlet 是一个在 javax.servlet 包里定义的接口。它为 Servlet 的生命周期声明里三个基本方法 —— init()、service() 和 destroy() 。每个 Servlet 都要实现这些方法（在 SDK 里定义或者用户定义）并在它们的生命周期的特定时间由服务器来调用这些方法。

类加载器通过懒加载或者预加载自动地把 Service 类加载到容器里，每个请求都拥有自己的线程，而一个 Service 对象可以同时为多个线程服务。当 Service 对象不再被使用时，它就会被 JVM 当作垃圾回收掉。

### 什么是 ServletContext？它由谁创建
当 Servlet 容器启动时，它会部署并加载所有的 web 应用。当 web 应用被加载时，Servlet 容器会一次性为每个应用创建 Servlet 上下文（ServletContext）并把它保存在内存里。Servlet 容器会处理 web 应用的 web.xml 文件，并且一次性创建在 web.xml 文件里定义的 Servlet、Filter 和 Listener ，同样也会把它们保存在内存里。当 Servlet 容器关闭时，它会卸载所有的 web 应用和 ServletContext ，所有的 Servlet、Filter 和 Listener 实例都会被销毁。

从 java 的文档中可知，ServletContext 定义了一组方法，Servlet 使用这些方法来与它的 Servlet 容器进行通信。例如，用来获取文件的 MIME 类型、转发请求或编写日志文件。在 web 应用的部署文件（deployment describtor）标明“分布式”的情况下，web 应用的每一个虚拟机都拥有一个上下文实例。在这种情况下，不能把 Servlet 上下文当作共享全局信息的变量（因为它的信息已经不具有全局性了）。可以使用外部资源来代替，比如数据库。

### ServletRequest 和 ServletResponse 从哪里进入生命周期？
Servlet 容器包含在 web 服务器中，web 服务器监听来自特定端口的 HTTP 请求，这个端口通常是 80 。当客户端发送一个 HTTP 请求时，Servlet 容器会创建新的 HttpServletRequest 和 HttpServletResponse 对象，并且把它们传递给已经创建的 Filter 和 URL 模式与请求 URL 匹配的 Servlet 实例的方法，所有的这些都使用同一个线程。

request 对象提供了获取 HTTP 请求的所有信息的入口，比如请求头和请求实体。response 对象提供了控制和发送 HTTP 响应的便利方法，比如设置请求头和请求实体。response 对象提供了控制和发送 HTTP 响应的便利方法，比如设置响应头和响应实体（通常是 JSP 生成的 HTML 内容）。当 HTTP 响应被提交并结束后，request 和 response 对象都会被销毁。

### 如何管理 Session？cookie 呢？
当客户端第一次访问 web 应用或第一次使用 request.getSession() 获取 HttpSession 时，Servlet 容器会创建 Session，生成一个 long 类型的唯一 ID （可以使用 session.getId() 获取它）并把它保存在服务器的内存里。Servlet 容器同样会在 HTTP 响应里设置一个 Cookie ，cookie 的名字是 JSESSIONID 并且 cookie 的值是 session 的唯一 ID 。

根据 HTTP cookie 规范（正规的 web 浏览器和 web 服务器必须遵守的约定），在 cookie 的有效期间，客户端（浏览器）之后的每个请求都要把该 cookie 返回给服务器，Servlet 容器会利用带有名为 JSESSIONID 的 cookie 检测每一个到来的 HTTP 请求头，并使用 cookie 的值从服务器内容里获取相关的 HttpSession 。

HttpSession 会一直存活着，除非超过一段时间没使用。可以在 web.xml 文件中设置该时间段，默认时间段是 30 分钟。因此，如果客户端已经超过 30 分钟没有访问 web 应用的话，Servlet 容器就会销毁 Session 。之后的每一个请求，即使带有特定的 cookie ，都再也不会访问到同一个 Session 了，ServletContainer 会创建一个新的 Session 。

另外，在客户端的 session cookie 拥有一个默认的存活时间，这个时间与浏览器的运行时间相同，因此，当用户关闭浏览器后（所以的标签或窗口），客户端的 Session 就会被销毁，重新打开浏览器后，与之前的 Session 关联的 cookie 就再也不会被发送出去了。再次使用 request.getSession() 会返回一个全新的 HttpSession 并且使用一个全新的 session ID 来设置 cookie 。

### 如何确保线程安全？
我们现在已经知道了所有的请求都在共享 Servlet 和 Filter 。它是多线程的并且不同的线程（HTTP 请求）可以使用同一个实例。否则，对每一个请求都重新创建一个实体会耗费很多的资源。同样也要知道，不应该使用 Servlet 或者 Filter 的实例变量来存放任何的请求或者会话范围内的数据，这些数据会被其它 Session 的所有请求共享，这是非线程安全的。