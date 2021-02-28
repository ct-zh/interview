# go语言
[effective go](https://learnku.com/docs/effective-go/2020)
[Go 语言简明教程](https://geektutu.com/post/quick-golang.html)
[Go 语言高性能编程](https://geektutu.com/post/high-performance-go.html)

## 我觉得如果要考察一个人对一门语言的熟悉程度，少说也要涵盖这些方面：
1. 语言基础是否学习完整？
例：除了 mutex 以外还有那些方式安全读写共享变量？

2. 语言基础是否足够扎实？
例：无缓冲 chan 的发送和接收是否同步？

3. 语言细节是否有过了解？
例：golang 采用什么并发模型？体现在哪里？
    大部分并发直接用 channel 和 waitGroup 就能解决问题，有的时候需要抽象出来，有的时候遇到的情况比较复杂，在《Concurrency In Go》这本书第四章，里面列举了基本全部的需求，fan-out / pipeline etc

4. 语言生态是否进行关注？
例：在 Vendor 特性之前包管理工具是怎么实现的？

5. 考察基础经验，
例：说说 golang 中常用的并发模式？
6. 考察实际经验，
例：JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗？


## 语言基础方面
1. go get命令, go的包管理方式, gopath, go mod, verdor;

2. go test;你常用的测试方法是什么? 简述你对单元测试的理解;简述你对表格驱动测试的理解;

3. 数组和slice有什么区别? 简述引用类型与值类型;

4. go一共有哪些数据类型? 

5. go一共有哪些内建容器? 变量的内存布局是怎样的?

6. 简述你对duck typing的理解, go语言是如何实现封装、继承、多态的?

7. 结构体的最佳实践是什么? 对结构体的内存分析?

8. map的底层实现

9. defer是怎么执行的? return和defer哪个最后执行?

10. 写网站的时候，浏览器写入url之后的全过程，包括后端的逻辑代码;

11. go 调度模型。发生网络io,会怎么调度。发生阻塞的IO会怎么调度。epoll详解

12. Go runtime 了解吗?讲一下调度。发生文件IO的时候 G 怎么调度的？

13. int int64的区别,占多少位?慢慢分析

14. go 逃逸分析   

15. []byte{}  string 的区别

16. Go 和 PHP 在运行的时候有什么区别和优势?

17. Golang 长连接的时候是怎样做心跳机制的？

18. `=`与`:=`的区别
    `:=` 声明+赋值
    `=` 仅赋值

19. 指针的作用
    指针用来保存变量的地址。`*`运算符，也称为解引用运算符，用于访问地址中的值。`＆`运算符，也称为地址运算符，用于返回变量的地址。

21. Go 有异常类型吗？
    Go 没有异常类型，只有错误类型（Error），通常使用返回值来表示异常状态。

22. 如何高效地拼接字符串
    Go 语言中，字符串是只读的，也就意味着每次修改操作都会创建一个新的字符串。如果需要拼接多次，应使用 `strings.Builder`，最小化内存拷贝次数。

23. 什么是 rune 类型
    ASCII 码只需要 7 bit 就可以完整地表示，但只能表示英文字母在内的128个字符，为了表示世界上大部分的文字系统，发明了 Unicode， 它是ASCII的超集，包含世界上书写系统中存在的所有字符，并为每个代码分配一个标准编号（称为Unicode CodePoint），在 Go 语言中称之为 rune，是 int32 类型的别名。

    Go 语言中，字符串的底层表示是 byte (8 bit) 序列，而非 rune (32 bit) 序列。例如下面的例子中`语`和`言`使用 UTF-8 编码后各占 3 个 byte，因此 len("Go语言") 等于 8，当然我们也可以将字符串转换为 rune 序列。
    ```go
    fmt.Println(len("Go语言")) // 8
    fmt.Println(len([]rune("Go语言"))) // 4
    ```

24. 如何判断 map 中是否包含某个 key
    ```go
    if val, ok := dict["foo"]; ok {
        //do something here
    }
    ```

25. Go 支持默认参数或可选参数吗
    Go 语言不支持可选参数（python 支持），也不支持方法重载（java支持）。

26. defer 的执行顺序
    - 多个 defer 语句，遵从后进先出(Last In First Out，LIFO)的原则，最后声明的 defer 语句，最先得到执行。
    - defer 在 return 语句之后执行，但在函数退出之前，defer 可以修改返回值。
    ```go
    func test() int {
        i := 0
        defer func() {
            fmt.Println("defer1")
        }()
        defer func() {
            i += 1
            fmt.Println("defer2")
        }()
        return i
    }

    func main() {
        fmt.Println("return", test())
    }
    // defer2
    // defer1
    // return 0        
    ```
    这个例子中，可以看到 defer 的执行顺序：后进先出。但是返回值并没有被修改，这是由于 Go 的返回机制决定的，执行 return 语句后，Go 会创建一个临时变量保存返回值，因此，defer 语句修改了局部变量 i，并没有修改返回值。那如果是有名的返回值呢？
    ```go
    func test() (i int) {
        i = 0
        defer func() {
            i += 1
            fmt.Println("defer2")
        }()
        return i
    }

    func main() {
        fmt.Println("return", test())
    }
    // defer2
    // return 1    
    ```
    这个例子中，返回值被修改了。对于有名返回值的函数，执行 return 语句时，并不会再创建临时变量保存，因此，defer 语句修改了 i，即对返回值产生了影响。


28. Go 语言 tag 的用处？
    tag 可以理解为 struct 字段的注解，可以用来定义字段的一个或多个属性。框架/工具可以通过反射获取到某个字段定义的属性，采取相应的处理方式。tag 丰富了代码的语义，增强了灵活性。

29. 如何判断 2 个字符串切片（slice) 是相等的？
    go 语言中可以使用反射 reflect.DeepEqual(a, b) 判断 a、b 两个切片是否相等，但是通常不推荐这么做，使用反射非常影响性能。
    通常采用遍历比较切片中的每一个元素（注意处理越界的情况）。

30. 字符串打印时，%v 和 %+v 的区别
    `%v` 和 `%+v` 都可以用来打印 struct 的值，区别在于 `%v` 仅打印各个字段的值，`%+v`还会打印各个字段的名称。

31. Go 语言中如何表示枚举值(enums)？
    通常使用常量(const) 来表示枚举值。

32. 空 struct{} 的用途
    使用空结构体 struct{} 可以节省内存，一般作为占位符使用，表明这里并不需要一个值。
    比如使用 map 表示集合时，只关注 key，value 可以使用 struct{} 作为占位符。如果使用其他类型作为占位符，例如 int，bool，不仅浪费了内存，而且容易引起歧义。
    再比如，使用信道(channel)控制并发时，我们只是需要一个信号，但并不需要传递值，这个时候，也可以使用 struct{} 代替。
    再比如，声明只包含方法的结构体。`type Lamp struct{}`


33. 使用Go语言编程实现堆栈和队列这两个数据结构，该如何实现。可以只说实现思路。

34. `var a []int`和`a := []int{}`是否有区别？如果有的话，具体有什么区别？在开发过程中使用哪个更好，为什么？


35. Go中，如何复制切片内容？如何复制map内容？如何复制接口内容？编程时会如何操作实现。

36. 了解读写锁吗，原理是什么样的，为什么可以做到？


37. 如何调试一个go程序？

38. 如何写单元测试和基准测试？

39. cap和len分别获取的是什么？


40. netgo，cgo有什么区别？


41. 什么是interface？


42. select可以用于什么

43. client如何实现长连接

44. 主协程如何等其余协程完再操作

45. slice，len，cap，共享，扩容

46. map如何顺序读取


47. Go的反射包怎么找到对应的方法

48. sync.Pool用过吗，为什么使用，对象池，避免频繁分配对象（GC有关），那里面的对象是固定的吗？


49. go的new和make区别

50. go怎么从源码编译到二进制文件


51. go的锁如何实现，用了什么cpu指令

52. go什么情况下会发生内存泄漏？

53. Golang 里的逃逸分析是什么？怎么避免内存逃逸？












