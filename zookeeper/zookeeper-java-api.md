## zookeeper java api

\[TOC\]

### 1.同步创建

```
　create(java.lang.String path, byte[] data, java.util.List<org.apache.zookeeper.data.ACL> acl, org.apache.zookeeper.CreateMode createMode)

path：创建节点路径，需保证父节点已存在

data：节点数据

acl:权限列表

提供默认的权限OPEN_ACL_UNSAFE、CREATOR_ALL_ACL、READ_ACL_UNSAFE

OPEN_ACL_UNSAFE：完全开放

CREATOR_ALL_ACL：创建该znode的连接拥有所有权限

READ_ACL_UNSAFE：所有的客户端都可读

注意事项
1.创建节点的时候一定要先判断节点是否存在，如果直接使用zkclient创建一个已经存在的节点，则会抛出异常。 
2. 使用zkclient创建节点的时候，path描述的路径，预新增的最终节点之前的所有父节点都必须要存在，否则会抛出异常。所以根据自己需要做好处理。	
```

```
自定义权限
ACL aclIp = new ACL(ZooDefs.Perms.READ,new Id("ip","127.0.0.1"));
                ACL aclDigest = new ACL(ZooDefs.Perms.READ| ZooDefs.Perms.WRITE,
                        new Id("digest", DigestAuthenticationProvider.generateDigest("id:pass")));
```

```
session设置权限　
zk.addAuthInfo("digest", "id:pass".getBytes());　

createMode:节点类型

PERSISTENT：持久化节点

PERSISTENT_SEQUENTIAL：持久化有序节点

EPHEMERAL：临时节点（连接断开自动删除）

EPHEMERAL_SEQUENTIAL：临时有序节点（连接断开自动删除）
```

```
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;
import org.apache.zookeeper.server.auth.DigestAuthenticationProvider;

import java.io.IOException;
import java.security.NoSuchAlgorithmException;

/**
 * 创建节点(同步)
 * Created by scot on 2016/6/8.
 */
public class CreateSessionSync implements Watcher{
    private static ZooKeeper zooKeeper;

    @Override
    public void process(WatchedEvent watchedEvent) {

        if(watchedEvent.getState().equals(Event.KeeperState.SyncConnected)) {
            doBus();
        }
        System.out.println("接收内容："+watchedEvent.toString());
    }

    private void doBus() {
        try {
            if(null != zooKeeper.exists("/note_scot/note_scot_a",false)) {
                System.out.println("/note_scot/note_scot_a 节点已存在");
                return;
            }
            String path = zooKeeper.create("/note_scot/note_scot_a","aa".getBytes(),
                    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    /*权限相关
        try {
            ACL aclIp = new ACL(ZooDefs.Perms.READ,new Id("ip","127.0.0.1"));
            ACL aclDigest = new ACL(ZooDefs.Perms.READ| ZooDefs.Perms.WRITE,
                    new Id("digest", DigestAuthenticationProvider.generateDigest("id:pass")));
            zooKeeper.addAuthInfo("digest", "id:pass".getBytes());　
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }*/

        System.out.println("zookeeper return:" + path);
    } catch (KeeperException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

public static void main(String[] args) {
    try {
        zooKeeper = new ZooKeeper("192.168.117.128:2181",5000, new CreateSessionSync());
        System.out.println(zooKeeper.getState());
        Thread.sleep(5000);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

}

```
### 2.异步创建

　　create(java.lang.String path, byte[] data, java.util.List<org.apache.zookeeper.data.ACL> acl,

　　　　　 org.apache.zookeeper.CreateMode createMode, org.apache.zookeeper.AsyncCallback.StringCallback cb, java.lang.Object ctx) 

- - StringCallback cb：回调接口，执行创建操作后，结果以及数据发送到此接口的实现类中
  - Object ctx：自定义回调数据，在回调实现类可以获取此数据
```

import org.apache.zookeeper.\*;

import java.io.IOException;

/\*\*

* 创建节点\(异步\)
* Created by scot on 2016/6/8.  
   \*/  
   public class CreateSessionASync implements Watcher{  
   private static ZooKeeper zooKeeper;

  @Override  
   public void process\(WatchedEvent watchedEvent\) {

  ```
   if(watchedEvent.getState().equals(Event.KeeperState.SyncConnected)) {
       doBus();
   }
   System.out.println("接收内容："+watchedEvent.toString());
  ```

  }

  private void doBus\(\) {  
           zooKeeper.create\("/note\_scot/note\_scot\_b","aa".getBytes\(\), ZooDefs.Ids.OPEN\_ACL\_UNSAFE,  
                   CreateMode.PERSISTENT\_SEQUENTIAL,new IStringCallBack\(\),"testAsync"\);  
   }

  public static void main\(String\[\] args\) {  
       try {  
           zooKeeper = new ZooKeeper\("192.168.117.128:2181",5000, new CreateSessionASync\(\)\);  
           System.out.println\(zooKeeper.getState\(\)\);  
           Thread.sleep\(5000\);  
       } catch \(IOException e\) {  
           e.printStackTrace\(\);  
       } catch \(InterruptedException e\) {  
           e.printStackTrace\(\);  
       }  
   }

  static class IStringCallBack implements AsyncCallback.StringCallback {

  ```
   @Override
   public void processResult(int i, String s, Object o, String s2) {
       System.out.println("i="+i);//创建成功返回0
       System.out.println("s="+s);//自定义节点名称
       System.out.println("o="+o);//自定义回调数据
       System.out.println("s2="+s2);//最终节点名称（顺序节点最终名称与自定义名称不同）

   }
  ```

  }  
   }  
  \`\`\`

常用方法列表：

| String create\(final String path, byte data\[\], List acl, CreateMode createMode\) | 参数： 路径、 znode内容，ACL\(访问控制列表\)、 znode创建类型；用途：创建znode节点 |
| --- | --- |
| void delete\(final String path, int version\) | 参数： 路径、版本号；如果版本号与znode的版本号不一致，将无法删除，是一种乐观加锁机制；如果将版本号设置为-1，不会去检测版本，直接删除；用途：删除节点 |
| Stat exists\(final String path, Watcher watcher\) | 参数： 路径、Watcher\(监视器\)；当这个znode节点被改变时，将会触发当前Watcher用途：判断znode节点是否存在 |
| Stat exists\(String path, boolean watch\) | 参数： 路径、并设置是否监控这个目录节点，这里的 watcher 是在创建 ZooKeeper 实例时指定的 watcher；判断znode节点是否存在 |
| Stat setData\(final String path, byte data\[\], int version\) | 参数： 路径、数据、版本号；如果为-1，跳过版本检查用途：设置znode上的数据 |
| byte\[\] getData\(final String path, Watcher watcher, Stat stat\) | 参数： 路径、监视器、数据版本等信息用途：获取znode上的数据 |
| List getChildren\(final String path, Watcher watcher\) | 参数： 路径、监视器;该方法有多个重载用途：获取节点下的所有子节点 |

3.zookeeper中的事件和状态

事件和状态构成了zookeeper客户端连接描述的两个维度。注意，网上很多帖子都是在介绍zookeeper客户端连接的事件，但是忽略了zookeeper客户端状态的变化也是要进行监听和通知的。这里我们通过下面的两个表详细介绍  
\`\`\`

| 连接状态 | 状态含义 |
| --- | --- |
| KeeperState.Expired | 客户端和服务器在ticktime的时间周期内，是要发送心跳通知的。这是租约协议的一个实现。客户端发送request，告诉服务器其上一个租约时间，服务器收到这个请求后，告诉客户端其下一个租约时间是哪个时间点。当客户端时间戳达到最后一个租约时间，而没有收到服务器发来的任何新租约时间，即认为自己下线（此后客户端会废弃这次连接，并试图重新建立连接）。这个过期状态就是Expired状态 |
| KeeperState.Disconnected | 就像上面那个状态所述，当客户端断开一个连接（可能是租约期满，也可能是客户端主动断开）这是客户端和服务器的连接就是Disconnected状态 |
| KeeperState.SyncConnected | 一旦客户端和服务器的某一个节点建立连接（注意，虽然集群有多个节点，但是客户端一次连接到一个节点就行了），并完成一次version、zxid的同步，这时的客户端和服务器的连接状态就是SyncConnected |
| KeeperState.AuthFailed | zookeeper客户端进行连接认证失败时，发生该状态 |

### 3.Watcher涉及的类

![](/assets/zkapi1.png)

Watcher，接口类型，其定义了process方法，需子类实现。

Event，接口类型，Watcher的内部类，无任何方法。

KeeperState，枚举类型，Event的内部类，表示Zookeeper所处的状态。

EventType，枚举类型，Event的内部类，表示Zookeeper中发生的事件类型。

WatchedEvent，表示对ZooKeeper上发生变化后的反馈，包含了KeeperState和EventType。

ClientWatchManager，接口类型，表示客户端的Watcher管理者，其定义了materialized方法，需子类实现。

ZKWatchManager，Zookeeper的内部类，继承ClientWatchManager。

MyWatcher，ZooKeeperMain的内部类，继承Watcher。

ServerCnxn，接口类型，继承Watcher，表示客户端与服务端的一个连接。

WatchManager，管理Watcher。

Watcher源码分析

3.1 内部类

Event，接口类型，表示事件代表的状态，除去其内部类，其源码结构如下

```
public interface Event {}
```

说明：可以看到，Event接口并没有定义任何属性和方法，但是其包含了KeeperState和EventType两个内部枚举类。

3.2 接口方法

```
abstract public void process(WatchedEvent event);
```

说明：其代表了实现Watcher接口时必须实现的的方法，即定义进行处理，WatchedEvent表示观察的事件。

内部类

1. KeeperState是一个枚举类，其定义了在事件发生时Zookeeper所处的各种状态，其还定义了一个从整形值返回对应状态的方法fromInt。

```
           public enum KeeperState { // 事件发生时Zookeeper的状态
                /* Unused, this state is never generated by the server /
               @Deprecated
               // 未知状态，不再使用，服务器不会产生此状态
               Unknown (-1),

           /** The client is in the disconnected state - it is not connected
            * to any server in the ensemble. */
           // 断开
           Disconnected (0),

           /** Unused, this state is never generated by the server */
           @Deprecated
           // 未同步连接，不再使用，服务器不会产生此状态
           NoSyncConnected (1),

           /** The client is in the connected state - it is connected
            * to a server in the ensemble (one of the servers specified
            * in the host connection parameter during ZooKeeper client
            * creation). */
           // 同步连接状态
           SyncConnected (3),

           /**
            * Auth failed state
            */
           // 认证失败状态
           AuthFailed (4),

           /**
            * The client is connected to a read-only server, that is the
            * server which is not currently connected to the majority.
            * The only operations allowed after receiving this state is
            * read operations.
            * This state is generated for read-only clients only since
            * read/write clients aren't allowed to connect to r/o servers.
            */
           // 只读连接状态
           ConnectedReadOnly (5),

           /**
             * SaslAuthenticated: used to notify clients that they are SASL-authenticated,
             * so that they can perform Zookeeper actions with their SASL-authorized permissions.
             */
           // SASL认证通过状态
           SaslAuthenticated(6),

           /** The serving cluster has expired this session. The ZooKeeper
            * client connection (the session) is no longer valid. You must
            * create a new client connection (instantiate a new ZooKeeper
            * instance) if you with to access the ensemble. */
           // 过期状态
           Expired (-112);

           // 代表状态的整形值
           private final int intValue;     // Integer representation of value
                                           // for sending over wire
```

                 // 构造函数
                 KeeperState(int intValue) {
                     this.intValue = intValue;
                 }
    
                 // 返回整形值
                 public int getIntValue() {
                     return intValue;
                 }
    
                 // 从整形值构造相应的状态
                 public static KeeperState fromInt(int intValue) {
                     switch(intValue) {
                         case   -1: return KeeperState.Unknown;
                         case    0: return KeeperState.Disconnected;
                         case    1: return KeeperState.NoSyncConnected;
                         case    3: return KeeperState.SyncConnected;
                         case    4: return KeeperState.AuthFailed;
                         case    5: return KeeperState.ConnectedReadOnly;
                         case    6: return KeeperState.SaslAuthenticated;
                         case -112: return KeeperState.Expired;
    
                         default:
                             throw new RuntimeException("Invalid integer value for conversion to KeeperState");
                     }
                 }
             }
     ```

　3.3. EventType

```
   public enum EventType { // 事件类型
            // 无
            None (-1),
            // 结点创建
            NodeCreated (1),
            // 结点删除
            NodeDeleted (2),
            // 结点数据变化
            NodeDataChanged (3),
            // 结点子节点变化
            NodeChildrenChanged (4);

            // 代表事件类型的整形 
            private final int intValue;     // Integer representation of value
                                            // for sending over wire

            // 构造函数
            EventType(int intValue) {
                this.intValue = intValue;
            }

            // 返回整形
            public int getIntValue() {
                return intValue;
            }

            // 从整形构造相应的事件
            public static EventType fromInt(int intValue) {
                switch(intValue) {
                    case -1: return EventType.None;
                    case  1: return EventType.NodeCreated;
                    case  2: return EventType.NodeDeleted;
                    case  3: return EventType.NodeDataChanged;
                    case  4: return EventType.NodeChildrenChanged;

                    default:
                        throw new RuntimeException("Invalid integer value for conversion to EventType");
                }
            }           
        }
    }
```

3.4 WatchedEvent 类属性

```
public class WatchedEvent {
    // Zookeeper的状态
    final private KeeperState keeperState;
    // 事件类型
    final private EventType eventType;
    // 事件所涉及节点的路径
    private String path;
}
```

3.5 ZKWatchManager

```
  private static class ZKWatchManager implements ClientWatchManager {
        // 数据变化的Watchers
        private final Map<String, Set<Watcher>> dataWatches =
            new HashMap<String, Set<Watcher>>();
        // 节点存在与否的Watchers
        private final Map<String, Set<Watcher>> existWatches =
            new HashMap<String, Set<Watcher>>();
        // 子节点变化的Watchers
        private final Map<String, Set<Watcher>> childWatches =
            new HashMap<String, Set<Watcher>>();
    }
```

3.6 materialize 该方法表示事件发生时，返回需要被通知的Watcher集合，可能为空集合。

```
public Set<Watcher> materialize(Watcher.Event.KeeperState state,
                                        Watcher.Event.EventType type,
                                        String clientPath)
        {
            // 新生成结果Watcher集合
            Set<Watcher> result = new HashSet<Watcher>();

            switch (type) { // 确定事件类型
            case None: // 无类型
                // 添加默认Watcher
                result.add(defaultWatcher);
                // 是否需要清空(提取对zookeeper.disableAutoWatchReset字段进行配置的值、Zookeeper的状态是否为同步连接)
                boolean clear = ClientCnxn.getDisableAutoResetWatch() &&
                        state != Watcher.Event.KeeperState.SyncConnected;

                synchronized(dataWatches) { // 同步块
                    for(Set<Watcher> ws: dataWatches.values()) {
                        // 添加至结果集合
                        result.addAll(ws);
                    }
                    if (clear) { // 是否需要清空
                        dataWatches.clear();
                    }
                }

                synchronized(existWatches) { // 同步块 
                    for(Set<Watcher> ws: existWatches.values()) {
                        // 添加至结果集合
                        result.addAll(ws);
                    }
                    if (clear) { // 是否需要清空
                        existWatches.clear();
                    }
                }

                synchronized(childWatches) { // 同步块
                    for(Set<Watcher> ws: childWatches.values()) {
                        // 添加至结果集合
                        result.addAll(ws);
                    }
                    if (clear) { // 是否需要清空
                        childWatches.clear();
                    }
                }
                // 返回结果
                return result;
            case NodeDataChanged: // 节点数据变化
            case NodeCreated: // 创建节点
                synchronized (dataWatches) { // 同步块
                    // 移除clientPath对应的Watcher后全部添加至结果集合
                    addTo(dataWatches.remove(clientPath), result);
                }
                synchronized (existWatches) { 
                    // 移除clientPath对应的Watcher后全部添加至结果集合
                    addTo(existWatches.remove(clientPath), result);
                }
                break;
            case NodeChildrenChanged: // 节点子节点变化
                synchronized (childWatches) {
                    // 移除clientPath对应的Watcher后全部添加至结果集合
                    addTo(childWatches.remove(clientPath), result);
                }
                break;
            case NodeDeleted: // 删除节点
                synchronized (dataWatches) { 
                    // 移除clientPath对应的Watcher后全部添加至结果集合
                    addTo(dataWatches.remove(clientPath), result);
                }
                // XXX This shouldn't be needed, but just in case
                synchronized (existWatches) {
                    // 移除clientPath对应的Watcher
                    Set<Watcher> list = existWatches.remove(clientPath);
                    if (list != null) {
                        // 移除clientPath对应的Watcher后全部添加至结果集合
                        addTo(existWatches.remove(clientPath), result);
                        LOG.warn("We are triggering an exists watch for delete! Shouldn't happen!");
                    }
                }
                synchronized (childWatches) {
                    // 移除clientPath对应的Watcher后全部添加至结果集合
                    addTo(childWatches.remove(clientPath), result);
                }
                break;
            default: // 缺省处理
                String msg = "Unhandled watch event type " + type
                    + " with state " + state + " on path " + clientPath;
                LOG.error(msg);
                throw new RuntimeException(msg);
            }

            // 返回结果集合
            return result;
        }
    }
    该方法在事件发生后，返回需要被通知的Watcher集合。在该方法中，首先会根据EventType类型确定相应的事件类型，然后根据事件类型的不同做出相应的操作，如针对None类型，即无任何事件，则首先会从三个键值对中删除clientPath对应的Watcher，然后将剩余的Watcher集合添加至结果集合；针对NodeDataChanged和NodeCreated事件而言，其会从dataWatches和existWatches中删除clientPath对应的Watcher，然后将剩余的Watcher集合添加至结果集合
```

