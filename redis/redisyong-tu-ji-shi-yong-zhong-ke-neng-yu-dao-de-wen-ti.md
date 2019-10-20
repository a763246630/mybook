redis 缓存雪崩





redis 缓存击穿





redis 分布式锁setnx 

可能出现的问题

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