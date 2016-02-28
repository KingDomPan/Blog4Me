title: Scala-Akka
date: 2015-09-13 23:54:23
tags: [Akka, Scala]
---

#### Actor是什么
- Actor引用: actor是以引用的方式暴露给外界的, 内部状态需要被隔离才能Actor模式中获益
- 一个Actor是一个容器, 包含了状态, 行为, 邮箱, 子Actor, 监管策略
  - 状态: 包含一些变量来反应当前所处的状态, 这些数据是有价值的, 必须被保护起来, 状态不一致是致命的
  - 行为: 每次当一个消息被处理时, 消息会与actor的当前的行为进行匹配. 行为是一个函数, 定义了当前消息所要采取的动作
  - 邮箱: 用来处理消息, 消息是从其他actor发送过来的, 每个actor仅有一个邮箱, 默认按照FIFO处理
  - 子Actor: 每个actor都是一个潜在的监管者, 如果创建子Actor来委托处理任务, 那么默认就会自动监管他们. 子actor维护在actor的上下文中, actor可以访问, 对列表的更改是通过context.actorOf()或者停止context.stop()来实现的, 并且这些更改会立刻生效. 实际的操作是在幕后异步方式完成.
  - 监管策略: 用来处理其子actor错误状况的机制, 错误处理由Akka透明进行, 策略是actor系统组织结构的基础, 所以actor一旦被创建就无法修改
  - 当actor停止时, 失败了并且不能用重启来解决, 停止它自己或者被监管者停止, 它会释放它的资源, 将它邮箱中所有未处理的消息放进系统的"死信邮箱"

<!-- more -->

#### 监管与监控
- 当一个下属失败了, 会将自己和自己的所有下属挂起, 然后向自己的监管者发送一个提示失败的消息, 监管者有以下四种选择:
  - 让下属继续执行, 保持下属当前的内部状态
  - 重启下属, 清除下属的内部状态
  - 永久的终止下属
  - 将失败沿着监管树向上传递
  - **一个actor的缺省行为是在重启前终止它的所有下属**, 这种行为可以用Actor类的preRestart来重写; 对所有子Actor的递归重启操作在这个之后执行
- 每个监管者都配置了一个函数, 将所有可能失败的原因翻译成以上四种选择之一. 这个函数并不将失败actor本身作为输入
  - 这种方式不是很灵活, 我们会希望对不同的下属有不同的失败处理行为
  - 但是监管树的作用其实是组建一个递归的失败处理机制, 如果试图在一个层次做太多事情, 那么这个层次就会很复杂
  - akka实现的是一种叫做"父监管"的形式, Actor只能由其它的actor创建, 而顶部的actor是由库来提供的, 每一个创建出来的actor都是由它的父亲所监管
- 重启的意思
  - 当actor在处理消息出现失败时, 失败的原因分出以下三类:
    - 对收到的特定的消息的系统错误, 比如程序错误
    - 处理消息时一些外部资源的临时性失败
    - actor内部状态崩溃了
  - 以下是重启过程中发生的事件的精确顺序
    - actor被挂起
    - 调用旧实例的`supervisionStrategy.handleSupervisorFailing`方法(缺省实现为挂起所有的子actor)
    - 调用旧实例的`preRestart`回调(缺省实现为向所有的子actor发送终止请求并调用`postStop`)
    - 等待所有子actor终止直到`preRestart`最终结束
    - 调用旧实例的`supervisionStrategy.handleSupervisorRestarted`方法(缺省实现为向所有剩下的子actor发送重启请求)
    - 再次调用之前提供的actor工厂创建新的actor实例
    - 对新实例调用`postRestart`
    - 恢复运行新的actor
- 生命周期的监控(One-For-One Strategy vs. All-For-One Strategy)
  - OneForOneStrategy
  - AllForOneStrategy

#### Actor引用, 路径和地址
- actor如何鉴别身份, 在一个可能的分布式系统中如何定位
- 与actor系统的核心概念相关, 固有的树形监管策略, 跨多个网络节点的actor之间透明的通信
- Actor引用是什么?
  - Actor引用是`ActorRef`的子类, 最重要的功能是支持向它所代表的actor发送消息
  - 通过`self`来访问它的标准本地引用, 在发送给其他actor的消息中也缺省包含这个引用
  - 反过来, actor可以通过`sender`来访问到当前消息的发送者的引用
  - 根据actor系统的配置, 支持几种不同的actor引用
    - 纯本地引用使用, 在配置为不使用网络功能的actor系统中. 不能在保证功能的前提下从网络向外传输
    - 支持远程调用, 同一个jvm中的网络功能的actor系统中, 必须包含协议和远程地址信息
    - 本地actor引用有一个子类actor路由. 逻辑结构与第一点一样, 但是发往它的消息会被直接重定向到它的子actor
    - 远程actor引用, 代表可以通过远程通讯访问的actor. 从别的jvm向他们发送消息时, Akka会透明地对消息进行序列化
  - 几个特殊的引用
    - `PromiseActorRef`表示一个`Promise`, 作用是从一个actor返回的响应来完成，它是由 ActorRef.ask 调用来创建的
    - `DeadLetterActorRef`, `DeadLetterActorRef`是死信服务的缺省实现, 所有接收方被关闭或不存在的消息都在此被重新路由
    - `EmptyLocalActorRef`是查找一个不存在的本地actor路径时返回的: 它相当于`DeadLetterActorRef`, 但是它保有其路径因此可以在网络上发送, 以及与其它相同路径的存活的actor引用进行比较, 其中一些存活的actor引用可能在该actor消失之前得到了.
    - 有一个actor引用但是并不代表任何actor, 只是作为根actor的伪监管者存在
    - 在actor创建设施启动之前运行的第一个日志服务是一个伪actor引用, 它接收日志事件并直接显示到标准输出上; 它就是`Logging.StandardOutLogger`

- Actor路径是什么?
  - 树形结构, 类比文件系统, 也可以通过不同的访问路径得到, 类似符号链接, 这就存在转换放到原始路径
  - **一个actor路径包含一个标识该actor系统的锚点, 之后是各路径元素连接起来, 从根到指定的actor; 路径元素是路径经过的actor的名字, 以"/"分隔**
  - Actor路径锚点: 每一条actor路径都有一个地址组件, 描述如何访问到actor的协议和位置, 之后是从根到actor所经过的树节点上actor的名字
    - `akka://my-system/user/service-a/worker1` // 纯本地
    - `akka://my-system@serv.example.com:5678/user/service-b` // 本地或远程
    - `cluster://my-cluster/service-c` // 集群(未来扩展)
    - akka是2.0版本中缺省的远程协议, 主机和端口的理解取决于所使用的传输机制
  - 逻辑路径: 顺着actor的父监管链一直到根的唯一路径称为逻辑actor路径, 这个路径与actor的创建祖先完全吻合, 所以当actor系统的远程调用配置(和配置中路径的地址部分)设置好后它就是完全确定的了
  - 物理Actor路径: 逻辑Actor路径描述一个actor系统内部的功能位置, 基于配置的远程部署意味着一个actor可能在另外一台网络主机上被创建, 即另外一个actor系统中, 这种情况下, 肯定要访问网络. 因此, 每一个actor同时还有一条物理路径, **从实际的actor对象所在的actor系统的根开始的**. 跟其它actor通信时使用物理路径作为发送方引用能够让接收方直接回复到这个actor上, 将路由延迟降到最小
  - 虚拟路径: 未来扩展

- 如何获得Actor引用?
  - 通过创建actor: 一个actor系统通常是在根actor上使用**ActorSystem.actorOf创建actor, 然后使用ActorContext.actorOf从创建出的actor中生出actor树来启动的**, 这些方法返回指向创建新的actor的引用, 每个actor都有父, 自己, 子actor的引用
  - 对actor的拜访查找
    - 通过具体的actor路径来创建actor引用
      - `ActorSystem.actorFor`返回一个未验证的本地, 远程或者集群的actor引用, 向这个引用发送消息或试图观察它的存活状态会在actor系统树中从根开始一层一层从父向子actor发送消息, 直到消息到达目标或是出现某种失败
      - `ActorContext.actorFor`这是在任何一个actor实例中可以用`context.actorFor`访问的, 所返回的actor引用与ActorSystem的返回值非常类似, 但它的路径查找是从当前actor开始的, 而不是从actor树的根开始. 可以用`..`路径来访问父actor
        - `context.actorFor("../brother") ! msg` // 相对路径
        - `context.actorFor("/user/serviceA") ! msg` // 绝对路径
    - 查询逻辑actor数: 使用通配符来进行对多个actor的匹配, 由于匹配的结果不是一个单一的actor引用, 所以类型是`ActorSelection`, 这个类型不完全支持ActorRef的所有操作. `ActorSystem.actorSelection`或`ActorContext.actorSelection`
    - `context.actorSelection("../*") ! msg` // 包括当前actor在内的所有兄弟

#### 与远程部署之间的互操作
- 当一个actor创建一个子actor, actor系统的部署者会决定新的actor是在同一个jvm中或是在其它的节点上
  - 后者的话actor的创建会通过网络连接来到另一个jvm中进行, 结果是新的actor会进入另一个actor系统
  - 远程系统会将新的actor放在一个专为这种场景所保留的特殊路径下
  - 新的actor的监管者会是一个远程actor引用
  - 这时`context.parent`（监管者引用）和`context.path.parent`（actor路径上的父actor）表示的actor是不同的

#### 路径中的地址部分用来做什么？
- 在网络上传送actor引用时, 是用它的路径来表示这个actor的
- 它的路径必须包括能够用来向它所代表的actor发送消息的完整的信息

#### Akka使用的特殊路径
- `/user`是所有由用户创建的顶级actor的监管者, 用`ActorSystem.actorOf`创建的actor在其下一个层次
- `/system`是所有由系统创建的顶级actor(如日志监听器或由配置指定在actor系统启动时自动部署的actor)的监管者
- `/deadLetters`是死信actor, 所有发往已经终止或不存在的actor的消息会被送到这里
- `/temp`是所有系统创建的短时actor(i.e.那些用在ActorRef.ask的实现中的actor)的监管者.
- `/remote`是一个人造的路径, 用来存放所有其监管者是远程actor引用的actor

#### 位置透明性
- 天生的分布式
  - akka中所有的消息都是被设计成分布式的, 所有actor都使用消息进行通信, 所有的操作都是异步的
  - 要实现的优化是**从远程到本地的优化**
- 透明性会被破坏的方式
  - 分布式的设计对可以做的事情做了一些限制, akka所满足的在使用akka的应用程序中也并不一定被满足
  - 最明显的是一条是网络上发送的所有消息都是可序列化的
  - 不那么明显的是这也包括在远程节点上创建actor时用作actor工厂的闭包(i.e. 在Props里)
- 远程调用如何使用？
  - 没有为远程调用设计的api, 完全靠配置来驱动, 配置文件中指定远程部署的actor子树, 可以不修改代码而进行扩展
  - 唯一允许编程来影响远程部署的是`Props`中包含的一个属性, 这个属性可能被设计为一个特定的`Deploy`实例
- 使用路由来进行垂直扩展的标记点
  - 可以在集群中的不同节点上运行一个actor系统的不同部分
  - 还可以通过并行增加actor子树的方法来垂直扩展到多个cpu核上

#### 配置
- ActorSystem是配置消息的唯一消费者, 构建一个ActorSystem系统的时候, 会传入一个Config对象, 不传递的话使用的是ConfigFactory.load()
- 使用的是typesafe Config库
- 读取classpath下的application.conf, application.json, application.properties
- 合并classpath根目录下的reference.conf来组成其内部的默认配置值
- ConfigFactory.load()会合并classpath中所有匹配名称的资源
  - 配置树中区分actor系统

    ```scala
    myapp1 {
      akka.loglevel = WARNING
      my.own.setting = 43
    }
    myapp2 {
      akka.loglevel = ERROR
      app2.setting = "appname"
    }
    my.own.setting = 42
    my.other.setting = "hello"
    ```

#### API
##### Actor
- Props, 是一个用来在创建actor时指定选项的配置类
- 未处理的消息行为, 默认实现是向actor系统的事件流中发布一条akka.actor.UnhandledMessage(message, sender, recipient)
- self代表本身的ActorRef
- sender收到消息的发送者
- supervisorStrategy, 对子actor的监管策略
- context 暴露actor和当前消息的上下文信息
  - actorOf
  - 所属的系统
  - 父监管者
  - 生命周期监控
    - preStart 启动后会立即被执行
    - preRestart 在处理一个消息的时候抛出异常, 那么将被重启
    - postRestart
    - postStop 终止, 保证消息队列被禁止后才运行, 之后发送给该actor的消息都被重定向到**deadLetters**
  - hotswap行为栈(Become/Unbecome)
- 使用DeathWatch进行生命周期的监控, 将自己注册为某个停止的actor的Terminated消息的接收者
  - case terminated('child')
- ! fire and forget 异步发送一个消息并立即返回 也称为tell
- ? 异步发送一条消息并返回一个Future对象, 代表一个可能的回应, 也称为ask, 作为一种模式使用
- sender, 如果不是从actor发送的, 那么sender的默认实例是deadLetters
- 消息转发, forward, 当实现功能类似路由器, 负载均衡器, 备份等的actor会很有用


##### TypedActor
##### EventBus 事件总线, 事件流, 事件处理器, system.event
##### 定时器
- scheduler返回一个akka.actor.Scheduler实例, 这个实例在每个System里面是唯一的, 用来指定一段时间后发生的行为
- 定时任务是使用ActorSystem.MessageDispatcher执行的
- 向actor发送消息或者执行任务的代码, 返回一个Cancellable类型的对象, 使用cancel来取消定时任务的执行, 不会终止正在进行的任务
  - `system.scheduler.scheduleOnce(50 milliseconds, testActor, "foo")` 计划在50mm后向testActor发送foo消息
  - `system.scheduler.scheduleOnce(50 milliseconds) { testActor ! System.currentTimeMills }` 计划50mm后发送当前时间给testActor
  - `val cancellable = system.scheduler.scheduleOnce(0 milliseconds, 50 milliseconds, testActor, "panqd")` 计划0mm后每隔50mm发送panqd给testActor

##### Future
- 用来获取某个并发操作的结果的数据结构, 这个操作通常是由Actor执行或者由Dispatcher直接执行的, 这个结果可以同步或者异步的方式访问
- 为了运行回调, 需要一个ExecutionContext, 与java.util.concurrent.Executor
  - `implicit val ec = ExecutionContext.fromExecutorService(这边是你自己定义的ExecutorSerivce)`
  - `val f = Promise successful "panqd"`
  - `es.shutdown`
- 阻塞方式的调用
  - `actor ? msg`
  - `Await.result(future, timeout.duration).asInstanceOf[String]` // Await.result 或者 Await.ready
- 使用非阻塞时, 类型转换要用mapTo
  - `val future: Future[String] = ask(actor, msg).mapTo[String]`
- 函数式Future
  - map
  - sequence
  - traverse
  - fold
  - reduce
  - 回调
    - onComplete
    - onSuccess
    - onFailure
  - 定义次序 andThen 会为指定的回调创建一个新的Future, 新的Future拥有相同的结果
  - fallbackTo 将2个Futures合并成一个新的Future, 如果第一个失败了, 它将持有第二个Future的成功值

##### 数据流并发
- 类似函数输入输出, 具有行为一致性

##### 容错
- 策略匹配 PartialFunction[Throwable, Directive]
- 缺省的监管机制
  - 如果定义的监管机制没有覆盖抛出的异常,就使用上溯机制
  - 如果某个actor没有定义监管机制, 下列异常将被缺省的处理
    - ActorInitializationException 将终止出错的actor
    - ActorKilledException将终止除错的actor
    - Exception 将重出错的子actor
    - 其他的Throwable将被上溯传给父actor
  - 如果异常一直被上溯到根监管者, 在那儿也会用上述的缺省方式进行处理
  - 监管者默认处理的Exception不会被处理而是会被上述到顶级监管者, 默认的处理策略会杀死所有的子actor, 这样造成出错误的actor都会被杀死. **覆盖preRestart**

##### 派发器
- 在没有为actor做配置的情况下, 一个system将有一个默认的派发器, 这个派发器是fork-join-executor的Dispatcher, 在大多数情况下拥有非常良好的性能
- 为acotr设置派发器, 需要做2件事
  - Props[Actor].withDispatcher("myDispatcher")
- 派发器的类型
  - Dispatcher
    - 可共享性: 无限制
    - 邮箱: 任何, 为每一个actor创建一个
    - 使用场景: 默认派发器, Bulkheading
    - 底层: java.util.concurrent.ExecutorService
  - PinnedDispacther
    - 共享性: 无
    - 邮箱: 同上
    - 使用场景: Bulkheading
    - 底层: akka.dispatch.ThreadPoolExecutorConfigurator, 缺省为一个thread-pool-executor
  - BalancingDispatcher
    - 共享性: 仅对同一类型的actor共享
    - 邮箱: 任何, 为所有的actor创建一个
    - 使用场景: Work-sharing
    - 底层使用: 指定使用"executor"使用"fork-join-executor", "thread-pool-executor"或akka.dispatcher.ExecutorServiceConfigurator的全称
  - CallingThreadDispatcher
    - 共享性: 无限制
    - 邮箱: 任何, 每actor每线程创建一个(需要时)
    - 使用场景: 测试
    - 底层使用: 调用线程
- 邮箱的类型
  - UnboundedMailbox
    - 底层: java.util.concurrent.ConcurrentLinkedQueue
    - 不阻塞, 不绑定
  - BoundedMailbox
    - 底层: java.util.concurrent.LinkedBlockingQueue
    - 阻塞, 绑定
  - UnboundedPriorityMailbox
    - 底层: java.util.concurrent.PriorityBlockingQueue
    - 阻塞, 不绑定
  - BoundedPriorityMailbox
    - 底层: java.util.PriorityBlockingQueue wrapped in an akka.util.BoundedBlockingQueue
    - 阻塞, 绑定
  - 持久邮箱

##### 路由
- 路由actor是将收到的消息路由到目的actor的actor
- 路由actor将消息发送给它所管理的称为'routees'的actor
- 子类的actor
  - akka.routing.RoundRobinRouter: actor均摊分配处理消息
  - akka.routing.RandomRouter: actor被随机处理
  - akka.routing.SmallestMailboxRouter: 选择未挂起的邮箱中消息数最少的routee
  - akka.routing.BroadcastRouter: 将消息转发给所有routees
  - akka.routing.ScatterGatherFirstCompletedRouter: 将消息作为一个future发送给所有的routees. 然后等待回送的第一个结果
- 远程部署Routee
  - 除了将查找到的远程actor作为routee, 你也可以让路由actor将自己创建的子actor部署到一组远程主机上, 这是以round-robin方式执行的
    - 包含akka-remote模块
    - 配置文件中RemoteRouterConfig中, 并附上作为部署目标的结点的远程地
