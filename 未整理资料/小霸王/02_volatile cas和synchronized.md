## 解决每次从主存读取数据效率问题引入高速缓存

1. 引入高速缓存 https://www.processon.com/view/link/5f9ca92c7d9c0803a836f095

2. 又导致了新的问题:处理器缓存数据不一致性问题

   

## 解决缓存一致性问题引入MESI协议

1. 多个处理器高速缓存通过总线相连

   - 处理器01向总线发送read请求读取数据
   - 总线从主存中读取数据给处理器01
   - 如果数据被多个处理器共享则flag标识为s状态
   - 当变量被修改时,处理器01会往总线发送一个invalidate message消息,等待其他处理器回复ack invalidate消息
   - 所有处理器都返回ack后获取数据修改的独占锁,修改数据flag=exclusive,修改数据完成后为modify状态
   - 此时其他处理器中数据为invalidate状态,其他处理器从处理器01的高速缓存或者主存中读取数据

2. 原理图

   ![image-20200713173435805](https://note.youdao.com/yws/api/personal/file/WEB8a27fe3de4d19efb3f1a01d8d88edd81?method=download&shareKey=e0b1ce8d8c9907b892a7805c8661db28)
   
3. 存在问题:多个写请求阻塞



## 解决多个写请求阻塞问题引入写缓冲区和无效队列

1. 优化多个写操作阻塞:
   - 写数据不等待invalidate ack直接写入到写缓冲区中
   - 其他处理器收到invalidate message后直接写入到无效队列中返回ack
   - 处理器01嗅探到invalidate ack消息后从写缓冲区刷新数据到高速缓存中

2. 原理图

   ![image-20200713175530707](https://note.youdao.com/yws/api/personal/file/WEBf292b18f00745214cc904bc476a6a5cb?method=download&shareKey=7acdbd611d953d588064c5a1a41f2f92)

3. 存在问题:
   
   - 有序性:
  - store load重排:处理器1的写操作写入到了写缓存区对处理器2不可见,处理器2读了一份数据则觉得load在前store在后
     - store store:处理器1的第一个写操作发现数据是s状态,写入到写缓冲区;第二个写操作为m状态直接修改,即两个写操作顺序反过来
   - 可见性:
     - 比如处理器1写入到自己的写缓冲器中,处理器2通过总线去read数据读取到的还是旧数据
     - 处理2将validate消息写入到无效队列中,此时处理器2的高速缓存中数据还未标记为invalidate,然后处理器2读取到旧的数据



## 为了解决可见性问题flush和reflush

1. volatile的读和写操作

   - flush:当读取数据时强制将无效队列中invalidate message刷新到高速缓存标记数据为无效重新从其他处理器高速缓存和主存中读取
   - reflush:当写数据时强制将写缓冲区的数据写入到高速缓存中



## 为了解决有序性问题内存屏障

1. volatile读数据
   - 后面loadload屏障:禁止下面普通读和volatile读重排
   
     ```java
     	boolean flag = false;
     	volatile int i = 1;
     
     
     	pulibc void method01(){
     		
     		if(flag){
           // loadload屏障 保证volatile的读操作一定会现在普通读操作之间发生
     			System.out.print("i:"+i);
     		}
     	}
     ```
   
     
   
   - 后面loadStore屏障:禁止下面普通写和volatile读重排
   
2. volatile写数据
   - 前面storestore屏障:禁止上面普通写和volatile写重排序 解决store和store的重排
   - 后面storeload屏障:禁止下面普通读和volatile读/写重排序  解决store和load的重排



## volatile的应用场景

1. 优雅停机isRunning中断线程后修改isRunning=false volatile
2. aqs中state变量保证另外一个线程读时可见
3. CopyOnWriteArrayList写时复制中数组对象通过volatile修饰保证写操作在下次读的时候可见
4. 注册中心心跳



## cas

1. 初始化
   - volatile修饰的value字段
   - unsafe组件获取字段value对应的内存中的地址valueOffset
2. 方法
   - compareAndSet方法底层调用unsafe的compareAndSwapInt方法通过valueOffset地址值获取value实际内存值,然后和期望值比较,如果相等则将目标值赋值给value否则返回false
3. 高阶应用:
   - 对象:atomicReference
   - aba问题:atomicStampedReference
   - 自旋问题:LongAddr,当更新某个cell失败后尝试去更新另外一个cell
4. 应用场景:
   - eureka 注册中心统计心跳次数
   - 线程池中的线程数量和worker线程的数量 ctl  高3位线程状态 线程数量:(2^29)-1
   - concurrentHashMap和concurrentLinkedQueue 基于cas来添加元素



## synchronized

1. 保证同一时刻只有一个线程能执行,保证可见性和原子性

2. 执行流程

   1. java中每个对象分为四部分

      - 对象头:
        - mark word(存储对象的锁信息和hashCode),class metadata Address,指向对应monitor监视器
        - 存储对象数据的类型指针
        - 数组长度
      - 实例数据 比如int 4字节 long8字节
      - 填充部分8倍数

   2. 锁的类型

      - 锁消除:编译器通过JIT逃逸技术分析synchronized是不是只有一个线程获取锁时则不需要monitor entry和monitor exist
      - 锁粗化:当编译器有代码连续多次加锁释放锁时会合并为一个锁
      - 偏向锁:如果大概只有一个线程加锁会给这个线程维护一个bias偏好,后面加锁基于bias不需要cas
      - 轻量级锁:当偏向锁加锁失败,mark word有一个轻量级的指针来直接指向持有锁的线程然后判断是不是自己加的锁
      - 重量级锁:当获取锁时锁被其他线程占用则升级为重量级锁
      - 自适应自旋锁:当线程获取锁失败后进入自旋状态去检查count变量值,不进行线程上下文切换因为锁等待时间会很短,而不是直接进入到entrylist中(从用户态到内核态)默认开启,自旋次数10次,jdk1.6之后加入了自适应的自旋锁通过上次自旋获取锁的时间和次数来解决

3. 原理图:https://www.processon.com/view/link/5f9cb4401e08532b4e8acff8
