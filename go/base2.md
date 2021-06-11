# 补充
### go调度模型。发生网络io,会怎么调度。发生阻塞的IO会怎么调度。epoll详解;
go是GMP调度模型；
- 发生阻塞io，代码会将当前g打包成sudog结构，进入waiting状态；g与m都将休眠；m与p解绑，p去寻找空闲的m或者新开线程；
- 网络io：// todo

### go语言的内存管理机制

### select可以用于什么?

### 内存泄漏
> [go内存泄漏实战](https://mp.weixin.qq.com/s/d0olIiZgZNyZsO-OZDiEoA?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)
场景：
- channel读/写阻塞；
- select操作所有case都阻塞；
- 死循环；

思考：
- g如何退出？
- 是否有阻塞可能导致无法退出？如果有，那会不会创建大量类似的g？

解决办法：
`go pprof`获取文件，利用`top`,`traces`,`list`定位内存泄漏;

