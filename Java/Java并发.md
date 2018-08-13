# Java并发

## 基本概念

### 进程和线程
进程是一个具有一定独立功能的程序关于某个数据集合的一次运行活动，是资源分配的基本单位，而线程是进程内的基本执行单元。

### 同步和异步
这里说的同步和异步是指方法调用方面。同步调用会等待方法的返回，而异步调用会瞬间返回，但是异步调用瞬间返回并不代表你的任务就完成了，他会在后台起个线程继续进行任务。

### 并发和并行
并行是两个任务同时进行，而并发是一会做一个任务一会又切换做另一个任务。

### 临界区
表示一种公共资源或者说是共享数据／代码，可以被多个线程使用，但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。

### 死锁和活锁
死锁是指两个或以上的任务由于竞争资源而造成的一种阻塞现象，而活锁是指两个或以上的任务都可以使用资源，但他们互相谦让，都想让对方先使用资源，由此造成的一种阻塞现象。

## Java线程数量的限制
* 第一个限制在操作系统，操作系统定义了总的最大线程数。
* 第二个限制在JVM，理论上我们能分配给线程的内存除以单个线程占用的内存就是最大线程数。对Java进程来讲，可以大致认为：线程数 = (系统空闲内存 - 堆内存 - 静态方法区内存)／线程栈大小

## Java线程的五种状态

### 新建（New）
实现Runnable接口和继承Thread可以得到一个线程类，new一个线程出来就进入了New状态。

### 可运行（Runnable）
线程对象创建后，其他线程（比如main线程）调用了该对象的start()方法，就把这个对象对应的线程放入可运行线程池中，等待被线程调度程序选中而获取CPU的使用权。

### 运行（Running）
线程调度程序从可运行线程池中选择一个Runnable线程，使其获取CPU时间片，执行程序代码。这是线程进入Running状态的唯一一种方式。

### 阻塞（Blocked）
* 阻塞状态是指线程因为某种原因让出CPU使用权，暂时停止运行。直到线程进入Runable状态，才有机会再次获得CPU时间片。
* 阻塞有三种情况： 
    * 等待阻塞：线程执行o.wait()方法，JVM会把该线程放入等待队列中。
    * 同步阻塞：线程在获取对象的同步锁时，若该同步锁被别的线程占用，JVM会把该线程放入锁池中。
    * 其他阻塞：线程执行Thread.sleep(long ms)或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入Runable状态。

### 死亡（Dead）
* 线程run()、main()方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。
* 死亡的线程不可再次复生。在一个死去的线程上调用start()方法，会抛出java.lang.IllegalThreadStateException异常。

## Java线程的相关方法
```java
thread.run();
// 在当前线程开始运行
thread.start();
// 在新的操作系统线程开始运行

Thread.sleep(long ms);
// 使当前线程进入阻塞，但不释放对象锁，一定时间后后线程自动苏醒进入Runnable状态。

Thread.yield();
// 使当前线程放弃获取的CPU时间片，由Running变为Runnable状态，让OS再次选择线程。用来让相同优先级的线程轮流执行，但并不保证一定会轮流执行，因为让步之后还可能被线程调度程序再次选中。

thread.join();
thread.join(long ms);
// 当前线程阻塞，但不释放对象锁，直到线程thread执行完毕或者ms时间到，当前线程进入Runnable状态。

thread.setDaemon(true);
// 设置当前线程为thread的守护线程，意思是如果当前线程运行结束，则thread也会停止运行。

thread.setPriority(Thread.MIN_PRIORITY);
// 设置线程优先级，Thread类中有3个变量定义了线程优先级：
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

## Java线程同步
Java使用共享内存的方式实现多线程之间的消息传递。因此，程序员需要写额外的代码用于线程之间的同步。

### synchronized方法和synchronized代码块
* 把任意一个非null的对象当作锁。
    * 作用于非静态方法时，锁住的是对象的实例（this）；
    * 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久代（jdk1.8中是metaspace），永久代是全局共享的，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程；
    * 作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。
* 既可以保证可见性，又能够保证原子性。
    * 可见性：通过synchronized或者Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存中。
    * 原子性：要么不执行，要么执行到底。
* synchronized不具有继承性，是一种高开销的操作，需要完成用户态到内核态的切换。
* synchronized在JDK1.5中的优化：
    * 锁消除：即删除不必要的加锁操作。根据代码逃逸技术，如果判断到一段代码中，堆上的数据不会逃逸出当前线程，那么可以认为这段代码是线程安全的，不必要加锁。
    * 锁粗化：将多次连接在一起的加锁和解锁操作合并为一次，即将多个连续的锁改成一个大锁。
    * 自旋锁：当线程在获取轻量级锁的过程中执行CAS操作失败时，就通过自旋来获取重量级锁。
    * 适应性自旋是指：线程如果自旋获取锁成功了，则下次自旋的循环次数会更多；而如果自旋获取锁失败了，则下次自旋的次数就会减少。
    * 偏向锁、轻量级锁、重量级锁：见Java的锁机制——偏向锁／轻量级锁／重量级锁。

### volatile（用于类的成员变量）

#### 可见性
* 可见性就是说一旦某个线程修改了该被volatile修饰的变量，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，可以立即获取修改之后的值。
* 在Java中每条线程都有各自独立的存储空间，还有一个所有线程共享的内存空间。当新建线程时，系统会将共享内存中的所有共享变量拷贝一份到线程自己的存储空间中。接下来该线程在结束前的所有操作都是基于自己的存储空间进行的。因此若一条线程改变了一个共享变量，仅仅改变的是这条线程专属存储空间中的变量值，而此时如果其他线程访问这个变量，访问的还是修改前的值。
* volatile修饰了一个成员变量后，这个变量的读写就会比普通变量多一些步骤：
    * volatile读：当读取一个被volatile修饰的变量时，会直接从共享内存中读，而非线程专属的存储空间中读。
    * volatile写：当被volatile修饰的变量进行写操作时，这个变量将会被直接写入共享内存，而非线程的专属存储空间。
* volatile的附送功能
    * 进行volatile写操作时，不仅会将volatile变量写入共享内存，系统还会将当前线程专属空间中的所有共享变量写入共享内存。
    * 进行volatile读操作时，系统也会一次性将共享内存中所有共享变量读入线程专属空间。
    * 这就意味着，如果普通变量在volatile写操作之前被修改，那么其他线程在volatile读操作之后就能正确读到他们。

#### 指令重排序和happens-before原则
* 指令重排包括编译期重排和运行期重排，是指处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证各个语句的执行顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。
* 若两行之间的某个变量被volatile修饰后，重排序规则会发生变化。在以下情况下，即使两行代码之间没有依赖关系，也不会发生重排序：
    * volatile读
        * 若volatile读操作的前一行为volatile读／写，则这两行不会发生重排序。
        * volatile读操作和它后一行代码都不会发生重排序。
    * volatile写
        * volatile写操作和它前一行代码都不会发生重排序。
        * 若volatile写操作的后一行代码为volatile读/写，则这两行不会发生重排序。

#### happens-before
* 在真实环境下，动作A和动作B的执行顺序是可以通过指令重排而发生变化的，但是你需要保证A和B的可见性，此时用户可以用满足happens-before原则的操作来规避指令重排：
* happens-before定义：
    * 如果操作A happens-before 操作B，那么操作A的执行顺序排在操作B之前，而且操作A的执行结果将对操作B可见。
    * 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行，只要重排序之后的执行结果与按照happens-before关系来执行的结果一致就行。
* happens-before具体规则：
    * 程序次序规则：一个线程内，按照代码顺序，写在前面的操作happens-before写在后面的操作；
    * 锁定规则：一个unlock操作happens-before后面对同一个锁的lock操作；
    * volatile变量规则：对一个volatile变量的写操作happens-before后面对这个变量的读操作；
    * 传递规则：如果操作A volatile 操作B，而操作B volatile 操作C，则可以得出操作A volatile 操作C；
    * 线程启动规则：Thread对象t的t.start()方法happens-before此线程的每个动作；
    * 线程中断规则：对线程对象的interrupt()方法的调用happens-before该线程的代码检测到中断事件的发生；
    * 线程终结规则：Thread的线程对象t中所有的操作都先行发生于t的终止检测，包括t.join()方法返回、t.isAlive()==false等手段可以检测到t的终止；
    * 对象终结规则：一个对象的初始化完成happens-before它的finalize()方法的开始。

### 原子变量和原子操作
* 原子性指的是一组操作必须一起完成，中途不能被中断。
* Java中的原子操作包括：
    * 对基本变量类型（除了double和long）的赋值。
    * 对引用的赋值。
    * 对java.concurrent.Atomic*所有类的操作。
    * 对volatile的long和double变量的赋值。（内部synchronized）
* 读写long和double变量不是原子的，而是需要分成两步（前32位和后32位）。如果一个线程正在修改该long或double变量的值，另一个线程可能只能看到该值的一半（前32位）。为了避免这种情况，需要在用volatile修饰long、double型成员变量。

#### UnSafe类
* Unsafe类能直接原子性的、从硬件级别访问底层操作系统，但实际编码时是不能用的，否则会报异常。
* Unsafe类的功能：
    * 对象实例化：一般我们用new或者反射来实例化对象，Unsafe使用`allocateInstance()`方法可以直接生成对象实例，并且无需调用构造方法和其它初始化方法。
    * 操作类、对象、变量：包括`staticFieldOffset`（静态域偏移）、`defineClass`（定义类）、`defineAnonymousClass`（定义匿名类）、`ensureClassInitialized`（确保类初始化）、`objectFieldOffset`（对象域偏移）等方法，通过这些方法我们可以获取对象的指针，通过对指针进行偏移，我们不仅可以直接修改指针指向的数据（即使它们是私有的），甚至可以找到JVM已经认定为垃圾、可以进行回收的对象。
    * 操作数组：包括`arrayBaseOffset`（获取数组第一个元素的偏移地址）和`arrayIndexScale`（获取数组中元素的增量地址），二者配合使用就可以定位数组中每个元素在内存中的位置。
    * CAS操作：Compare And Swap，比较替换操作是指令级别的操作，它为Java的锁机制提供了一种新的解决办法，比如AtomicXXX等类都是通过该方法来实现的。
    * 线程挂起与恢复：通过`park`方法实现将一个线程进行挂起，调用后，线程将一直阻塞直到超时或者中断等条件出现。`unpark`可以终止一个挂起的线程，让其恢复正常。

#### AtomicXXX类
* 提供了一些原子性的操作，可以避免使用高代价的synchronized。
* `set()`是原子性地修改value值；而`lazySet()`修改的值不会对其他线程立即可见，可以减少不必要的内存屏障，从而提高程序执行的效率。
* `getAndSet()`是原子性地返回旧值，设置新值；`compareAndSet()`是原子性地比较，一致时替换并返回true，不一致时返回false。
* CAS操作常常与自旋锁一起使用，是一种高效率的无锁算法。

### Java的锁机制

#### 可重入锁
* 可重入锁又名递归锁，是指在同一个线程在外层方法获取锁之后，进入内层方法时也会自动获取锁，好处是可以一定程度上避免死锁。
    * ReentrantLock是一个可重入锁，它内部维护了一个计数器，加锁一次计数器+1，加多少次锁就要解对应次数的锁。
    * synchronized也是一个可重入锁。
* 以下代码是一个可重入锁的一个特点，如果不是可重入锁的话，taskB可能不会被当前线程执行，可能造成死锁。

```java
synchronized void taskA() throws Exception {
    Thread.sleep(1000);
    taskB();
}
synchronized void taskB() throws Exception {
    Thread.sleep(1000);
}
```

* synchronized和ReentrantLock的区别：
    * ReentrantLock相对于synchronized有如下特点：可响应中断、可设置限时、公平锁。
    * synchronized适用于资源竞争不激烈的场景，而ReentrantLock适用于资源竞争激烈的场景。
    * synchronized是JVM层面实现的，而ReentrantLock是代码层面实现的。
    * 尽可能使用synchronized而不是ReentrantLock，原因是：简单易懂、编译器自动优化（适应性自旋、锁消除等）。

#### 独享锁／共享锁
* 独享锁是指该锁同时只能被一个线程所持有。
* 共享锁是指该锁可被多个线程所持有。
* Java的ReentrantLock和synchronized都是独享锁。但是Lock的另一个实现类ReadWriteLock的读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读是非常高效的。
* 独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

#### 乐观锁／悲观锁
* 乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
* 悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
* 乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。
* 悲观锁适合写操作多的场景，乐观锁适合读操作多的场景。
* 悲观锁在Java中的使用，就是利用各种锁；乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS实现原子操作的更新。

#### 自旋锁
* 如果持有锁的线程能在很短时间内释放锁资源，那么尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁。
* 好处：减少线程上下文（内核态和用户态）切换的消耗。
* 缺点：循环会消耗CPU。

#### 公平锁／非公平锁
* 公平锁是指多个线程按照申请锁的顺序来获取锁。
* 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。这样有可能会造成优先级反转或者饥饿现象。
* 对于Java的ReentrantLock，可以通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
* synchronized也是一种非公平锁，由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

#### 偏向锁／轻量级锁／重量级锁
* 这三种锁是指锁的状态，并且是针对synchronized的。在Java 1.5通过引入锁升级的机制来实现高效的synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
* 偏向锁是指：偏向于第一个访问锁的线程。如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，则线程是不需要触发同步的，这种情况下，就会给线程加一个偏向锁。而如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，将锁恢复到标准的轻量级锁。
* 轻量级锁是指：当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
* 重量级锁是指：当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

### wait()／notify()
```java
object.wait();
object.wait(long t);
// 当前线程释放对象锁，进入等待队列。依靠其他线程调用该对象的notify()/notifyAll()唤醒或者t时间到自动唤醒。

object.notify();
// 唤醒在此对象监视器上等待的单个线程，选择是任意性的。
object.notifyAll();
// 唤醒在此对象监视器上等待的所有线程。
```
> wait()和notify()必须在synchronized的代码块中使用，因为只有在获取当前对象的锁时才能进行这两个操作，否则会报异常。

### Condition
* Condition与ReentrantLock的关系就类似于synchronized与obj.wait()／obj.signal()。
* con.await()方法会使当前线程等待，同时释放当前锁，当其他线程中使用con.signal()时或者con.signalAll()方法时，线程会重新获得锁并继续执行。或者当线程被中断时，也能跳出等待。
* con.awaitUninterruptibly()方法与con.await()方法基本相同，但是它并不会再等待过程中响应中断。
* con.singal()方法用于唤醒一个在等待中的线程，con.singalAll()方法会唤醒所有在等待中的线程。

### ThreadLocal
* 每一个使用ThreadLocal变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。
* ThreadLocal类中维护一个ThreadLocalMap，用于存储每一个线程的变量副本，该Map中key为线程对象，而value为对应线程的变量副本。

### BlockingQueue
```java
LinkedBlockingQueue<T> queue = new LinkedBlockingQueue<>();
// 创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue。
queue.put(T t);
// 在队尾添加一个元素，如果队列满则阻塞，类似生产者线程。
queue.take();
// 移除并返回队头元素，如果队列空则阻塞，类似消费者线程。
queue.size();
// 返回队列中的元素个数。
```

### PipedInputStream／PipedOutputStream
* 这个类位于java.io包中，是解决同步问题的最简单的办法。
* 一个线程将数据写入管道，另一个线程从管道读取数据，这样便构成了一种生产者/消费者的缓冲区编程模式。
* 只能用于多线程模式，用于单线程下可能会引发死锁。

### Semaphore
```java
Semaphore s = new Semaphore(5);
// 创建一个初始值为5的信号量
s.acquire();
// 申请使用信号量，如果s的值大于0则允许使用，否则阻塞等待其他线程release()。
s.release();
// 释放信号量资源，信号量的值+1，如果有等待的线程则通知它。
```

## JVM同步机制——监视器Monitor
* 三个部分：入口区、拥有区和等待区。入口区和等待区内可能有多个线程，但是任何时刻最多只有一个线程在拥有区。
* 四种操作原语：
    * 进入：线程进入入口区，准备获取监视器，此时如果没有别的线程拥有该监视器，则这个线程拥有此监视器，否则它要在入口区等待；
    * 获取：在入口区和等待区的线程按照某种策略机制被选择可拥有该监视器时的操作；
    * 拥有：线程在它拥有该监视器的时候排他地占有它，从而阻止其它线程的进入；
    * 释放：拥有监视器的线程执行完监视器范围内的代码或异常退出之后，要释放掉它所拥有的此监视器。
* 对临界区的调度原则：
    * 无空等待：一次至多一个线程能够在临界区内；
    * 有空让进：不能让一个线程无限地留在临界区；
    * 择一而入：不能强迫一个线程无限地等待进入临界区；
    * 算法可行：不能因所选的调度策略而造成线程的饥饿，甚至死锁。

## Java线程池

### Executor接口和ThreadPoolExecutor
* 在Java中，线程池的概念是Executor这个接口，具体实现为ThreadPoolExecutor类。对线程池的配置，就是对ThreadPoolExecutor构造函数的参数的配置。

```java
// 五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue);
                          
// 六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory);

// 六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler);

// 七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler);
```
```
使用ThreadPoolExecutor时，一般只使用5个参数的构造函数。
```

* ThreadPoolExecutor构造函数的参数
    * int corePoolSize => 核心线程数最大值
    * int maximumPoolSize => 线程总数最大值
    * long keepAliveTime => 非核心线程闲置超时时长
    * TimeUnit unit => keepAliveTime的单位
    * BlockingQueue<Runnable> workQueue => 任务队列：维护着等待执行的Runnable对象
    * ThreadFactory threadFactory => 创建线程的方式，一般用不到
    * RejectedExecutionHandler handler => 用来抛异常的，默认不需要指定

### 核心线程
* 线程池新建线程的时候，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过corePoolSize，则新建的是非核心线程。
* 核心线程默认情况下会一直存活在线程池中，即使它啥也不干（闲置状态），但如果指定ThreadPoolExecutor的allowCoreThreadTimeOut这个属性为true，那么核心线程如果一直是闲置状态的话，超过一定时间就会被销毁掉。

### TimeUnit
TimeUnit是一个枚举类型，包括如下值：

* NANOSECONDS：1微毫秒 = 1微秒／1000
* MICROSECONDS：1微秒 = 1毫秒／1000
* MILLISECONDS：1毫秒 = 1秒／1000
* SECONDS：秒
* MINUTES：分
* HOURS：小时
* DAYS：天

### workQueue等待队列
* SynchronousQueue：这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，但如果所有线程都在工作时，会新建一个线程来处理这个任务。所以为了避免线程数达到了maximumPoolSize而不能新建线程的错误，使用这个类型队列时，maximumPoolSize一般指定成Integer.MAX_VALUE。
* LinkedBlockingQueue：这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建一个核心线程处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize。
* ArrayBlockingQueue：可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建一个核心线程执行任务，如果达到了，则入队等候，如果队列已满，则新建一个非核心线程执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误。
* DelayQueue：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务。

### 向ThreadPoolExecutor添加任务
通过`ThreadPoolExecutor.execute(Runnable command)`方法即可向线程池内添加一个任务。

### ThreadPoolExecutor的执行策略
当一个任务被添加进线程池时：

* 线程数量未达到corePoolSize，则新建一个核心线程执行任务。
* 线程数量达到了corePoolSize，则将任务移入队列等待：
    * 队列已满，总线程数未达到maximumPoolSize，则新建非核心线程执行任务。
    * 队列已满，总线程数达到了maximumPoolSize，就会抛出异常。

### 常见的四种线程池

#### CachedThreadPool：可缓存线程池（推荐使用）
当线程池大小超过了处理任务所需的线程，就会回收部分空闲（一般是60秒无执行）的线程；当有任务来时，也能添加新线程来执行。

```java
// 创建方法
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

// 源码
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}
```

#### FixedThreadPool：定长线程池
可控制线程最大并发数（同时执行的线程数），超出的线程会在队列中等待。

```java
// 两种创建方法
// nThreads => 最大线程数即maximumPoolSize
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(int nThreads);
// threadFactory => 创建线程的方法
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(int nThreads, ThreadFactory threadFactory);

// 源码
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

#### ScheduledThreadPool：周期线程池
支持定时及周期性任务的执行，用于执行计划任务。

```java
// 创建方法
// nThreads => 最大线程数即maximumPoolSize
ExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(int corePoolSize);

// 源码
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS, new DelayedWorkQueue());
}
```

#### SingleThreadExecutor：单线程化的线程池
只有一个工作线程执行任务，所有任务按照指定顺序执行，即遵循队列的入队出队规则。

```java
// 创建方法
ExecutorService singleThreadPool = Executors.newSingleThreadPool();

// 源码
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()));
}
```

### 线程池的扩展

#### 回调接口
* 我们可以通过实现ThreadPoolExecutor的子类去覆盖以下三个方法：
    * `beforeExecute(Thread t, Runnable r);`
    * `afterExecute(Runnable r, Throwable t);`
    * `terminated();`
* 从而来实现在线程执行前后，线程池退出时的日志管理或其他操作。

#### 拒绝策略
* 有时候，任务非常繁重，导致系统负载太大。在上面说过，当任务量越来越大时，任务都将放到FixedThreadPool的阻塞队列中，导致内存消耗太大，最终导致内存溢出。
* 因此当我们发现线程数量要超过最大线程数量时，我们应该放弃一些任务，有四种放弃策略：
    * AbortPolicy：如果不能接受任务了，则抛出异常。
    * CallerRunsPolicy：如果不能接受任务了，则让调用的线程去完成。
    * DiscardOldestPolicy：如果不能接受任务了，则丢弃最老的一个任务，由一个队列来维护。
    * DiscardPolicy：如果不能接受任务了，则丢弃任务。

