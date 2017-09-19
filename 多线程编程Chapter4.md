1. ReentrantLock类：基于AbstractQueuedSynchronizer(AQS)实现的，AbstractQueuedSynchronizer可以实现独占锁也可以实现共享锁，ReentrantLock只是使用了其中的独占锁模式,实现线程之间的同步互斥，并且在扩展功能上比synchronized更强大。提供了可轮询的锁请求，他可以尝试的去取得锁，如果取得成功则继续处理，取得不成功，可以等下次运行的时候处理，所以不容易产生死锁`Lock lock = new ReentrantLock(); lock.lock();/*要同步的内容*/lock.unlock();`调用lock()方法的线程就会持有"对象监视器"，其他线程只有等待锁被释放时再次争抢。
2. Condition实现等待/通知：ReentranLock实现等待通知需借助Condition对象，可实现多路通知功能，即在一个Lock对象里面可以穿件多个Condition实例(对象监视器)，线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知。
3. 公平锁和非公平锁：每一个Lock可以有任意数据的Condition对象，Condition是与Lock绑定的，所以就有Lock的公平性特性。ReentrantLock是可重入的互斥锁，由最近成功获取锁，还没释放的线程所拥有。`public ReentrantLock(boolean fair) {sync = fair ? new FairSync() : new NonfairSync();}`，公平锁将会严格的按照FIFO顺序获取锁，非公平锁由由自己设定，如按照优先级别等，后续的锁的竞争并不能保证FIFO的顺序。  
<pre>
//公平锁核心
//tryAcquire的公平版
protected final boolean tryAcquire(int acquires) {  
    final Thread current = Thread.currentThread(); 
    //获取锁的数量c
    int c = getState();
    //判断锁有没有被劫持0：没有  
    if (c == 0) {  
	//判断当前线程节点前是否还有其他线程在排队等待锁，
	//使用CAS将state（锁数量）从0设置为1
        if (!hasQueuedPredecessors() &&  
            compareAndSetState(0, acquires)
            //如果没有并且设置成功，当前线程current独占锁  
            setExclusiveOwnerThread(current);
            //请求成功  
            return true;  
        }  
    }
    //判断是不是当前线程current独占锁  
    else if (current == getExclusiveOwnerThread()) {
        //如果是就将当前锁数量状态值+1(acquires一般为1)
        //可重入锁的名称的来源  
        int nextc = c + acquires;  
        if (nextc < 0)  
            throw new Error("Maximum lock count exceeded");
        //设置当前所得状态为nextc  
        setState(nextc);
        //请求成功    
        return true;  
    }
    //获取失败  
    return false;  
}
//对于非公平版nonfairTryAcquire：没有判断当前线程节点前是否还有其他线程在排队等待锁
//锁持有数为0时，直接将当前线程设置为锁的持有者,以抢占的方式  
</pre>
4.1) getHoldCount()方法：查询当前线程保持保持锁定的个数，即调用lock()方法的次数.2）getQueueLength()返回正在等待获取此锁定的线程的估计数。3）getWaitQueueLength(Condition condition)返回等待与次锁定相关的给定条件Condition的线程估计数。4）isHeldByCurrentThread()查询当前线程是否保持此锁定。<font color=red>需要注意的地方：1）lock 必须在 finally 块中释放。否则，如果受保护的代码将抛出异常，锁就有可能永远得不到释放。2）Lock 类只是普通的类，JVM 不知道具体哪个线程拥有 Lock 对象。</font> 



1. ReentrantReadWriteLock类：读写锁，一个与读操作有关的锁称为共享锁；另一个与写操作有关的锁称为排它锁。即多个读操作的锁之间不互斥，读操作的锁与写操作的锁互斥，写操作的锁与写操作的锁互斥。
2. 提供的ReadLock是共享的，而WriteLock是独占的。于是Sync类同时实现了AQS(AbstractQueuedSynchronizer)中独占和共享模式的抽象方法(tryAcquire/tryAcquireShared等)，用同一个等待队列来维护读/写排队线程，而用一个32位int类型来表示锁被占用的线程数--要实现读锁和写锁，把状态的高16位用作读锁，记录所有读锁重入次数之和，低16位用作写锁，记录写锁重入次数。所以无论是读锁还是写锁最多只能被持有65535次.
   
