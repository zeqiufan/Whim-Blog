# java 基础
1. jdk包含jre，jre中包含jvm。jdk比jre主要多编译器javac，此外还有一些javadoc、jdb（java调试器，形式为命令行）等工具。  
使用jsp部署web需要jdk，因为jsp转换为java servlet时需要编译。
2. java和c++。内存回收自动/手动，类单继承/多继承。
3. 重写方法时返回值可以是原方法返回值的子类。
4. 继承，子类拥有父类的私有属性和方法但无法访问，只能通过非私有方法访问。
5. 多态，两种体现形式：多个子类对父类同一方法多种重写，对接口中同一方法多种实现。
6. StringBuilder线程不安全，没有对方法加同步锁，比StringBuffer高10%左右性能。和StringBuffer同时继承自AbstractStringBuilder。
7. int->Integer 装箱，自动调用valueOf(int)方法；Integer->int 拆箱，intValue方法。上述方法全写在了包装器（如Integer中）。
8. 静态方法可以调用其他静态方法，不能直接调用非静态成员，需要先new一个相关对象。  
***（Q：对static内存加载的详细理解）***  
static修饰的变量存放在栈内存，没有static修饰的属于该类的实例放在堆内存。  
***（Q：堆内存和栈内存的区别）***  

9. 从设计层⾯来说，抽象是对类的抽象，是⼀种模板设计，⽽接⼝是对⾏为的抽象，是⼀种⾏为的规范。
10. uncheck Exception、Exception、RuntimeException   
    Throwable->Exception->RuntimeException
11. 无论是否有异常finally都会执行，如果try或catch中有return那么finally会在返回之前被执行。
12. transient  
    对于不想进⾏序列化的变量，使⽤ transient 关键字修饰。  
    阻⽌实例中那些⽤此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。  
    transient 只能修饰变量，不能修饰类和⽅法。
13. Scanner、BufferedReader
14. IO
    输⼊流和输出流  
    字节流和字符流  
    节点流和处理流 节点流：可以从或向一个特定的地方（节点）读写数据，如FileReader。处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写，构造方法总是要带一个其他的流对象做参数，如BufferedReader  
    InputStream/Reader 输入字节/字符流  
    OutputStream/Writer 输出字节/字符流
15. BIO,NIO,AIO  
    同步阻塞，同步非阻塞，异步非阻塞
    **p52 年轻人回头记得再仔细看看**
16. 浅拷贝，传递引用；深拷贝，创建新对象并复制内容。

# java 集合
1. List、Map、Set
2. ArrayList *    
   底层使用Object数组。  
   add，默认表尾，O(1)；  
   指定位置i则O(n-i)。  
   高效随机访问，实现了RandomAccess接口。  
   list结尾会预留一定的空间容量。  
   线程不安全。  
   Vector也是用Object[]实现的，但是线程安全极其古老。
   ***具体实现可以看一下源码（随机访问、扩容机制）***
3. LinkedList  
   底层使用双向链表（1.6之前为循环链表）。  
   查找i耗时O(n)，具体插入删除O(1)。  
   需要存放前驱后继信息。  
   未实现RandomAccess接口。  
   * 在Arrays的binarySearch()方法中会判断是否为RandomAccess实例，如果是，调⽤ indexedBinarySearch() ⽅法，如果不是，那么调⽤ iteratorBinarySearch() ⽅法
4. HashMap *  
   线程不安全，想要安全使用ConcuerrentHashMap  
   默认打算小为16，每次扩容变成原来的2倍，总是使用2的幂来作为大小  
   数组长度大于64且链表长度大于8之后，链表转化为红黑树  
   ***可以再复习一下红黑树***  
   下标为 (n-1)&hash，hash%length==hash&(length-1)的前提 是 length 是2的 n 次⽅  

5. ConcurrentHashMap  
   线程安全  
   jdk1.7 分段锁  
   jdk1.8 数据结构与HashMap相似，synchronized只锁当前链表或红黑树的首节点  
   ***synchronized锁，回头看一下***  
6. HashSet  
   底层由HashMap实现

# 多线程
1. 进程与线程的异同  
   一个进程多个线程，线程们共享进程的堆和方法区  
   线程包含：程序计数器、虚拟机栈、本地方法栈
2. 程序计数器。改变以达到代码流程控制，另一方面主要用于线程切换后能恢复到正确的执行位置。
3. 虚拟机栈。java方法调用直至执行完成，就对应着栈帧在java虚拟机中的入栈和出栈的过程。栈帧中存放局部变量、操作数栈、常量池引用等信息。  
   操作数栈可理解为java虚拟机栈中的一个用于计算的临时数据存储区  
4. 本地方法栈。与虚拟机栈相似，不过主要用于native方法。
5. 堆与方法区。堆和⽅法区是所有线程共享的资源，其中堆是进程中最⼤的⼀块内存，主要⽤于存放新创建的对象 (所有对象都在这⾥分配内存)；⽅法区主要⽤于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
6. 多线程的好坏。  
   好：资源利用率高  
   坏：内存泄漏、上下⽂切换、死锁  
7. 线程生命周期  
   new、runnable、blocked、waiting、time_waiting、terminated
   初始、运行、阻塞、等待、超时等待、终止  
   刚刚new还没start、就绪/运行、被锁阻塞、需要其他线程的特定动作、相比普通等待到了特定时间就可以返回、已执行完毕  
8. 上下文切换。因一个cpu内核一次只能运行一个线程，需要调度。
9. 死锁。类似操作系统死锁。  
   * 互斥条件：该资源任意⼀个时刻只由⼀个线程占⽤。 
   * 请求与保持条件：⼀个进程因请求资源⽽阻塞时，对已获得的资源保持不放。
   * 不剥夺条件:线程已获得的资源在末使⽤完之前不能被其他线程强⾏剥夺，只有⾃⼰使⽤完毕 后才释放资源。
   * 循环等待条件:若⼲进程之间形成⼀种头尾相接的循环等待资源关系。
10. 线程start()与run()的区别。run()是实际业务逻辑，start()是让线程去干活，至于什么时候run什么时候等待那是多线程调度的事情。
11. synchronized *  
    synchronized 关键字可以保证 被它修饰的⽅法或者代码块在任意时刻只能有⼀个线程执⾏。  
    * 修饰实例⽅法: 作⽤于当前对象实例加锁，进⼊同步代码前要获得 当前对象实例的锁
    * 修饰静态⽅法: 也就是给当前类加锁，会作⽤于类的所有对象实例 ，进⼊同步代码前要获得 当前 class 的锁。static 表明这是该类的⼀个 静态资源，不管 new 了多少个对象，只有⼀份  
    * 修饰代码块：指定加锁对象，对给定对象/类加锁。 synchronized(this|object) 表示进⼊同步代码块前要获得给定对象的锁。 synchronized(.class) 表示进⼊同步代码前要获得 当前 class 的锁
12. a