## java常见的集合

[TOC]

#### ArrayList

##### 1.基本特性

初始容量10,当添加元素时数组元素数量+1大于（初始容量和数组元素数量+1中的较大的值）时,进入扩容

扩容为数组原来大小的1.5倍，(生成一个新的原数组大小的1.5倍的数组覆盖原数组对象)。

数组越大扩容会越占用资源，所有初始化指定合适的容量,会提高程序性能。

(正常情况下会扩容1.5倍，特殊情况下（新扩展数组大小已经达到了最大值）则只取最大值。)

优点:插入快.

缺点:查找慢，删除慢，长度固定，只能存储单一元素。  

##### 2.源码解析

###### 2.1属性

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

###### 2.2构造函数

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

###### 2.3 主要方法

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

#### LinkList