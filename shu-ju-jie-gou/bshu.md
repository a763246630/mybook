**B 树**可以看作是对2-3查找树的一种扩展，即他允许每个节点有M-1个子节点。

* 根节点至少有两个子节点
* 每个节点有M-1个key，并且以升序排列
* 位于M-1和M key的子节点的值位于M-1 和M key对应的Value之间
* 其它节点至少有M/2个子节点

![](/assets/btree1.png)

下图是一个M=4 阶的B树:

[![](https://images0.cnblogs.com/blog/94031/201403/290047064066682.png "B tree")](https://images0.cnblogs.com/blog/94031/201403/290047034539184.png)

