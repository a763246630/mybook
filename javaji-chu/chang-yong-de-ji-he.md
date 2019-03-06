## java常见的集合

[TOC]

#### ArrayList

##### 基本特性

初始容量10,

优点:插入快.

缺点:查找慢，删除慢，长度固定，只能存储单一元素。  

##### 源码解析

```

1.成员变量

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
//长度
private int size;
//最大数组容量
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//initialCapacity 实例化传入指定初始容量
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {// 初始容量大于0,数组大小为指定大小
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//传入0直接赋值空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {// 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }


```



#### LinkList