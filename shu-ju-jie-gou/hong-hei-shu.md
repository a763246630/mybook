**R-B Tree简介**

```
R-B Tree，全称是Red-Black Tree，又称为“红黑树”，它一种特殊的二叉查找树。红黑树的每个节点上都有存储位表示节点的颜色，可以是红\(Red\)或黑\(Black\)
```

特点

1.每个节点是黑色或红色

2.根节点必须是黑色

3.叶子节点\(NULL\)是黑色

4.如果一个节点是红色，那么它的子节点必须是黑色

5.从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点

特性\(3\)中的叶子节点，是只为空\(null\)的节点。  
 特性\(5\)，确保没有一条路径会比其他路径长出俩倍。因而，红黑树是相对是接近平衡的二叉树

![](/assets/RBTREE1.png)

### **红黑树的应用**

红黑树的应用比较广泛，主要是用它来存储有序的数据，它的时间复杂度是O\(lgn\)，效率非常之高。  
例如，Java集合中的[TreeSet](http://www.cnblogs.com/skywang12345/p/3311268.html)和[TreeMap](http://www.cnblogs.com/skywang12345/p/3310928.html)，C++ STL中的set、map，以及Linux虚拟内存的管理，都是通过红黑树去实现的。

### **红黑树的Java实现\(代码说明\)**

红黑树的基本操作是**添加**、**删除**和**旋转**。在对红黑树进行添加或删除后，会用到旋转方法。为什么呢？道理很简单，添加或删除红黑树中的节点之后，红黑树就发生了变化，可能不满足红黑树的5条性质，也就不再是一颗红黑树了，而是一颗普通的树。而通过旋转，可以使这颗树重新成为红黑树。简单点说，旋转的目的是让树保持红黑树的特性。  
旋转包括两种：**左旋** 和 **右旋**。下面分别对红黑树的基本操作进行介绍

**1. 基本定义**

```
public class RBTree<T extends Comparable<T>> {

    private RBTNode<T> mRoot;    // 根结点

    private static final boolean RED   = false;
    private static final boolean BLACK = true;

    public class RBTNode<T extends Comparable<T>> {
        boolean color;        // 颜色
        T key;                // 关键字(键值)
        RBTNode<T> left;    // 左子字节
        RBTNode<T> right;    // 右子字节
        RBTNode<T> parent;    // 父结点

        public RBTNode(T key, boolean color, RBTNode<T> parent, RBTNode<T> left, RBTNode<T> right) {
            this.key = key;
            this.color = color;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

    }

    ...
}
```

RBTree是红黑树对应的类，RBTNode是红黑树的节点类。在RBTree中包含了根节点mRoot和红黑树的相关API。  
注意：在实现红黑树API的过程中，我重载了许多函数。重载的原因，一是因为有的API是内部接口，有的是外部接口；二是为了让结构更加清晰。

**2. 左旋**

![](/assets/RBTREE2.png)

对x进行左旋，意味着"将x变成一个左节点"。

左旋的实现代码(Java语言)

```
/* 
 * 对红黑树的节点(x)进行左旋转
 *
 * 左旋示意图(对节点x进行左旋)：
 *      px                              px
 *     /                               /
 *    x                               y                
 *   /  \      --(左旋)-.           / \                #
 *  lx   y                          x  ry     
 *     /   \                       /  \
 *    ly   ry                     lx  ly  
 *
 *
 */
private void leftRotate(RBTNode<T> x) {
    // 设置x的右子节点为y
    RBTNode<T> y = x.right;

    // 将 “y的左子节点” 设为 “x的右子节点”；
    // 如果y的左子节点非空，将 “x” 设为 “y的左子节点的父字节”
    x.right = y.left;
    if (y.left != null)
        y.left.parent = x;

    // 将 “x的父节点” 设为 “y的父节点”
    y.parent = x.parent;

    if (x.parent == null) {
        this.mRoot = y;            // 如果 “x的父节点” 是空节点，则将y设为根节点
    } else {
        if (x.parent.left == x)
            x.parent.left = y;    // 如果 x是它父节点的左子节点，则将y设为“x的父节点的左子节点”
        else
            x.parent.right = y;    // 如果 x是它父节点的左子节点，则将y设为“x的父节点的左子节点”
    }
    
    // 将 “x” 设为 “y的左子节点”
    y.left = x;
    // 将 “x的父节点” 设为 “y”
    x.parent = y;
}
```

**3. 右旋**

