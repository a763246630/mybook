### 1.同步创建

　　create(java.lang.String path, byte[] data, java.util.List<org.apache.zookeeper.data.ACL> acl, org.apache.zookeeper.CreateMode createMode)

path：创建节点路径，需保证父节点已存在

data：节点数据

acl:权限列表

提供默认的权限OPEN_ACL_UNSAFE、CREATOR_ALL_ACL、READ_ACL_UNSAFE

OPEN_ACL_UNSAFE：完全开放

CREATOR_ALL_ACL：创建该znode的连接拥有所有权限

READ_ACL_UNSAFE：所有的客户端都可读

自定义权限　　

```
ACL aclIp = new ACL(ZooDefs.Perms.READ,new Id("ip","127.0.0.1"));
                ACL aclDigest = new ACL(ZooDefs.Perms.READ| ZooDefs.Perms.WRITE,
                        new Id("digest", DigestAuthenticationProvider.generateDigest("id:pass")));
```

session设置权限　

```
zk.addAuthInfo("digest", "id:pass".getBytes());　　
```

createMode:节点类型

PERSISTENT：持久化节点

PERSISTENT_SEQUENTIAL：持久化有序节点

EPHEMERAL：临时节点（连接断开自动删除）

EPHEMERAL_SEQUENTIAL：临时有序节点（连接断开自动删除）



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
}
```

### 2.异步创建

　　create(java.lang.String path, byte[] data, java.util.List<org.apache.zookeeper.data.ACL> acl,

　　　　　 org.apache.zookeeper.CreateMode createMode, org.apache.zookeeper.AsyncCallback.StringCallback cb, java.lang.Object ctx) 

- - StringCallback cb：回调接口，执行创建操作后，结果以及数据发送到此接口的实现类中
  - Object ctx：自定义回调数据，在回调实现类可以获取此数据

```
import org.apache.zookeeper.*;

import java.io.IOException;

/**
 * 创建节点(异步)
 * Created by scot on 2016/6/8.
    */
    public class CreateSessionASync implements Watcher{
    private static ZooKeeper zooKeeper;

    @Override
    public void process(WatchedEvent watchedEvent) {

        if(watchedEvent.getState().equals(Event.KeeperState.SyncConnected)) {
            doBus();
        }
        System.out.println("接收内容："+watchedEvent.toString());
    }

    private void doBus() {
            zooKeeper.create("/note_scot/note_scot_b","aa".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,
                    CreateMode.PERSISTENT_SEQUENTIAL,new IStringCallBack(),"testAsync");
    }

    public static void main(String[] args) {
        try {
            zooKeeper = new ZooKeeper("192.168.117.128:2181",5000, new CreateSessionASync());
            System.out.println(zooKeeper.getState());
            Thread.sleep(5000);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class IStringCallBack implements AsyncCallback.StringCallback {

        @Override
        public void processResult(int i, String s, Object o, String s2) {
            System.out.println("i="+i);//创建成功返回0
            System.out.println("s="+s);//自定义节点名称
            System.out.println("o="+o);//自定义回调数据
            System.out.println("s2="+s2);//最终节点名称（顺序节点最终名称与自定义名称不同）
        
        }
    }
    }
```

常用方法列表：

| String create(final String path, byte data[], List acl, CreateMode createMode) | 参数： 路径、 znode内容，ACL(访问控制列表)、 znode创建类型；用途：创建znode节点 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void delete(final String path, int version)                  | 参数： 路径、版本号；如果版本号与znode的版本号不一致，将无法删除，是一种乐观加锁机制；如果将版本号设置为-1，不会去检测版本，直接删除；用途：删除节点 |
| Stat exists(final String path, Watcher watcher)              | 参数： 路径、Watcher(监视器)；当这个znode节点被改变时，将会触发当前Watcher用途：判断znode节点是否存在 |
| Stat exists(String path, boolean watch)                      | 参数： 路径、并设置是否监控这个目录节点，这里的 watcher 是在创建 ZooKeeper 实例时指定的 watcher；判断znode节点是否存在 |
| Stat setData(final String path, byte data[], int version)    | 参数： 路径、数据、版本号；如果为-1，跳过版本检查用途：设置znode上的数据 |
| byte[] getData(final String path, Watcher watcher, Stat stat) | 参数： 路径、监视器、数据版本等信息用途：获取znode上的数据   |
| List getChildren(final String path, Watcher watcher)         | 参数： 路径、监视器;该方法有多个重载用途：获取节点下的所有子节点 |

3.zookeeper中的事件和状态

事件和状态构成了zookeeper客户端连接描述的两个维度。注意，网上很多帖子都是在介绍zookeeper客户端连接的事件，但是忽略了zookeeper客户端状态的变化也是要进行监听和通知的。这里我们通过下面的两个表详细介绍 
```

| 连接状态                  | 状态含义                                                     |
| ------------------------- | ------------------------------------------------------------ |
| KeeperState.Expired       | 客户端和服务器在ticktime的时间周期内，是要发送心跳通知的。这是租约协议的一个实现。客户端发送request，告诉服务器其上一个租约时间，服务器收到这个请求后，告诉客户端其下一个租约时间是哪个时间点。当客户端时间戳达到最后一个租约时间，而没有收到服务器发来的任何新租约时间，即认为自己下线（此后客户端会废弃这次连接，并试图重新建立连接）。这个过期状态就是Expired状态 |
| KeeperState.Disconnected  | 就像上面那个状态所述，当客户端断开一个连接（可能是租约期满，也可能是客户端主动断开）这是客户端和服务器的连接就是Disconnected状态 |
| KeeperState.SyncConnected | 一旦客户端和服务器的某一个节点建立连接（注意，虽然集群有多个节点，但是客户端一次连接到一个节点就行了），并完成一次version、zxid的同步，这时的客户端和服务器的连接状态就是SyncConnected |
| KeeperState.AuthFailed    | zookeeper客户端进行连接认证失败时，发生该状态                |

