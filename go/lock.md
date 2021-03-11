1. lock-free https://blog.csdn.net/nawenqiang/article/details/83027222

2. 解释下乐观锁悲观锁
    悲观锁又叫阻塞同步，思路是防患于未然。因为异步可能会产生问题，所以要彻底杜绝异步。
    乐观锁也叫非阻塞同步，因为异步不一定会产生冲突，当真正产生异步问题时再同步执行。

3. 关于同步锁，下面说法正确的是（）
    A. 当一个goroutine获得了Mutex后，其他goroutine就只能乖乖的等待，除非该goroutine释放这个Mutex
    B. RWMutex在读锁占用的情况下，会阻止写，但不阻止读
    C. RWMutex在写锁占用情况下，会阻止任何其他goroutine（无论读和写）进来，整个锁相当于由该goroutine独占
    D. Lock()操作需要保证有Unlock()或RUnlock()调用与之对应
    


## answer

3. 关于同步锁，下面说法正确的是（）
     ABC
