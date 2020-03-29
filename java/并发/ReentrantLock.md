### 基本介绍
ReentrantLock 是自 java1.5 开始提供的基于 AQS(一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架) 的可重入锁的实现，支持公平锁与非公平锁，提供了一些 synchronised 功能之外的一些高级功能

### 源码解析
#### 非公平锁获取锁方法 lock()
先 CAS 设置同步器状态由 0 到 1，若设置成功，则成功获取到锁，否则执行等待获取锁方法 acquire(1)
```
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

如下可见，acquire(1) 最终是调用了 nonfairTryAcquire(1) ,nonfairTryAcquire 方法一开始就尝试获取锁，获取成功则直接返回。这里就体现了锁的非公平性，新线程可直接抢占锁，只有抢占失败才会排队等待。下面直接一步步写代码注释解析代码逻辑，解析顺序按照 acquire() 方法的判断条件执行顺序，即 tryAcquire() --> addWaiter() --> acquireQueued()
```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

tryAcquire() 方法解析
```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        /**
         * 如果有线程释放锁，那么尝试获取锁（即 CAS 设置同步器状态由 0 到 1），
         * 若 CAS 成功，则获取锁成功，当前线程占有锁，并直接返回。
         */
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }

    /**
     * 若尝试获取锁失败，则判断当前线程是否已持有锁，
     * 若已持有，则将同步器状态加一并返回，这里就是处理锁的重入情况。
     */
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
addWaiter() 方法解析
```
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        /**
         * 如果等待链表（或者叫等待队列也行，毕竟链表可以当队列使用）的尾节点不为空，
         * 即有其它线程在前面排队，那么 CAS 将当前线程节点设置成尾节点
         */
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 如果尾节点为空即没有线程在排队或者等待队列未初始化则执行如下方法进行等待队列的初始化
    enq(node);
    return node;
}

private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        // 若等待队列未初始化，则 CAS 初始化等待队列
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 若等待队列已初始化，则将当前线程节点设置成等待队列尾节点
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
acquireQueued() 方法解析
```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            /**
             * 若前置节点是头节点，并且尝试获取锁成功，
             * 那么将当前线程节点设置成头节点，并且释放前置节点
             */
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            /**
             * 两次自旋排队获取锁，若都失败则阻塞（使用 UnSafe 的 park() 方法）当前线程  
             */
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
#### 释放锁方法 unlock()
如果尝试释放锁成功则返回 true ，否则返回 false，按照执行流程依次解析 tryRelease() --> unparkSuccessor()
```
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            // 解锁成功则解除等待队列头节点对应线程的阻塞状态
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
tryRelease() 方法解析
```
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        // 如果当前释放锁的线程非持有锁线程，则抛异常
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        // 如果同步器状态为 0 则锁释放成功
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```
unparkSuccessor() 方法解析
```
private void unparkSuccessor(Node node) {
    /*
     * 如果当前线程对应节点等待状态为负值，可能是等待唤醒，
     * 那么清空等待状态，即赋值 waitStatus = 0
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * 如果头结点的下一个节点为空或等待状态为取消状态，
     * 则从尾节点向前遍历等待队列，找到离头结点最近的等待唤醒的线程节点
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 如果找到了下一个等待唤醒的线程，则解除该线程的阻塞状态
        LockSupport.unpark(s.thread);
}
```
#### 公平锁获取锁方法 lock()
主要看如下方法，可得知跟非公平锁的区别，另外需解析一下 hasQueuedPredecessors() 方法是如何判断当前线程前面没有线程在排队的
```
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        /**
         * 如果当前线程前面没有线程在排队，那么如果 CAS 成功设置同步器状态从 0 到 1，
         * 那么当前线程就可获取锁，这里跟非公平锁的实现方案不同，
         * 非公平锁里面新线程可以直接抢占锁，而此处非公平锁的实现可以看出新线程必须先排队，
         * 排队到自己才能获取锁，这对没个线程而言都是按照申请获取锁的顺序 FIFO 获取锁的
         */
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
hasQueuedPredecessors() 解析
```
public final boolean hasQueuedPredecessors() {
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    /**
     * 如果头结点跟尾节点相等，此时相当于单线程获取锁，
     * 没有其它线程参与竞争，当头结点跟尾节点不相等并且头结点的后置
     * 节点不为空并且头结点的后置节点对应线程为当前线程，
     * 那么证明已经排队到当前线程了，当前线程拥有了获取锁的资格
     */
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

另外还有能响应中断的锁获取方法 lockInterruptibly()，指定锁等待时长的锁获取方法 tryLock(long time, TimeUnit unit)，实现线程等待与唤醒的绑定当前锁的条件对象 Condition  
lockInterruptibly(): 使用此方法获取锁，那么该线程便可以响应中断，在适当的场景下，其它线程可以执行此线程的 interrupt 方法来打断此线程的锁等待。  
tryLock((long time, TimeUnit unit): 使用此方法可以指定单位，指定时长等待锁，若等待锁超时则返回 false，若在等待时间范围内获取到了锁则返回 true  
Condition: 此对象可以绑定当前锁，并且一个锁可以绑定多个 Condition 对象，不同的对象可以应用至不同的场景，并提供线程的等待与唤醒方法
