### 基础篇
### java基础
#### hook（钩子）
hook（钩子）是一种特殊的消息处理机制，它可以监视系统或者进程中的各种事件消息，截获发往目标窗口的消息并进行处理。
#### 反射
- 由于java静态语言，反射是它非常突出的动态相关的一个机制
- 核心：在jvm运行时才动态加载的类或者调用方法或者属性
- 主要用途：开发各种通用框架
- -由于反射会额外消耗一定的系统资源，因此如果不需要动态地创建一个对象，那么就不需要用反射。
另外，反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。
#### 自定义注解
- 不同于普通注释（comment）能存在于源码，而且还能存在编译期跟运行期，会最终编译成一个.class文件，所以注解能有比普通注释更多的功能。
- @Documented @Retention @Target @Inherited
#### session与cookie的区别
- session保存在服务器，客户端不知道其中的信息；cookie保存在客户端，服务器能够知道其中的信息
- session中保存的是对象，cookie中保存的是字符串
- session不能区分路径，同一个用户在访问一个网站期间，所有的session在任何一个地方都可以访问到。而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相是访问不到的。
    ```
    https://www.cnblogs.com/cxrz/p/8529587.html
    ```
### 线程篇

![](https://i.loli.net/2019/01/16/5c3f2f659a2bc.png)

#### 什么叫做线程安全
当多个线程访问同一个对象时，如果不用考虑这些线程在运行时调度和交替执行，也不需要额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获取正确的结果，那么这个对象是线程安全的。
- 总结：代码本身封装了所有必须的正确性保证手短，让调用者无需关心多线程的问题，更无需自己采用任何措施来保证多线程的正确调用。
#### 线程安全的实现方式
 - 互斥同步
   - 互斥是方法，同步是目的。
   - 基本手段synchronized与reentrantLoack(目前二者性能基本相当)
   - 悲观的并发策略（总是认为只要不去做正确的同步措施，就肯定会出现问题）
   - 最大问题：先成阻塞和唤醒带来的性能问题
 - 非阻塞同步
   - 乐观并发策略
   - 先进行操作，如果没有其他线程争用数据，那就操作就成功了。如果有。那就采取其他的补偿措施（最常见就是不断重试，直到成功为止），不需要把线程挂起
#### 逃逸分析
- 基本行为就是分析对象动态作用域。
  - 方法逃逸，线程逃逸。
- 如果别的方法或者线程无法通过任何途径访问到这个对象，则可以进行高校的优化
  - 栈上分配。
  - 同步消除
  - 标量替换
#### 锁优化
 - 减少锁的持有时间
 - 减少锁力度
 - 锁分离
 - 锁粗化
 - 锁消除
#### 1. synchronize 和Lock接口的区别是什么
##### synchronized
原理： 编译之后会在同步快前后分别形成monitoreenter和moitorexit2个字节码指令。这2个自节目都需要一个refernece类型的参数来指明要锁定和解锁的对象。在执行monitorenter指令时候，尝试获取对象的锁，如果没被锁定/已经拥有了对象的锁，则把锁的计数器加1.相应的。执行monitorexit的时候，减1.当计数器为0时候，锁就被释放。如果获取对象的锁失败，就要一直等待。
- 可重入，互斥（保证共享数据在同一时刻植被一个线程使用）锁
 - 用法区别
   - synchronized：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
   - 需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。
   - 如果获取锁的线程需要等待I/O或者调用了sleep()方法被阻塞了，但仍持有锁，其他线程只能干巴巴的等着，这样就会很影响程序效率。 
- ReentrantLock有三个高级功能：
  - 1. 等待可中断
    2. 可实现公平锁
    3. 锁可以绑定多个条件
 总结：


1. Lock是一个接口，而synchronized是关键字。
2. synchronized会自动释放锁，而Lock必须手动释放锁。
3. Lock可以让等待锁的线程响应中断，而synchronized不会，线程会一直等待下去。

4. 过Lock可以知道线程有没有拿到锁，而synchronized不能。
5. Lock能提高多个线程读操作的效率。synchronized能锁住类、方法和代码块，而Lock是块范围内的
因此就需要一种机制，可以不让等待的线程已知等待下去，比如值等待一段时间或响应中断，Lock锁就可以办到
#### 2. Java 锁
##### 锁之前的知识
- 锁类型（乐观锁，悲观锁）
- java线程阻塞的代价
  - java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在用户态与核心态之间切换，这种切换会消耗大量的系统资源
 ##### mark word和对象头
java的对象由三部分组成：对象头，实例数据，填充字节。

非数组对象的对象头由两部分组成：指向类的指针，Mark Word。

数组对象的对象头由三部分组成，比非数组对象多了块用于记录数组长度。
Mark Word的最后2bit是锁标志位，代表当前对象处于哪种锁状态，当Mark Word处于不同的锁状态时，Mark Word记录的信息也有所不同。
- 01 未锁定/偏向锁
- 00 轻量锁
- 10 重量锁

 ##### java中的锁
 
锁从宏观上分类，分为悲观锁与乐观锁。

##### 锁优化


引入偏向锁的目的和引入轻量级锁的目的很像，他们都是为了没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。
- 偏向锁：乐观锁，加锁、解锁不需要额外的消耗，性能接近执行非同步方法。缺点是锁竞争过程中，锁撤消会带来消耗。适用于只有一个线程访问同步块的场景。
  - 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01，确认为可偏向状态。
    
    - 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤5，否则进入步骤3。
    
    - 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行5；如果竞争失败，执行4。
    
    - 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the word）
    
     - 执行同步代码。

- 轻量级锁：乐观锁，竞争的线程不会阻塞，提高了程序的响应速度。缺点是CAS自旋带来的CPU消耗。适用于追求响应时间，锁占用时间很短的场景。
     -   在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为 Displaced Mark Word。
     - 拷贝对象头中的Mark Word复制到锁记录中；
    
    - 拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向object mark word。如果更新成功，则执行步骤4，否则执行步骤5。
    
    -  如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如图所示
    - 如果这个更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，轻量级锁就要膨胀为重量级锁，锁标志的状态值变为“10”，Mark Word中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。 而当前线程便尝试使用自旋来获取锁，自旋就是为了不让线程阻塞，而采用循环去获取锁的过程
 
- 重量级锁：悲观锁，优点是线程竞争不使用自旋，不消耗CPU资源。缺点是线程阻塞，响应时间缓慢。适用于追求呑吐量，锁占用时间较长的场景。
![](https://i.loli.net/2019/01/01/5c2b6717caf83.png)


- 锁优化思路
  - 减少锁持有时间
  - 减小锁粒度
   - 它的思想是将物理上的一个锁，拆成逻辑上的多个锁，增加并行度，从而降低锁竞争。它的思想也是用空间来换时间；
   - ConcurrentHashMap
：Segment继承自ReenTrantLock，所以每个Segment就是个可重入锁，每个Segment 有一个HashEntry< K,V >数组用来存放数据，put操作时，先确定往哪个Segment放数据，只需要锁定这个Segment，执行put，其它的Segment不会被锁定；所以数组中有多少个Segment就允许同一时刻多少个线程存放数据，这样增加了并发能力。

java中很多数据结构都是采用这种方法提高并发操作的效率：

ConcurrentHashMap
java中的ConcurrentHashMap在jdk1.8之前的版本，使用一个Segment 数组
  - 锁分离
  - 锁粗化（常见for循环）
  - 锁消除

#### 3. synchronized 
- 一个synchronized块2个部分：1.锁对象的引用以及这个锁保护的代码块。
- 锁范围越小越好
- synchronized不具备继承性
#### 4.线程池
 ![](https://i.loli.net/2018/08/28/5b853e2588900.png)
##### 核心类：ThreadPoolExecutor
- 重要的参数
  - corePoolSize 核心池大小。初值为0.当任务来临的时候创建一个线程，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中
  - workQueue 阻塞队列，用来存储等到执行的任务，线程的排队策略
  ```
  ArrayBlockingQueue;
  LinkedBlockingQueue;
  SynchronousQueue;
  ```
  - 3钟线程池类型
  - 饱和策略
 ```
AbortPolicy：直接抛出异常，默认策略；
CallerRunsPolicy：用调用者所在的线程来执行任务；
DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
DiscardPolicy：直接丢弃任务； 
当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。
 ```
 - Executors创建的几种类型线程池
    - newFixedThreadPool
    - newSingleThreadExecutor
    - newCachedThreadPool
  - 监控线程池状态
    - 可以用ThreadPoolExecutor的以下方法：getTaskCount()，getCompletedTaskCount()等
  - 配置线程池考虑的因素
    - 任务执行时间长短
    - 任务的性质（io/cpu密集）
      - CPU密集型：尽可能少的线程
      - IO密集型：尽可能多的线程
    - 任务优先级
 - 导出excel 占用大量tomcat线程。
##### hashMap
- 为什么线程不安全
  - v 
##### cas算法
 - compare and swap，比较并替换
 - 三个参数：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false
 - 缺点明显：aba问题：如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性
 - CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成所谓的数据不一致问题。
 - 依赖于unsafe类（全是native），c的指针一样操作内存。提供了获取内存地址，内存页大小等操作
 - 基于三种方法，compareAndSwapInt等
 ##### 内存模型
  - 原子性，可见性，有序性。
 #### java 内存模型
 - 在Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model，JMM）来屏蔽各个硬件平台和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果
 - happens-before原则：
 - Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。它是判断数据是否存在竞争，线程是否安全的主要依据
   - 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
   - 锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作
   - volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
   - 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
- volatile关键字
  - 可见性，当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。
  - 有序性，禁止进行指令重排序。
  - volatile主要用在多个线程感知实例变量被更改了场合，从而使得各个线程获得最新的值。它强制线程每次从主内存中讲到变量，而不是从线程的私有内存中读取变量，从而保证了数据的可见性。
  -  synchronized 块相比，volatile 优点是所需的编码较少，并且运行时开销也较少， 不会引起线程阻塞
  -   对变量的写操作不依赖于当前值。 (count++这种就不行了)
    该变量没有包含在具有其他变量的不变式中（Invariants，例如 “start <=end”）。
    我的理解是， 这两个条件都是因为volatile不能提供原子性导致的， 如果多线程执行的一个操作不是原子性的， 使用volatile时就一定要慎重。
  
#### 
#### atomic与volatile对比
## jvm
#### java运行过程
- 编译
- 类加载
- 执行
#### 类加载过程
- 加载
  - 编译完成的.class文件加载到内存中，供程序使用，用到的就是类加载器classLoader
- 验证
  - 确保class文件的字节流中包含的信息符合当前虚拟机的要求(确保)
    - 文件格式验证
    - 元数据验证
    - 字节码验证
- 准备
  - 为类变量分配内存并设置类变量的初始值
- 解析
  - 把类中的符号引用转换为直接引用。
- 初始化
  - 为类的静态变量赋予正确的初始值，上述的准备阶段为静态变量赋予的是虚拟机默认的初始值
 
 整个流程图
![](https://i.loli.net/2018/09/06/5b91133598a2e.png)
#### 类加载器
 比较2个是否相等，只有在这2个类是由同一个类加载器加载的前期下才有意义。
 - 类加载器
    - 启动类加载器（bootStrap classLoader）,javahome/lib下的
    - 扩展类加载器（extension classloader），javahome/lib/ext
    - 应用程序加载器（Application classLoader）
- 双亲委派模型过程：
  - 如果一个类加载器收到了类加载的过程。它不会自己加载。给父级完成。每一层都是如此。只有当父级反馈完成不了。子级才自己加载。
  - 好处：保证核心类。只被顶端加载器加载。java体系不会被破坏
#### oom相关
- 产生oom的区域
  - 除开程序计数器外。
     - java堆溢出，java.lang.OutOfMemoryError:Java heap spaces，对象创建太多。
     重点是要分清楚是内存泄漏，内存溢出（所谓的定位问题）。如果是内存泄漏。通过工具查看对象到gc root的引用链。定位代码位置
如果是内存溢出。增加堆内存（-xmx,-xms）
  - 虚拟机栈和本地方法栈溢出
     -  如果线程请求的栈深度大于虚拟机最大深度。stackoverFlowError
     -  如果虚拟机在扩展栈时候无法申请到足够的内存。OutOfMemoryError
  - 方法区和运行时常量溢出
    - permGen space
    - 方法区用于存放class的相关信息，类名，访问修饰符，常量池等。方法区溢出也是一种常见的内存溢出异常，一个类如果要被垃圾收集器回收，判定条件是很苛刻的。在经常动态生成大量Class的应用中，要特别注意这点。
### 并发编程的一些问题
#### 如何减少上下文切换？
- 让步式上下文切换，指执行线程主动释放CPU，与锁竞争严重程度成正比，可通过减少锁竞争和使用CAS算法来避免
- 抢占式上下文切换，；后者是指线程因分配的时间片用尽而被迫放弃CPU或者被其他优先级更高的线程所抢占，一般由于线程数大于CPU可用核心数引起，可通过适当减少线程数
#### 避免死锁
- 避免一个线程同时获得多个锁
- 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源
- 尝试使用定时锁，使用lock.tryLock(timeout)来替代使用内部锁机制
#### 如何解决资源限制
- 硬件：集群
- 软件：可以考虑使用资源池将资源复用。比如使用连接池将数据库和Socket复用，或者在调用对方webservice接口获取数据时，只建立一个连接。
### 其他技术
#### 网络
##### get/post 区别
```
https://www.cnblogs.com/logsharing/p/8448446.html
```
#### nginx
#### activemq
1. jms
2. 消息5种类型
3. 如何保证消息队列高可用？如何保证消息不被重复消费？
4. 如何保证消息不丢失

 答：消息系统一般都会采用持久化机制。
 
 ```
 参考：
 https://www.jianshu.com/p/43cd33dc96af
 ```


## 









