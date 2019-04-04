### CountDownLatch用法

CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。例如有个任务要等待其他线程执行完再执行，这个时候可以使用countdownlatch来实现。

CountDownLatch类只提供了一个构造器：

```
public CountDownLatch(int count) {  };  //参数count为计数值
```

然后下面这3个方法是CountDownLatch类中最重要的方法：

```
public void await() throws InterruptedException { }; 
 //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行


public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行


public void countDown() { };  //将count值减1
```

```
public CountDownLatch countDownLatch = new CountDownLatch(2);

@Test


public void test() throws InterruptedException {


​


  new Thread(() -
>
 {


  try {


  //线程阻塞状态 等待 countdown n  =计算器count  次后并发执行


  countDownLatch.await();


  } catch (InterruptedException e) {


  e.printStackTrace();


  }


  System.out.println("a");


  }).start();


​


 new Thread(() -
>
 {


 try {


 //线程阻塞状态 countdown n次后并发执行


 countDownLatch.await();


 } catch (InterruptedException e) {


 e.printStackTrace();


 }


  System.out.println("b");


 }).start();


  countDownLatch.countDown();


  countDownLatch.countDown();


  System.out.println("c");


​


​


}
```

### CyclicBarrier用法

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await\(\)方法之后，线程就处于barrier了。

CyclicBarrier类位于java.util.concurrent包下，CyclicBarrier提供2个构造器：

```
public CyclicBarrier(int parties, Runnable barrierAction) {


}




public CyclicBarrier(int parties) {


}


参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。
```

```
第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；


​


​


// 每调用cyclicBarrier.await()一次累加 次数累加2次后并发执行先执行runnable


  public static CyclicBarrier cyclicBarrier=new CyclicBarrier(2,()-
>
{


  System.out.println(123);


  }); 


​


@Test


public void cyclicBarrierTest(){


  new Thread(() -
>
 {


  try {


  //线程阻塞状态 等待 countdown n  =计算器count  次后并发执行


  cyclicBarrier.await();


  } catch (Exception e) {


  e.printStackTrace();


  }


  System.out.println("a");


  }).start();


​


  new Thread(() -
>
 {


  try {


  //线程阻塞状态 countdown n次后并发执行


  cyclicBarrier.await();


  } catch (Exception e) {


  e.printStackTrace();


  }


  System.out.println("b");


  }).start();


  System.out.println("c");


}
```

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

```
@Test


public void cyclicBarrierTest1() throws InterruptedException {


Thread t=new Thread(() -
>
 {


  try {


  //等待N秒 如果没有barrier状态就直接让到达barrier的线程执行后续任务。


  cyclicBarrier.await(2, TimeUnit.SECONDS);


  } catch (Exception e) {


  e.printStackTrace();


  }


  System.out.println("a");


  });


​


  new Thread(() -
>
 {


  try {


  //线程阻塞状态 countdown n次后并发执行


  //  cyclicBarrier.await(2, TimeUnit.SECONDS);;


  } catch (Exception e) {


  e.printStackTrace();


  }


  System.out.println("b");


  }).start();


​


  t.start();


  t.join();


​


  System.out.println("c");


}
```

### Semaphore用法

Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire\(\) 获取一个许可，如果没有就等待，而 release\(\) 释放一个许可。

Semaphore类位于java.util.concurrent包下，它提供了2个构造器：

```
public Semaphore(int permits) { 
  //参数permits表示许可数目，即同时可以允许多少线程进行访问


  sync = new NonfairSync(permits);


}


public Semaphore(int permits, boolean fair) { 
  //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可


  sync = (fair)? new FairSync(permits) : new NonfairSync(permits);


}
```

重要的几个方法

```
public void acquire() throws InterruptedException {  } 
 //获取一个许可


public void acquire(int permits) throws InterruptedException { } 
  //获取permits个许可


public void release() { } 
  //释放一个许可


public void release(int permits) { } 
  //释放permits个许可
```

acquire\(\)用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release\(\)用来释放许可。注意，在释放许可之前，必须先获获得许可。

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```
public boolean tryAcquire() { }; 
  //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false


public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false


public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false


public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```

availablePermits\(\)方法得到可用的许可数目。

