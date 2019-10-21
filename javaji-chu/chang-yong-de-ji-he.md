## java常见的集合

[TOC]



### ArrayList

#### 1.基本特性

初始容量10,当添加元素时数组元素数量+1大于（初始容量和数组元素数量+1中的较大的值）时,进入扩容

扩容为数组原来大小的1.5倍，\(生成一个新的原数组大小的1.5倍的数组覆盖原数组对象\)。

数组越大扩容会越占用资源，所有初始化指定合适的容量,会提高程序性能。

\(正常情况下会扩容1.5倍，特殊情况下（新扩展数组大小已经达到了最大值）则只取最大值。\)

优点: 当随机访问List时（get和set操作），ArrayList比LinkedList的效率更高，因为LinkedList是线性的数据存储方式，所以需要移动指针从前往后依次查找。

缺点:添加或删除元素慢，所有元素的下标。指向

#### 2.源码解析

##### 2.1属性

```
// 版本号
private static final long serialVersionUID = 8683452581122892189L;
//默认初始容量 10
private static final int DEFAULT_CAPACITY = 10;
//共享空数组实例用于空实例。
private static final Object[] EMPTY_ELEMENTDATA = {};
//缺省空对象数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//对象数组：ArrayList的底层数据结构  用transient关键字修饰的变量不被序列化
public transient Object[] elementData; 
//List元素数量不等于数组长度
private int size;
//最大数组容量 Integer.MIN_VALUE，即-2147483648，二进制位如下： 1000 0000 0000 0000 0000 0000 0000 0000 
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

##### 2.2构造函数

```
 public ArrayList() { 
        // 无参构造函数，设置元素数组为空 
        this
        .elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
 }

 //initialCapacity 实例化传入指定初始容量
 public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) { // 初始容量大于0
            this.elementData = new Object[initialCapacity]; // 初始化元素数组
        } else if (initialCapacity == 0) { // 初始容量为0
            this.elementData = EMPTY_ELEMENTDATA; // 为空对象数组
        } else { // 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
 }   
 //构造传入集合转换成数组
  public ArrayList(Collection<? extends E> c) { // 集合参数构造函数
        elementData = c.toArray(); // 转化为数组
        if ((size = elementData.length) != 0) { // 参数为非空集合
            if (elementData.getClass() != Object[].class) // 是否成功转化为Object类型数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
                // 不为Object数组的话就进行复制
        } else { // 集合大小为空，则设置元素数组为空
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

##### 2.3 主要方法

###### 2.3.1 add 方法

![](/assets/集合1.png)

```
// 添加元素 jdk1.8    
public boolean add(E e) { 
        //添加新元素后,数组最小容量等于元素数量加1和默认容量的中较大值为最小容量。
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
 }
 //确保容量是安全的，超过阀值就扩容
 private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { // 判断元素数组是否为空数组
            //取元素数量加1和默认容量的中较大值为最小容量
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity); // 取较大值
          }
          ensureExplicitCapacity(minCapacity);
 }
 private void ensureExplicitCapacity(int minCapacity) {
        // 结构性修改加1
        modCount++;
        //最小容量大于数组的容量就库容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

//扩容方法 数组元素数量+1 小于 最小容量 则 扩容1.5倍
private void grow(int minCapacity) {
        int oldCapacity = elementData.length; // 旧容量
        int newCapacity = oldCapacity + (oldCapacity >> 1); 
        // 新容量为旧容量的1.5倍 oldCapacity>>1 等于oldCapacity的一半
        if (newCapacity - minCapacity < 0) // 新容量小于参数指定容量，修改新容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0) // 新容量大于最大容量
            newCapacity = hugeCapacity(minCapacity); // 指定新容量
        // 拷贝扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
}
    // jdk1.7之前
     // 向elementData中添加元素 
    public boolean add(E e) {
        ensureCapacity(size + 1);
        //确保对象数组elementData有足够的容量，可以将新加入的元素e加进去
        elementData[size++] = e;
        //加入新元素e，size加1
        return true;
    }


    // 确保数组的容量足够存放新加入的元素，若不够，要扩容

    public void ensureCapacity(int minCapacity) {
        modCount++;
        int oldCapacity = elementData.length;//获取数组大小（即数组的容量）
        //当数组满了，又有新元素加入的时候，执行扩容逻辑
        if (minCapacity > oldCapacity) {
            Object oldData[] = elementData;
            int newCapacity = (oldCapacity * 3) / 2 + 1;//新容量为旧容量的1.5倍+1
            if (newCapacity < minCapacity)//如果扩容后的新容量还是没有传入的所需的最小容量大或等于（主要发生在addAll(Collection<? extends E> c)中）
                newCapacity = minCapacity;//新容量设为最小容量
            elementData = Arrays.copyOf(elementData, newCapacity);//复制新容量
        }
    }
```

###### 2.3.2 set 方法

```
//等价于update 只能覆盖存在元素的下标和元素
public E set(int index, E element) {
        // 检验索引是否合法
        rangeCheck(index);
        // 获取旧值
        E oldValue = elementData(index);
        // 赋新值
        elementData[index] = element;
        // 返回旧值
        return oldValue;
  }
 private void rangeCheck(int index) {
 //索引必须小于元素数量
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
 }   
  E elementData(int index) {
        return (E) elementData[index];
  }
```

###### 2.3.2 indexOf方法

```
//list是否包含传入元素，包含则返回元素下标，不在则返回-1
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

###### 2.3.3 get

```
  //返回传入索引对应的元素
  public E get(int index) {
        // 检验索引是否合法
        rangeCheck(index);

        return elementData(index);
    }
```

2.3.4 remove

```
public E remove(int index) {
        // 检查索引是否存在
        rangeCheck(index);

        modCount++;
        //获取下标的元素
        E oldValue = elementData(index);
        // 需要移动的元素的个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 赋值为空，有利于进行GC
        elementData[--size] = null; 
        // 返回旧值
        return oldValue;
    }
```

ArrayList和LinkedList的区别（也是顺序表和链表的区别）：

（1）ArrayList是基于动态数组实现的，LinkedList是基于双向链表实现的。

（2）ArrayList支持随机访问（通过下标），LinkedList不支持。

（3）ArrayList的查询和尾部插入元素效率较高，中间插入和删除元素效率较低，因为有大量的数组复制操作。LinkedList的插入和删除效率较高，只需要把节点指针改变一下就行，但是查询效率较低，即使有双向链表的特性可以从两个方向查找，但还是需要使用蛮力法的方式进行遍历，所以效率较低。

（4）ArrayList会造成一定的空间浪费，因为随时需要探测数组容量然后扩容；LinkedList通过节点方式进行存放数据，不存在空间浪费。

### LinkList

1.基本特性

底层采用双向链表

![](/assets/集合2.png)

### Victor



### HashMap

##### 默认初始容量为16，负载因子是0.75，超过12（16*0.75）就扩容1倍

我们知道HashMap是线程不安全的，在多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。

* HashMap概念和底层结构

```
HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。HashMap储存的是键值对，HashMap很快。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

数组：存储区间连续，占用内存严重，寻址容易，插入删除困难；  
链表：存储区间离散，占用内存比较宽松，寻址困难，插入删除容易；  
Hashmap综合应用了这两种数据结构，实现了寻址容易，插入删除也容易。
```

![](/assets/hashmap1.png)

- **HashMap的基本存储原理以及存储内容的组成**

```
最外层是个数组（transient Node<K,V>[] table）,在new HashMap()只是将initialCapacity初始容量（默认大小16）负载因子loadFactor(默认16)赋值给 (final float loadFactor)计算出阈值
(int threshold)，。另外设计一个哈希函数（也叫做散列函数）来获得每一个元素的Key（关键字）的函数值（即数组下标，hash值）相对应，数组存储的元素是一个Entry类，这个类有三个数据域，key、value（键值对），next(指向下一个Entry)。
例如， 第一个键值对A进来。通过计算其key的hash得到的index=0。记做:Entry[0] = A。
第二个键值对B，通过计算其index也等于0， HashMap会将B.next =A,Entry[0] =B,
第三个键值对 C,index也等于0,那么C.next = B,Entry[0] = C；这样我们发现index=0的地方事实上存取了A,B,C三个键值对,它们通过next这个属性链接在一起。我们可以将这个地方称为桶。 对于不同的元素，可能计算出了相同的函数值，这样就产生了“冲突”，这就需要解决冲突，“直接定址”与“解决冲突”是哈希表的两大特点。
```

- **HashMap的工作原理以及存取方法过程**

```
HashMap的工作原理 ：HashMap是基于散列法（又称哈希法hashing）的原理，使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket（桶）位置来储存Entry对象。”HashMap是在bucket中储存键对象和值对象，作为Map.Entry。并不是仅仅只在bucket中存储值。
```

- **HashMap中的碰撞探测(collision detection)以及碰撞的解决方法**

```
当两个对象的hashcode相同时，它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用LinkedList存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在LinkedList中。这两个对象就算hashcode相同，但是它们可能并不相等。 那如何获取这两个对象的值呢？当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，遍历LinkedList直到找到值对象。找到bucket位置之后，会调用keys.equals()方法去找到LinkedList中正确的节点，最终找到要找的值对象使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。
```

- **如何重新调整HashMap的大小**

```
“如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？”
HashMap的扩容阈值（threshold = capacity* loadFactor 容量值范16~2的30次方），就是通过它和size进行比较来判断是否需要扩容。默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组（jdk1.6，但不超过最大容量），来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。
```

- **解决 hash 冲突的常见方法**

```
a. 链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为 i 的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。

b. 开放定址法：即发生冲突时，去寻找下一个空的哈希地址。只要哈希表足够大，总能找到空的哈希地址。

c. 再哈希法：即发生冲突时，由其他的函数再计算一次哈希值。

d. 建立公共溢出区：将哈希表分为基本表和溢出表，发生冲突时，将冲突的元素放入溢出表。

HashMap 就是使用链地址法来解决冲突的（jdk8中采用平衡树来替代链表存储冲突的元素，但hash() 方法原理相同）。数组中的每一个单元都会指向一个链表，如果发生冲突，就将 put 进来的 K- V 插入到链表的尾部。
```

- **HashMap并发环境下是线程不安全的,可能会出现下面的并发问题.**

**多线程put后可能导致get死循环**

**多线程put的时候可能导致元素丢失**

**put非null元素后get出来的却是null**

**resize扩容造成链表死循环**



### HashTable

HashTable和HashMap的实现原理几乎一样，差别无非是

* HashTable不允许key和value为null
* HashTable是线程安全的

但是HashTable线程安全的策略实现代价却太大了，简单粗暴，get/put所有相关操作都是synchronized的，这相当于给整个哈希表加了一把大锁。

多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞，相当于将所有的操作串行化，在竞争激烈的并发场景中性能就会非常差。

### HashSet

有序的HashMap

线程不安全，存取速度快

底层实现是一个HashMap（保存数据），实现Set接口

### ConcurrentHashMap

JDK1.7和1.8实现线程安全的区别

## JDK1.7版本的CurrentHashMap的实现原理

在JDK1.7中ConcurrentHashMap采用了数组+Segment+分段锁的方式实现。

1.Segment\(分段锁\)

ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

2.内部结构

ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问\(读写锁\)，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：

从上面的结构我们可以了解到，ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作。

第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部



**坏处**

这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长

**好处**

写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作（刚好这些写操作都非常平均地分布在所有的Segment上）。ConcurrentHashMap的并发能力可以大大的提高。

## JDK1.8版本的CurrentHashMap的实现原理

JDK8中ConcurrentHashMap参考了JDK8 HashMap的实现，采用了数组+链表+红黑树的实现方式来设计，内部大量采用CAS操作，这里我简要介绍下CAS。

CAS是compare and swap的缩写，即我们所说的比较交换。cas是一种基于锁的操作，而且是乐观锁。在java中锁分为乐观锁和悲观锁。悲观锁是将资源锁住，等一个之前获得锁的线程释放锁之后，下一个线程才可以访问。而乐观锁采取了一种宽泛的态度，通过某种方式不加锁来处理资源，比如通过给记录加version来获取数据，性能较悲观锁有很大的提高。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值\(B\)。如果内存地址里面的值和A的值是一样的，那么就将内存里面的值更新成B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a线程获取地址里面的值被b线程修改了，那么a线程需要自旋，到下次循环才有可能机会执行。

JDK8中彻底放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想。

Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性。

其中抛弃了原有的 Segment 分段锁，而采用了CAS + synchronized来保证并发安全性。

数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。

保证线程安全机制：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。

锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）。链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储

#### CopyOnWriteArrayList



