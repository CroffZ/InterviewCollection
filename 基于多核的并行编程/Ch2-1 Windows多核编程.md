# Ch2-1 Windows多核编程

## 进程

### 进程的概念
进程是一个可以并发执行的具有独立功能的程序关于某个数据集合的一次执行过程，也是操作系统进行资源分配和保护的基本单位。

### 进程的组成部分
* 内核对象：存放进程的管理信息。
* 独立的地址空间：对同一个进程而言，其地址空间内包含了所有代码段和数据段。

### 进程的特点
* 进程具有独立性，也称为异步性。
* 进程是系统中资源分配和保护的基本单位，也是系统调度的独立单位。
* 凡是未建立进程的程序，都不能作为独立单位参与运行。
* 通常每个进程都可以以各自独立的速度在CPU上推进。
* 进程开销较大。

## 线程

### 线程的概念
线程是进程中包含的一个或多个执行单元，是CPU调度的最小单位。

### 线程的组成部分
* 内核对象：存放线程的管理信息。
* 线程栈：每个线程都要维护一个线程栈，它不需要独立的地址空间，用来保存函数的调用历史、参数信息和局部变量。

### Windows单核CPU机器对多线程的支持
用不同线程的轮转调度来实现多线程。

### 线程API的实现层次
* 操作系统层（如WIN32 API）
* 库或运行时环境（MFC、.Net框架）
* 专门的多线程库（pthread）
## 创建线程：三对API

### CreateThread() & ExitThread()
```C++
HANDLE CreateThread (
    LPSECURITY_ATTRIBUTES lpThreadAttributes, // 用于确定子线程的权限和安全值，没有特殊需要时，提供NULL即可。
    DWORD dwStackSize, // 线程栈的大小，如果填0则为默认大小。
    LPTHREAD_START_ROUTINE lpStartAddress, // 线程主函数的函数指针。
    LPVOID lpParameter, // 传递给线程主函数的参数指针。
    DWORD dwCreationFlags, // 标志位，0表示定义线程立即执行，CREATE_SUSPENDED表示创建后先挂起。
    LPDWORD lpThreadId // 输出参数，表示新线程的ID。
);
VOID ExitThread (DWORD dwExitCode);
DWORD WINAPI ThreadFunc (LPVOID data) {    // 线程函数    return 0; // 退出时会自动地隐式调用ExitThread(0);}
```

* 这对API是最基本、最简单的创建／销毁线程API，其他API都通过这对API来创建／销毁线程，但这对API实际用得较少。
* 句柄（HANDLE）理解为指向一个复杂结构的指针，本身占用的内存不大。
* 返回值是当前线程对象的指针。
* 所有LP类型都是对应类型的指针，如LPVOID就是void*。
* DWORD表示双字长整数。

### _beginthreadex() & _endthreadex()
```C++
unsigned long _beginthreadex (    void *security,    unsigned stack_size,    unsigned (*start_address) (void *),    void *arglist,
    unsigned initflag,
    unsigned *threadID
);
void _endthreadex (unsigned retval);
```

* 使用最广泛的创建／销毁线程API，调用上一对API实现，有进一步的封装。
* 参数类型与功能和上一对API没有任何区别。
* 安全性比上一对API更好，因为上下文增加了对多个线程同时调用库函数时内存泄漏和访问冲突的检查和预防。
* 这对API存在于C／C++运行时库的多线程版本中。

### AfxBeginThread() & AfxEndThread()
```C++
CWinThread* AfxBeginThread (
    AFX_THREADPROC pfnThreadProc,
    LPVOID pParam,
    int nPriority = THREAD_PRIORITY_NORMAL,
    UINT nStackSize = 0,
    DWORD dwCreateFlags = 0,
    LPSECURITY_ATTRIBUTES lpSecurityAttrs = NULL);
CWinThread* AfxBeginThread (    CRuntimeClass* pThreadClass,
    int nPriority = THREAD_PRIORITY_NORMAL,
    UINT nStackSize = 0,
    DWORD dwCreateFlags = 0,
    LPSECURITY_ATTRIBUTES lpSecurityAttrs = NULL
);
```

* 用于MFC的一对线程API。

## 获取当前进程／线程信息
```C++
HANDLE GetCurrentProcess();
HANDLE GetCurrentThread();

DWORD GetCurrentProcessId();
DWORD GetCurrentThreadId();
```
## 管理线程
```C++
DWORD SuspendThread (HANDLE hThread); // 挂起线程
DWORD ResumeThread (HANDLE hThread); // 恢复线程
BOOL TerminateThread (HANDLE hThread, DWORD dwExitCode); // 中止线程
```

* 以线程句柄hThread为参数
* 挂起和恢复的实现机制：内核维护一个挂起计数。
* 手动中止线程需要当心内存泄漏。
* 挂起或中止线程的危险性：挂起、中止线程时不会自动释放资源，从而共享资源或锁的占用会导致死锁发生。
## 线程同步

### 用户态机制
互锁函数只能在单值上运行；临界区只能对单个进程中的线程同步，但开销小，速度快。

#### 互锁函数
* 以原子操作的方式修改一个值。
* 相比其它同步（互斥）方式，速度极快，是WinAPI中最快的同步方式。
* 实现时，互锁函数对总线发出硬件信号，防止另一个CPU访问同一内存地址。

```C++
InterlockedIncrement (PLONG); // 使一个LONG变量加1。
InterlockedDecrement (PLONG); // 使一个LONG变量减1。
InterlockedExchangeAdd (PLONG, LONG); // 将参数2加到参数1指向的变量中。
InterlockedExchange (PLONG, PLONG); // 将参数2的值赋给参数1指向的值，返回原始值。
InterlockedExchangePointer (PVOID*, PVOID); // 功能同上。
InterlockedCompareExchange (PLONG, LONG, LONG); // 将参数1指向的值与参数3比较，相同则把参数2的值赋给参数1指向的值，不同则不变。
InterlockedCompareExchangePointer (PVOID*, PVOID, PVOID); // 功能同上。
```

#### 自旋锁／循环锁（Spin Lock）
* 非阻塞锁：由某个线程独占，采用循环锁时，等待线程并不是静态地阻塞在同步点，而是必须不断循环尝试直到获得该锁。
* 用于锁持有时间（受保护资源使用时间）较短的情况。
* 避免在单CPU或单核计算机中使用循环锁，因为此锁占用CPU很大。
* 使用场景：
    * 预计线程等待锁的时间很短，短到比线程两次上下文切换时间要少，则使用自旋锁。
    * 预计线程等待锁的时间较⻓，至少比两次线程上下文切换的时间要⻓，则使用互斥量而不是自旋锁。
    * 如果加锁的代码经常被调用，但竞争情况很少发生时，应该优先考虑使用自旋锁，因为自旋锁的开销比较小，互斥量的开销较大。
* 可以用InterlockedExchange来实现自旋锁：

```C++
BOOL resourceInUse = FALSE;void func() {
    // 不断循环尝试获取锁，直到获取了锁才能进入下一步操作。    While (InterlockedExchange(&resourceInUse, TRUE) == TRUE)        sleep(0);    // 已获取到锁，接下来完成需要执行的操作。    ......
    // 完成操作后释放锁。
    InterlockedLockedExchange(&resourceInUse, FALSE);}
```

#### 临界区对象（CRITICAL_SECTION）
保证临界区内所有被访问的资源不被其它线程访问，直到当前线程执行完临界区代码。

```C++
// 临界区相关API
void InitializeCriticalSection (LPCRITICAL_SECTION);
void EnterCriticalSection (LPCRITICAL_SECTION);
void LeaveCriticalSection (LPCRITICAL_SECTION);
void DeleteCriticalSection (LPCRITICAL_SECTION);

// 使用临界区的例子
void example() {
    CRITICAL_SECTION cs;
    InitializeCriticalSection (&cs);
    try {
        EnterCriticalSection (&cs);
        // Do something
    } finally {
        LeaveCriticalSection (&cs);
    }
    // try-catch机制用于防止代码的异常导致其它线程继续被阻塞。
}
```
	
### 内核态机制
* 适应性广泛，但开销大，速度慢。
* 可用于同步的内核对象：
    * Processes, Threads, Jobs, Files
    * Events
    * Semaphore, Mutexes
    * File change notifications
    * Waitable timers
    * Console input
* 每个内核对象，要么处于触发状态，要么处于未触发状态。

#### 等待函数（Wait Function）
线程可以等待直到内核对象触发了才继续执行。

```C++
DWORD WaitForSingleObject (
    HANDLE hObject, // 内核对象的HANDLE
    DWORD dwMilliseconds // 最大等待时长
);

// 使用WaitForSingleObject的例子
DWORD dw = WaitForSingleObject (hProcess, 5000);
switch (dw) {case WAIT_OBJECT_0:
    // The process terminated.
    break;case WAIT_TIMEOUT:
    // The process did not terminated within 5000 milliseconds.    break;case WAIT_FAILED:
    // bad call to function (invalid handle?)    break;
}

DWORD WaitForMultipleObjects (
    DWORD dwCount, // 等待的事件个数
    CONST HANDLE *phObjects, // 第一个内核对象的HANDLE指针
    BOOL fWaitAll, // TRUE表示等待所有对象变为已通知状态，FALSE表示等待任一个对象变为已通知状态
    DWORD dwMilliseconds // 最大等待时长
);

// 使用WaitForMultipleObjects的例子
HANDLE h[3];h[0] = hProcess1;
h[1] = hProcess2;
h[3] = hProcess3;
DWORD dw = WaitForMultipleObject(3, h, FALSE, 5000);
switch (dw) {case WAIT_TIMEOUT:    // The process did not terminated within 5000 milliseconds.    break;case WAIT_FAILED:    // bad call to function (invalid handle?)    break;
case WAIT_OBJECT_0 + 0:
    // The process identified by h[0] (hProcess1) terminated.    break;case WAIT_OBJECT_0 + 1:    // The process identified by h[1] (hProcess2) terminated.    break;case WAIT_OBJECT_0 + 2:    // The process identified by h[2] (hProcess3) terminated.    break;
}
```

* 等待函数的副作用：
    * 在等待内核对象的过程中，等待函数会将已经触发的内核对象重置为未触发状态。
    * 可能会造成死锁。
* 两个API的返回值都表示等待结束的原因：
    * WaitForSingleObject的返回值有三种：`WAIT_OBJECT_0`（等待的事件发生了）、`WAIT_TIMEOUT`（时间到了）、`WAIT_FAILED`（出错了，比如内核对象在等待过程中被释放等情况）。
    * WaitForMultipleObjects的返回值有：`WAIT_OBJECT_0 + i`（i表示第i个事件发生了）、`WAIT_TIMEOUT`（时间到了）、`WAIT_FAILED`（出错了，比如内核对象在等待过程中被释放等情况）。

#### 事件对象（Event）
* 是Win32提供的最灵活的线程间同步方式。
* 事件的状态包括两种：激发与未激发。
* 事件的分类：
    * 手动设置：只能用程序来手动设置，在需要该事件或者事件发生时，采用SetEvent及ResetEvent来进行设置。
    * 自动恢复：一旦事件发生并被处理后，自动恢复到没有事件状态，不需要再次设置。

```C++
// 创建事件
HANDLE CreateEvent (
    PSECURITY_ATTRIBUTES psa, // 安全属性，同创建线程，通常置NULL
    BOOL fManualReset, // 是否需要手动设置
    BOOL fInitialState, // 事件初始状态
    PCTSTR pszName // 好像是事件名称吧
);
// 设置事件为激发状态
BOOL SetEvent (HANDLE hEvent);
// 设置事件为未激发状态
BOOL ResetEvent (HANDLE hEvent);
```

#### 信号量对象（Semaphore）
```C++
// 创建信号量
HANDLE CreateSemaphore (
    LPSECURITY_ATTRIBUTES lpSemaphoreAttributes, // 安全属性，同创建线程，通常置NULL
    LONG lInitialCount, // 信号量初始计数
    LONG lMaximumCount, // 最大计数
    LPCTSTR lpName // 好像是信号量名称吧
);
// 测试信号量：使用WaitForSingleObject();
// 释放信号量
BOOL ReleaseSemaphore (
    HANDLE hSemaphore, // 要释放的信号量HANDLE
    LONG lReleaseCount, // 要释放的计数数量
    LPLONG lpPreviousCount // 输出参数，返回释放前信号量的计数
);
```

#### 互斥量对象（Mutex）
```C++
// 创建互斥量
HANDLE CreateMutex (
    LPSECURITY_ATTRIBUTES lpMutexAttributes, // 安全属性，同创建线程，通常置NULL
    BOOL bInitialOwner, // 表示当前线程是否是该互斥量持有者
    LPCTSTR lpName // 好像是互斥量名称吧
);
// 测试互斥量：使用WaitForSingleObject();
// 释放互斥量
BOOL ReleaseMutex (HANDLE hMutex);
```

## 其他线程相关函数
### 线程池
* 一些应用需要动态创建线程，如Web服务器。
* 由于创建／销毁线程的开销很大，所以预先将这批线程创建好，需要使用时就分配相应任务，使用完后放回池中等待下次使用。
* 由操作系统提供支持：QueueUserWorkItem()。

```C++
BOOL QueueUserWorkItem(
    LPTHREAD_START_ROUTINE Function, // 所有线程的入口函数
    PVOID Context, // 线程主函数的参数
    ULONG Flags // 控制符，可以用来释放池中的线程或创建新线程，默认置0
);						
```

* 第一次调用此API时，Windows创建一个线程池，其中一个线程执行Function函数，此线程的任务结束后后，将返回线程池，等待新任务。
* Windows内部的调度算法决定处理当前线程工作负载的最佳方式。
### 线程优先级
线程优先级 = 进程优先级 + 线程相对优先级

```C++
Bool SetThreadPriority (
    HANDLE hThread, // 要设置优先级的线程HANDLE
    int nPriority // 要设置的优先级
);
// 返回值为是否设置成功
```

### 处理器亲和
* 当线程被调度执行时，操作系统确定线程运行在哪个处理器上。
* 亲和：线程对其所运行的处理器的有限选择。
    * 例如I/O密集型线程与计算密集型线程的搭配。
    * 指定处理器亲和的效果必须谨慎。

```C++
DWORD_PTR SetThreadAffinityMask (
    HANDLE threadHandle,
    DWORD_PTR threadAffinityMask
);
```

* 线程的AffinityMask必须是所属进程AffinityMask的子集。
* DWORD_PTR是一个二进制串，直接写十六进制常数即可，其中每个bit表示一个核是否使用，希望谁使用，就置相应位为1。
* 设置多个核时，就由系统在这些核中进行调度和切换。

## 线程局部存储（Thread Local Storage，TLS）

### TLS介绍
* 方便的存储线程局部数据的系统。
* 可使用TLS将数据与特定的线程相关联。
* 利用TLS机制可以为进程中的所有线程关联若干个数据，各个线程通过TLS分配的全局索引来访问自己关联的数据。
* 本质上仍然是全局变量。

### Windows的TLS实现
* 定义了一个数组，需要使用时向其申请一个项。
* 早期时，项的数量为64。
* 一旦TLS被启用，此变量将使得每个线程都拥有一份。
* 每个线程有其私有空间，其中的数组与公有空间的数组长度对应。
* 公有空间中将相应元素置为FREE或INUSE：
    * INUSE表示私有空间中的数组上有值。
    * FREE表示私有空间上的位置没有被使用。
    * 默认情况下，所有私有空间都没有被使用。

### 涉及到的API

#### TlsAlloc()：分配一个空闲项，返回对应下标。
```C++
DWORD WINAPI TlsAlloc (void);
```

* 每个进程都维护一个长度为`TLS_MINUMUM_AVAILABLE`的位数组，此位数组用于记录哪个下标对应的项被使用，哪个下标对应的项空闲，TlsAlloc的返回值就是数组的一个下标。
* 初始状态下，此位数组成员的值都是FREE，当调用TlsAlloc时直接遍历数组寻找一个空闲项，直到找到一个值为FREE的成员，然后把找到的成员的值由FREE改成INUSE后，TlsAlloc返回该成员的索引。
* 如果找不到一个为FREE的项，则返回`TLS_OUT_OF_INDEXES`，表示分配失败。
* 当一个线程被创建时，Windows就会在进程地址空间中为该线程分配一个长度为`TLS_MINIMUM_AVAILABLE`的数组，数组成员的值都初始化为0，在内部系统将此数组与该线程关联起来，保证只能在该线程内访问此数组中的数据。

#### TlsSetValue()与TlsGetValue()：对应项的存取
```C++
BOOL WINAPI TlsSetValue (
    __in DWORD dwTlsIndex, // 索引值，表示在数组中的具体位置
    __in_opt LPVOID lpTlsValue // 要设置的值
);

LPVOID WINAPI TlsGetValue (
    __in DWORD dwTlsIndex // 索引值
);
```

* 如果调用成功则返回TRUE，否则返回FALSE。
* 只能改变线程自身的数组，不能设置其它线程的值。
* 存储的值一般为指针。

#### TlsFree()：释放一个项
```C++
BOOL WINAPI TlsFree (
    __in DWORD dwTlsIndex // 索引值
);
```

* 实现时直接将全局的位数组对应位置的INUSE改为FREE。
* 如果试图释放一个未被分配的槽将产生一个错误。

