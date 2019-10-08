# GC和JVM

java  gc主要是对堆内存和方法区空间的清理（回收没有引用的对象（无GCROOT根引用））。

### 堆内存对象分布结构

![](/assets/内存分布.png)

##### 堆内存主要分为Eden（新生代），两个Survivor区（to,from）（年轻代包含新生代）,old区（ 老年代）

##### 堆大小设置  初始堆大小 -Xms100M  最大堆大小   -Xmx100M

##### 设置年轻代大小 -Xmn 100M

##### 设置年轻代比例 -XX:SurvivorRatio=8:1:1  年轻代默认比例为Eden:to:from\(8:1:1

##### 设置元空间大 初始-XX:MetaspaceSize=8m  最大-XX:MaxMetaspaceSize=50m

##### 设置年轻代和年老代的比值 默认-XX:NewRatio=1:2

设置打印GC信息 -XX:+PrintGCDetails

##### minor GC（young GC）年轻代GC清理年轻代内存空间（回收无引用的对象）

##### full GC清理所有内存空间，

##### 对象gc轮转过程

新创建的对象会放入Eden（伊甸园区）伊甸园区放满会发生minorGC，发生gc会将Eden区存活对象放入Survivor区from，再次发生GC将Eden区和Survivor区from存活对象放入to区。**Survivor不够用直接放入老年代，** 15次GC没有回收的对象会移到老年代，老年代满了会发生fullGC

jvm内存设置案例

订单系统



![](/assets/ty1.png)

#### 内存调优工具

**Jinfo**

查看jvm的运行时参数  jinfo -flags 30880\(端口号\)

查看正在运行的Java应用程序的扩展参数  jinfo -sysprop 30880\(端口号\)

