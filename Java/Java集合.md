# Java集合

## Java常用集合
* Collection接口：代表一个对象集合。
* List接口：继承Collection接口，存放可重复、有序的元素。
* Queue接口：继承Collection接口，模拟队列操作，采用先进先出的方式组织数据，但优先队列PriorityQueue是一个例外。Deque接口继承Queue接口，模拟双端队列，可以被作为栈来使用。
* Set接口：继承Collection接口，存放非重复、无序的元素，使用`equals()`方法比较两个对象是否相同，如果返回true，则两个对象的`hashCode()`返回值也应该相同。
* Map接口：没有继承Collection接口，它可以把键（key）映射到值（value）的对象，键不能重复。

### Collection和Collections
* Collection是代表集合的接口，List、Set等集合接口都继承它（Map接口除外）。
* Collections是工具类，提供了排序、同步等很多实用方法。
    * 排序：`Collections.sort(Collection collection)`
    * 同步List：`Collections.synchronizedList(List list)`
    * 同步Map：`Collections.synchronizedMap(Map map)`
    * 同步Set：`Collections.synchronizedSet(Set set)`
* Collections的同步实现是指自身实现List／Map／Set的接口，然后在内部维护一个对应的List／Map／Set的实例，synchronized地调用这些实例来实现接口。

### Comparable和Comparator
* Comparable接口：只包含一个`compareTo()`方法，这个方法可以个给两个对象排序。具体来说，它返回负数、0、正数来表明输入对象小于、等于、大于自身对象。
* Comparator接口：包含`compare()`和`equals()`两个方法。`compare()`方法用来给两个输入参数对象排序，返回负数、0、正数表明第一个参数对象是小于、等于、大于第二个参数对象。`equals()`方法需要一个对象作为参数，它用来决定输入参数是否和Comparator相等。只有当输入参数也是一个Comparator并且输入参数和当前Comparator的排序结果是相同的时候，这个方法才返回true。

## 迭代器

### Iterator和Enumeration
* Enumeration的速度是Iterator的两倍，也使用更少的内存。
* Iterator更加安全，因为当一个集合正在被遍历的时候，它会阻止其它线程去修改该集合。
* Iterator允许调用者从集合中移除元素，而Enumeration不能做到。

### Iterator和ListIterator
* Iterator可以遍历Set和List，而ListIterator只能遍历List。
* Iterator只能前向遍历，而ListIterator可以双向遍历。
* ListIterator从Iterator接口继承，添加了一些额外的功能，比如：添加一个元素、替换一个元素、获取前面或后面元素的索引位置。

### fail-fast和fail-safe
* fail-fast机制
    * 在遍历一个集合时，当集合结构被修改，会抛出ConcurrentModificationException。
    * 实现方法：比较modcount和expectModcount，如果不同则抛出异常。
* fail-safe机制
    * 将原集合复制后再开始遍历，任何对原集合的修改不会影响到复制后的集合，因此不会抛出ConcurrentModificationException。
    * 存在两个问题：需要复制集合，产生大量的无效对象，开销大；无法保证读取的数据是目前原始数据结构中的数据。
* Java.util包中的所有集合类都被设计为fail-fast的，而java.util.concurrent中的集合类都为fail-safe的。

## List接口
List接口继承Collection接口，存放可重复、有序的元素。

### ArrayList和Array
* Array可以包含基本类型和对象类型，ArrayList只能包含对象类型。
* Array大小是固定的，ArrayList的大小是动态变化的。
* ArrayList提供了更多的方法和特性，比如：addAll，removeAll等。

### ArrayList和LinkedList
* ArrayList是基于索引的数据接口，它的底层是数组，可以以O(1)时间复杂度对元素进行随机访问。
* LinkedList是以双链表的形式存储它的数据，查找的时间复杂度是O(n)，但插入和删除元素的时间复杂度是O(1)，因为当元素被添加到集合任意位置的时候，不需要像ArrayList那样重新计算大小或者是更新索引。

### ArrayList和Vector
* 两者都继承了抽象类AbstractList，内部实现都是数组，迭代器都是fast-fail的，都允许存储null。
* Vector是线程安全的，而ArrayList是非线程安全的。然而如果希望在迭代的时候对列表进行改变，应该使用CopyOnWriteArrayList。
* 需要扩容时，ArrayList扩展到原来的1.5倍大小，Vector扩展到原来的2倍大小，而且Vector可以自由设置扩展的比例。

### Vector和CopyOnWriteArrayList
* Vector是通过给`add()`、`remove()`、`set()`、`get()`等方法加synchronized来进行线程同步，但是没有控制到遍历操作，所以Vector也不能线程安全的遍历。
* CopyOnWriteArrayList是java.util.concurrent包中的一个实现了List接口的类，CopyOnWrite的意思是在对List进行修改的时候复制一份List并在这份List上进行修改，最后将原List的引用指向修改后的List。遍历也是一样的思想。
* CopyOnWriteArrayList使用ReentrantLock和volatile实现同步，相对于Vector的优势是并发读的效率高，但缺点在于读的可能不是最新值，而且需要创建新数组，会占用额外空间。

### Stack
* Stack继承Vector，它实现了一个标准的后进先出的栈。
* Stack只定义了默认构造函数，用来创建一个空栈。
* Stack除了包括由Vector定义的所有方法外，还定义了栈需要的方法：`boolean empty()`、`Object peek()`、`Object pop()`、`Object push(Object element)`、`int search(Object element)`。

## Queue接口和Deque接口
* Queue接口模拟队列，采用先进先出的方式组织数据，但优先队列PriorityQueue是一个例外。
* Deque接口继承Queue接口，模拟双端队列，可以被作为栈来使用。

### LinkedList
* Java中，LinkedList实现了Deque接口，可以作为一个双端队列来使用。
* 选择LinkedList来实现队列的原因是：LinkedList进行插入、删除操作效率较高。

### PriorityQueue
* PriorityQueue实现了Queue接口，基于堆实现，是一个非FIFO队列，没有容量限制，可以自动扩容，初始化时可以指定初始大小。
* 在实例化优先队列时，可以在构造函数中提供Comparator比较器，然后队列中的项目顺序将根据提供的比较器来决定。如果没有提供比较器，则默认将使用Collection的自然顺序（Comparable）来排序元素。
* 优先队列中不允许存null元素。
* 如果有多个对象拥有相同的优先级，则它们出队的顺序是随机的。
* 优先队列是非线程安全的，所以Java提供了PriorityBlockingQueue（实现BlockingQueue接口）用于Java多线程环境。

### ArrayDeque
* ArrayDeque实现了Deque接口，基于可变数组实现，没有容量限制，可以自动扩容，初始化时可以指定初始容量，默认为16。
* ArrayDeque中不允许存null元素。
* ArrayDeque底层使用环形数组存储元素，使用了两个索引表示当前数组的状态，分别是head和tail。其中head是头部元素的索引，tail是尾部元素的下一位的索引。因为是环形数组，所以索引0是索引length-1的下一位。
* ArrayDeque对数组的大小（即队列的容量）有特殊的要求，必须是2^n。这是因为在容量保证为2^n的情况下，仅通过按位与运算`(tail+1)&(elements.length-1)`就可以完成环形索引的计算，而不需要进行边界的判断，在实现上更为高效。
* 在每次添加元素后，如果head索引和tail索引相遇，则说明数组空间已满，需要进行扩容，扩容后的数组容量是原来的两倍，这也可以满足容量必须是2的幂次方的要求。
* ArrayDeque是非线程安全的，当多个线程同时使用的时候，需要手动同步。

### ConcurrentLinkedQueue
ConcurrentLinkedQueue是一个基于链表实现的线程安全的无界队列，它使用非阻塞的方式实现线程安全，即使用自旋+CAS指令。

### ArrayBlockingQueue和LinkedBlockingQueue
* 这两个队列都实现了BlockingQueue接口，是阻塞队列，可以阻塞添加和阻塞删除。
    * 阻塞添加：当阻塞队列元素已满时，队列会阻塞加入元素的线程，直队列元素不满时才重新唤醒线程执行元素加入操作。
    * 阻塞删除：在队列元素为空时，删除队列元素的线程将被阻塞，直到队列不为空再执行删除操作。
* ArrayBlockingQueue是一个用数组实现的有界阻塞队列，通过可重入锁ReentrantLock和条件队列Condition实现同步，所以ArrayBlockingQueue中的元素存在公平访问与非公平访问的区别。
    * 公平访问：被阻塞的线程可以按照阻塞的先后顺序访问队列，即先阻塞的线程先访问队列。
    * 非公平访问：当队列可用时，阻塞的线程将进入争夺访问资源的竞争中，谁先抢到谁就执行，没有固定的先后顺序。
* LinkedBlockingQueue是一个用链表实现的有界阻塞队列，其大小默认值为Integer.MAX_VALUE，所以在实例化时建议手动设置大小，避免因队列容量过大而造成内存溢出。与ArrayBlockingQueue不同的是，LinkedBlockingQueue使用了takeLock和putLock这两个ReentrantLock来实现同步，也就是说，添加和删除操作不是互斥操作，可以同时进行，这样可以大大提高吞吐量。

## Map接口
Map接口没有继承Collection接口，其中数据以无序键值对形式存在，键不可重复，值可以重复。

### HashMap
* HashMap在静态内部类Map.Entry中存储key-value对，使用每个对象的`hashCode()`和`equals()`方法计算其对应Hash值，解决冲突的方法是链表探测法，用LinkedList实现。
* HashMap的API调用流程：
    * 调用put方法时，HashMap先调用key对象的`hashCode()`来算出其Hash值，再到数组对应其Hash值的位置的链表里查找，如果entry不存在则会创建一个新的entry保存，而如果entry存在则使用`equals()`方法来比较二者是否相同，如果不同就会覆盖原来的entry。
    * 调用get方法时，HashMap先调用key对象的`hashCode()`来算出其Hash值，再到数组对应其Hash值的位置的链表里查找，使用`equals()`方法找出正确的entry并返回它的值。
* HashMap的容量、负荷系数、阀值和rehash
    * HashMap默认的初始容量是32，负荷系数是0.75。HashMap的容量总是2的幂，因为这样才能使用与运算来计算元素下标：`hashcode & (length-1)`。
    * 阀值是为负荷系数乘以容量，无论何时我们尝试添加一个entry，如果map的大小比阀值大的时候，HashMap会对map的内容进行rehash，使用更大的容量来存储。
    * 所以如果预先知道需要存储大量的key-value对，比如缓存从数据库里面拉取的数据，应该使用正确的容量和负荷系数来初始化HashMap。
* HashMap线程不安全的体现：
    * rehash死循环：Java中HashMap解决冲突的办法是用链表，在rehash的时候需要把链表转移，转移时会逆序，多线程情况下会导致单链表的死循环。
    * fail-fast机制：使用迭代器遍历HashMap过程中，如果有其他线程修改了HashMap，那么会抛出ConcurrentModificationException。
* HashMap在JDK1.8中的优化：HashMap采用链表法解决冲突，但当冲突过多使得链表过长时（默认是链表长度大于8时），把链表换成红黑树。

### HashMap和HashTable
* HashMap和HashTable都实现了Map接口，都是键值对保存数据的方式。
* HashMap允许key和value为null，而HashTable不允许。
* HashMap不是线程安全的类，而HashTable是线程安全的类。
* HashMap提供对key的Set进行遍历，因此它是fail-fast的，而HashTable提供对key的Enumeration进行遍历，它不支持fail-fast。
* HashTable被认为是个遗留的类，应该使用ConcurrentHashMap代替它。

### HashTable和ConcurrentHashMap
* HashTable线程安全策略的实现代价大，简单粗暴，`get()`/`put()`等所有相关操作都是synchronized的，相当于给整个哈希表加了一把大锁。多线程访问的时候，只能同时有一个线程访问或操作该对象，其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。
* ConcurrentHashMap采用了非常精妙的"分段锁"策略，将原来的数组分成多个Segment，一个Segment就是一个子哈希表，每个Segment里维护了一个HashEntry数组，因此对于不同Segment的数据进行操作是不用考虑锁竞争的。

### LinkedHashMap
* HashMap的迭代顺序不是元素插入顺序，而LinkedHashMap通过维护一个存Entry<K,V>的双向链表，从而保证元素迭代的顺序，该迭代顺序默认是插入顺序，也可以是访问顺序，缺点是增加了时间和空间上的开销。
* LinkedHashMap继承HashMap，所以跟HashMap一样允许key和value为null，而且也一样不是线程安全的类。

### TreeMap
* TreeMap继承于AbstractMap，实现了NavigableMap接口，它可以确保集合元素处于排序状态，使用树形结构（算法书中的红黑树）实现，要求其中的元素是可排序的，有两种排序方式：
    * 自然排序（Comparable接口），也是默认排序。
    * 比较器排序（Comparator接口），构造方法中作为参数传入。
* TreeMap基本操作的时间复杂度为O(log(n))。
* TreeMap不是线程安全的，它的迭代器是fail-fast的。

### EnumMap
* EnumMap是一个与枚举类一起使用的Map实现，其中所有key都必须是指定枚举类的枚举值，创建EnumMap时必须显式或隐式指定它对应的枚举类。
* EnumMap在内部以数组形式保存，根据key的自然顺序（即枚举值在枚举类中的定义顺序）来维护key-value对的次序，当程序通过`keySet()`、`entrySet()`、`values()`等方法来遍历EnumMap时即可看到这种顺序。
* EnumMap不允许使用null作为key，但允许使用null作为value，如果试图使用null作为key将抛出NullPointerException异常。

### IdentityHashMap
* IdentityHashMap使用Hash算法实现Map接口，但它的key可以重复。
* 在正常的Map实现中（如HashMap），判断两个键k1和k2相等的条件是：`(k1==null ? k2==null : e1.equals(e2))`；而在IdentityHashMap中，判断两个键值k1和k2相等的条件是`k1==k2`。

### Properties
* Properties继承HashTable，表示一个持久的属性集，属性列表中每个key及其value都是一个String。
* Properties被许多Java类使用，例如获取环境变量时它是`System.getProperties()`方法的返回值。

## Set接口
Set接口继承Collection接口，存放非重复、无序的元素，使用`equals()`方法比较两个对象是否相同，如果返回true，则两个对象的`hashCode()`返回值也应该相同。

### HashSet
* HashSet实现了Set接口，不保证迭代顺序，特别是不保证该顺序恒久不变，允许存放null元素，基本操作的时间复杂度为O(1)。
* HashSet底层使用一个HashMap来保存元素，存入HashSet的元素都存到HashMap的Key中，而value是一个统一值：`private static final Object PRESENT = new Object();`。
* HashSet不是线程安全的，它的迭代器是fail-fast的。

### LinkedHashSet
* LinkedHashSet跟LinkedHashMap一样以元素的插入顺序遍历其中的元素，它继承HashSet，所以跟HashSet一样允许存放null元素，也一样不是线程安全的类，基本操作的时间复杂度也是O(1)。
* LinkedHashSet底层使用一个LinkedHashMap来保存元素，存入LinkedHashSet的元素都存到LinkedHashMap的Key中，而value是一个统一值：`private static final Object PRESENT = new Object();`。

### TreeSet
* TreeSet是SortedSet接口的唯一实现类，它可以确保集合元素处于排序状态，使用树形结构（算法书中的红黑树）实现，要求其中的元素是可排序的，有两种排序方式：
    * 自然排序（Comparable接口），也是默认排序。
    * 比较器排序（Comparator接口），构造方法中作为参数传入。
* TreeSet底层使用一个TreeMap来保存元素，存入TreeSet的元素都存到TreeMap的Key中，而value是一个统一值：`private static final Object PRESENT = new Object();`。因此TreeSet跟TreeMap一样，基本操作的时间复杂度为O(log(n))，也一样线程不安全，迭代器是fail-fast的。

### EnumSet
* EnumSet是一个与枚举类一起使用的Set，其中所有元素都必须是指定枚举类的枚举值，创建EnumSet时必须显式或隐式地指定它对应的枚举类。
* EnumSet跟EnumMap一样是有序的，根据枚举值在枚举类中的定义顺序来维护Set元素的顺序。
* EnumSet在内部以位向量的形式存储，因此EnumSet对象占用内存很小，而且运行效率很好。
* EnumSet集合不允许加入null元素，如果试图插入null元素将抛出NullPointerException异常。

## Best Practice
* 根据需要选择正确的集合类型。比如，如果指定了大小，我们会选用Array而非ArrayList；如果我们不想重复，我们应该使用Set。
* 一些集合类允许指定初始容量，所以如果能够估计好存储元素的数量，就可以提前设置好容器大小，避免重新哈希或大小调整带来的开销。
* 基于接口编程，而非基于实现编程，它允许我们后来轻易地改变实现。
* 总是使用类型安全的泛型，避免在运行时出现ClassCastException。
* 使用JDK提供的不可变类作为Map的key，可以避免自己实现`hashCode()`和`equals()`。
* 尽可能使用Collections工具类，或者获取只读、同步或空的集合，而非编写自己的实现。它将会提供代码重用性，它有着更好的稳定性和可维护性。

