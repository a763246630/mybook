#### 缓存穿透

缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

解决方案：接口层增加校验，id<=0的直接拦截；
	从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击

#### redis 缓存雪崩

同一个时刻大量缓存失效。

​    **解决方案**

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
   1. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
2. 设置热点数据永远不过期。

#### redis 缓存击穿

同一个时间大量的线程发起请求，同时缓存没有存（业务会先查缓存没有的话去查库然后放在缓存里），导致所有线程直接查库，数据库压力瞬间增大，造成过大压力。

解决方法

1.查询时候使用分布式锁（互斥锁），同一时刻只允许一个线程去查库放到缓存，其他线程等待获取到锁之后再查一次缓存如果获取到就不用查库了。

2.设置热点数据永远不过期

redis 分布式锁setnx 

可能出现的问题

1.

确保过期时间大于业务执行时间

方法1.抽象类RedisLock增加一个boolean类型的属性isOpenExpirationRenewal，用来标识是否开启定时刷新过期时间。
在增加一个scheduleExpirationRenewal方法用于开启刷新过期时间的线程。

```
public abstract class RedisLock implements Lock {
	//...

    protected volatile boolean isOpenExpirationRenewal = true;

    /**
     * 开启定时刷新
     */
    protected void scheduleExpirationRenewal(){
        Thread renewalThread = new Thread(new ExpirationRenewal());
        renewalThread.start();
    }

    /**
     * 刷新key的过期时间
     */
    private class ExpirationRenewal implements Runnable{
        @Override
        public void run() {
            while (isOpenExpirationRenewal){
                System.out.println("执行延迟失效时间中...");

                String checkAndExpireScript = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                        "return redis.call('expire',KEYS[1],ARGV[2]) " +
                        "else " +
                        "return 0 end";
                jedis.eval(checkAndExpireScript, 1, lockKey, lockValue, "30");

                //休眠10秒
                sleepBySencond(10);
            }
        }
    }
}
```

加锁代码在获取锁成功后将isOpenExpirationRenewal置为true，并且调用scheduleExpirationRenewal方法，开启刷新过期时间的线程。

```java
public void lock() {
    while (true) {
        String result = jedis.set(lockKey, lockValue, NOT_EXIST, SECONDS, 30);
        if (OK.equals(result)) {
            System.out.println("线程id:"+Thread.currentThread().getId() + "加锁成功!时间:"+LocalTime.now());

            //开启定时刷新过期时间
            isOpenExpirationRenewal = true;
            scheduleExpirationRenewal();
            break;
        }
        System.out.println("线程id:"+Thread.currentThread().getId() + "获取锁失败，休眠10秒!时间:"+LocalTime.now());
        //休眠10秒
        sleepBySencond(10);
    }
}
```

方法2.设置业务方法的超时时间小于redis  setnx锁的超时时间