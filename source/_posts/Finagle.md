title: Scala-Finagle
date: 2015-09-14 21:23:32
tags: [Finagle, Scala]
---

#### 基本描述
- 基于RPC, 多协议支持, 运行在JVM上, 统一api构建服务端和客户端
- 并发模型基于Futures, 安全的模块化编程
- 服务端和客户端都发布监控和统计信息

<!-- more -->

#### 使用指导
- 快速开始
- 使用Futures并发编程
  - 类似轻量级线程, 但是少了线程切换的开销
  - 数以百万的并发请求是没有问题的
  - 解耦了操作系统和运行时的调度器 `com.twitter.util.Future`
  - 例子
    - 远程的RPC主机(可能被中断)
    - 长时间的外线程计算(可能会抛出异常)
    - 磁盘IO(读取失败)
  - Futute[T]的改进
    - Empty(pending)
    - Succeeded(with a result of T)
    - Failed(with a throwable)
  - 当Future可用时, 一个callback会被执行
    - `onSuccess { result: T } `
    - `onFailure { cause: Throwable }`
  - 顺序组合
    - 回调是有用的但是代表了一个笨重的api编程模式
    - Future的力量在于组合(**一些操作可以被分成很多的小操作**)
    - 例子
      - 从一个网站上获取一个代表性的缩略图
        - 获取主页
        - 解析页面获取第一个图片地址
        - 获取图片地址
      - 为了完成第三步必须先保证前2步骤的成功执行
    - Future.flatMap
      - flatMap的结果是一个顺序组合的Future的结果

        ```scala
        def fetchUrl(url: String): Future[Array[Byte]]
        def findImageUrls(bytes: Array[byte]): Seq[String]

        val url = "http://www.google.com"
        val f: Future[Array[Byte]] = fetchUrl(url).flatMap { bytes =>
          val images = findImageUrls(bytes)
          if (images.isEmpty)
            Future.exception(new Exception("XXXX"))
          else
            fetchUrl(images(0))

        f onSuccess { image =>
          println(image.size)
        }
        ```
    - 并发组合
      - 上述的代码例子类似于分号间隔的语句, 并于传统的IO耦合在一起
      - 并发组合由Future.collect提供

        ```scala
        def collected: Future[Seq[Array[Byte]]] =
          fetchUrl(url).flatMap { bytes =>
            val fetches = findImagesUrl(byte).map { url => fetchUrl(url) }
            Future.collect(fetches)
          }
        ```
  - 接受失败信息
    - rescue方法: `def rescue[B >: A](f: PartialFunction[Throwable, Future[B]]): Future[B]`
    - The rescue combinator on Future is the dual to flatMap

      ```scala
      def fetchUrl(url: String): Future[HttpResponse]

      def fetchUrlWithRetry(url: String) =
        fetchUrl(url).rescue {
          case exc: TimeoutException => fetchUrlWithRetry(url)
        }
      ```
- Services & Filters
  - finagle中关服务端和客户端抽象的核心, 简单且多用途
  - trait Service[Req, Rep] extends (Req => Future[Rep])
    - 接受请求类型并返回最后的Future[响应]结果
  - service代表了服务端和客户端
    - 一个服务的实例是通过客户端来使用的
    - 一个服务器实现自一个Service
      - 作为一个客户端使用

        ```scala
        val httpService: Service[HttpRequest, HttpResponse] = ...

        httpService(new DefaultHttpRequest(...)).onSuccess { res =>
          println("received response "+res)
        }
        ```
      - 作为服务端使用

        ```scala
        val httpService = new Service[HttpRequest, HttpResponse] {
          def apply(req: HttpRequest) = ...
        }
        ```

  - filters: 可以定义于应用逻辑无关的行为
    - 比如, 一个请求的超时机制, 当一个请求在指定的时间内未完成的话, 超时机制会抛出异常

      ```scala
      abstract class Filter[-ReqIn, +RepOut, +ReqOut, -RepIn]
        extends ((ReqIn, Service[ReqOut, RepIn]) => Future[RepOut])
      ```
    - In most common cases, ReqIn is equal to ReqOut, and RepIn is equal to RepOut

      ```scala
      trait SimpleFilter[Req, Rep] extends Filter[Req, Rep, Req, Rep]
      class TimeoutFilter[Req, Rep](timeout: Duration, timer: Timer) extends SimpleFilter[Req, Rep] {
        def apply(request: Req, service: Service[Req, Rep]): Future[Rep] = {
          val res = service(request)
          res.within(timer, timeout) // Future里面的超时方法
        }
      }
      ```
    - Composing filters and services: Filters and services compose with the andThen method
    - 超时过滤器的例子

      ```scala
      val service: Service[HttpRequest, HttpResponse] = ...
      val timeoutFilter = new TimeoutFilter[HttpRequest, HttpResponse](...)
      val serviceWithTimeout: Service[HttRequest, HttpResponse] =
        timeoutFilter andThen service // 创建一个新的Service
      ```


      ```scala
      val timeoutFilter = new TimeoutFilter[..](..)
      val retryFilter = new RetryFilter[..](..)
      val retryWithTimeoutFilter: Filter[..] =
        retryFilter andThen timeoutFilter
      ```
  - ServiceFactory: Service的获取过程(连接池对象等)

    ```scala
    abstract class ServiceFactory[-Req, +Rep] extends (ClientConnection => Future[Service[Req, Rep]])
    ```

- Servers
  - 实现一个简单的接口

  ```scala
  def serve(
    addr: SocketAddress,
    factory: ServiceFactory[Req, Rep]
  ): ListeningServer
  ```
  - 使用方式: 协议.serve(地址, 服务工厂) `val server = Httpx.serve(":8080", myService); Await.ready(server); // 等待服务器资源释放`

- Clients: Protocol.newClient(...)
  - `def newClient(dest: Name, label: String): ServiceFactory[Req, Rep]`
  - `def newService(dest: Name, label: String): Service[Req, Rep]`
  - Client Modules
    - 所有到达客户端转化的请求都会经过N多模块
      - the client stack manages name resolution and balances requests across multiple endpoints
      - the endpoint stack provides session qualification and connection pooling
      - the connection stack provides connection life-cycle management and implements the wire protocol
    - Module Composition
      - A materialized Finagle client is a ServiceFactory
    - Observability
    - Timeouts & Expiration
    - Request Draining
    - Load Balancer
    - Heap + Least Loaded
    - Power of Two Choices (P2C) + Least Loaded
    - ......
- Names and Naming in Finagle: Finagle uses names to identify network locations
  - Names must be supplied when constructing a Finagle client through ClientBuilder.dest or through implementations of Client
  - Names are represented by the data-type Name comprising two variants
    - `case class Name.Bound(va: Var[Addr])` Identifies a set of network locations
    - `case class Name.Path(path: Path)` Represents a name denoted by a hierarchical path, represented by a sequence of byte strings

- Extending Finagle
