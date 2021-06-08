# mutex锁



mutex几种状态

- `mutexLocked` — 表示互斥锁的锁定状态；
- `mutexWoken` — 表示从正常模式被从唤醒；
- `mutexStarving` — 当前的互斥锁进入饥饿状态；
- `waitersCount` — 当前互斥锁上等待的 Goroutine 个数；

**正常模式(非公平锁)**

正常模式下waiter都是先入先出(FIFO)，在队列中等待的waiter被唤醒后不会直接获取锁，因为要和新来的goroutine 进行竞争，新来的goroutine相对于被唤醒的waiter是具有优势的，新的goroutine 正在cpu上运行，被唤醒的waiter还要进行调度才能进入状态，所以在并发的情况下waiter大概率抢不过新来的goroutine，这个时候waiter会被放到队列的头部，如果等待的时间超过了1ms，这个时候Mutex就会进入饥饿模式。

```
在正常状态下，所有等待锁的goroutine按照FIFO(先进先出)顺序等待。唤醒的goroutine不会直接拥有锁，而是会和新请求锁的goroutine竞争锁的拥有。新请求锁的goroutine具有优势：它正在CPU上执行，而且可能有好几个，所以刚刚唤醒的goroutine有很大可能在锁竞争中失败。在这种情况下，这个被唤醒的goroutine会加入到等待队列的前面。 如果一个等待的goroutine超过1ms没有获取锁，那么它将会把锁转变为饥饿模式。
```

**饥饿模式(公平锁)**

为了解决了等待G队列的长尾问题
饥饿模式下，直接由unlock把锁交给等待队列中排在第一位的G(队头)，同时，饥饿模式下，新进来的G不会参与抢锁也不会进入自旋状态，会直接进入等待队列的尾部,这样很好的解决了老的g一直抢不到锁的场景。
饥饿模式的触发条件，当一个G等待锁时间超过1毫秒时，或者当前队列只剩下一个g的时候，Mutex切换到饥饿模式。

**总结**

对于两种模式，正常模式下的性能是最好的，goroutine可以连续多次获取锁，饥饿模式解决了取锁公平的问题，但是性能会下降，其实是性能和公平的一个平衡模式。

**补充**

- 允许自旋的条件

  1 锁已被占用，并且锁不处于饥饿模式。

  2 积累的自旋次数小于最大自旋次数（active_spin=4）。

  3 cpu核数大于1。

  4 有空闲的P。

  5 当前goroutine所挂载的P下，本地待运行队列为空。



这里的队列



cpu的指令重排 

内存屏障，CPU提供了 barrier指令





## Reference

[golang中的锁源码实现](http://legendtkl.com/2016/10/23/golang-mutex/)

[很细，mutex中runtime队列说法](http://birjemin.com/wiki/go-lock2)

