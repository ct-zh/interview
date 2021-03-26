# go语言
## 推荐教程
1. [effective go](https://learnku.com/docs/effective-go/2020)
2. [Go 语言简明教程](https://geektutu.com/post/quick-golang.html)
3. [Go 语言高性能编程](https://geektutu.com/post/high-performance-go.html)
4. [go语言圣经](https://docs.hacknode.org/gopl-zh/index.html)

## 「我觉得如果要考察一个人对一门语言的熟悉程度，少说也要涵盖这些方面」：
1. 语言基础是否学习完整？
    例：除了mutex以外还有那些方式安全读写共享变量? => channel

2. 语言基础是否足够扎实？
    例：无缓冲chan的发送和接收是否同步？ => 同步

3. 语言细节是否有过了解？
    例：golang采用什么并发模型？体现在哪里？
    大部分并发直接用channel和waitGroup就能解决问题，有的时候需要抽象出来，有的时候遇到的情况比较复杂，在《ConcurrencyInGo》这本书第四章，里面列举了基本全部的需求，fan-out/pipeline etc

4. 语言生态是否进行关注？
    例：在Vendor特性之前包管理工具是怎么实现的？ => GOPATH

5. 考察基础经验，
    例：说说golang中常用的并发模式？

6. 考察实际经验，
    例：JSON标准库对nil slice和 空slice的处理是一致的吗？
    不一致, nil slice => null,empty slice => `[]`


## Question:
1. go get命令, go的包管理方式, gopath, go mod, vendor;

2. go test;你常用的测试方法是什么? 简述你对单元测试的理解;简述你对表格驱动测试的理解;

3. 数组和slice有什么区别? 简述引用类型与值类型;

4. go一共有哪些数据类型? 

6. 简述你对duck typing的理解, go语言是如何实现封装、继承、多态的?

7. 结构体的最佳实践是什么? 对结构体的内存分析?

9. defer是怎么执行的? return和defer哪个最后执行?

10. 写网站的时候，浏览器写入url之后的全过程，包括后端的逻辑代码;

11. go 调度模型。发生网络io,会怎么调度。发生阻塞的IO会怎么调度。epoll详解;

13. int int64的区别,占多少位?

15. []byte{}  string 的区别

16. Go 和 PHP 在运行的时候有什么区别和优势?

17. Golang 长连接的时候是怎样做心跳机制的？

18. `=`与`:=`的区别*

19. 指针的作用*

21. Go 有异常类型吗？*

22. 如何高效地拼接字符串?*

23. 什么是 rune 类型?*

24. 如何判断 map 中是否包含某个 key*

25. Go 支持默认参数或可选参数吗*

26. defer 的执行顺序*

28. Go 语言 tag 的用处？*

29. 如何判断 2 个字符串切片（slice) 是相等的？*

30. 字符串打印时，%v 和 %+v 的区别*

31. Go 语言中如何表示枚举值(enums)？*

32. 空 struct{} 的用途*

33. 使用Go语言编程实现堆栈和队列这两个数据结构，该如何实现。可以只说实现思路。

34. `var a []int`和`a := []int{}`是否有区别？如果有的话，具体有什么区别？在开发过程中使用哪个更好，为什么？

35. Go中，如何复制切片内容？如何复制map内容？如何复制接口内容？编程时会如何操作实现。

37. 如何调试一个go程序？

38. 如何写单元测试和基准测试？

39. cap和len分别获取的是什么？*

41. 什么是interface？

42. select可以用于什么

43. client如何实现长连接

44. 主协程如何等其余协程完再操作

45. slice，len，cap，共享，扩容

46. map如何顺序读取

47. Go的反射包怎么找到对应的方法

49. go的new和make区别

50. go怎么从源码编译到二进制文件

52. go什么情况下会发生内存泄漏？

53. go struct能不能比较

54. go defer（for defer）

56. context包的用途

64. 当go服务部署到线上了，发现有内存泄露，该怎么处理

65. go map 的线程安全问题
    就是并发读写要挂掉，recover 都救不回来的那种，解决方式，粗暴的用 rwmtext，go 官方推荐用 sync.Map(没有分段锁，性能要求更高，请使用第三方实现)

66. 反射能获得类里面方法的名称吗?参数名称呢?参数类型呢?

67. 关于接口和类的说法，下面说法正确的是?
    A. 一个类只需要实现了接口要求的所有函数，我们就说这个类实现了该接口
    B. 实现类的时候，只需要关心自己应该提供哪些方法，不用再纠结接口需要拆得多细才合理
    C. 类实现接口时，需要导入接口所在的包
    D. 接口由使用方按自身需求来定义，使用方无需关心是否有其他模块定义过类似的接口

68. golang中大多数数据类型都可以转化为有效的JSON文本，下面几种类型除外（）
    A. 指针
    B. channel
    C. complex
    D. 函数

69. 关于cap函数的适用类型，下面说法正确的是（）
    A. array
    B. slice
    C. map
    D. channel

70. go日常开发常用命令有哪些?

71. init()函数是什么时候执行的

72. Go语言的局部变量分配在栈上还是堆上？

73. 2个interface可以比较吗？

74. 2个nil可能不相等吗？

75. 函数返回局部变量的指针是否安全？

76. 非接口的任意类型 T() 都能够调用 *T 的方法吗？反过来呢？

77. Golang 里的逃逸分析是什么？怎么避免内存逃逸？

78. Go runtime 了解吗?讲一下调度。发生文件IO的时候 G 怎么调度的？

79. netgo，cgo有什么区别？

80. go语言的内存管理机制

81. go语言并发编程需要注意些什么? 

82. 你对Scalability理解是什么

83. sync.Pool用过吗，为什么使用，对象池，避免频繁分配对象（GC有关），那里面的对象是固定的吗？

84. http包的基本使用; 如何获取url以及其参数分析; 如何获取header、auth; 如何获取form data; 如何进行文件上传与下载? 如何获取cookies与sessions? 

85. context包的具体使用环境是什么?简述context包的设计理念

86. 你有什么常用的包? 简述你对其的理解

87. 简述你对go rpc的理解;简述你对grpc的理解;简述rest与rpc的对比



## Answer
> Answer内容由受限于作者的经验水平,不保证准确性与时效性;

1. go get命令, go的包管理方式, gopath, go mod, vendor;

    go最早只支持gopath,1.5开始支持vendor,1.11知识module.(todo vendor了解不多)
    gomod依然使用GOPATH,但是以前的包下载在`$GOPATH/src/`里, 而gomod的包下载在`$GOPATH/pkg/mod/`里.
    > 详见: gomod工程化实践 https://segmentfault.com/a/1190000018398763


2. go test;你常用的测试方法是什么? 简述你对单元测试的理解;简述你对表格驱动测试的理解;

    测试一般有单元测试和基准测试;
    - 单元测试是编写一系列测试用例,保证一个程序模块符合设计,并且在将来修改的时候,可以极大程度保证该模块的行为仍然是正确的;表格驱动测试是单元测试的一种方法,将参数与想要的结果用表格记录下来直接输送到测试代码中,看起来更加直观,也更加易于修改;
    - 基准测试可以获取代码内存占用与运行效率有关的性能数据;

    比较常用的框架/工具有`go convey`测试加`go monkey`打桩;也可以用`go mock`打桩;(某些模块依赖于其他模块导致不便于单独测试,打桩就是手动代替这些依赖的部分)


3. 数组和slice有什么区别? 简述引用类型与值类型;

    数组是一段连续的内存空间,数组的长度不能改变.切片使用数组作为底层结构,切片可以随时进行扩展.

    值类型: 1. 基本数据类型; 2. 数组; 3. 结构体;    
    引用类型: slice、map、channel、func、interface、ptr


4. go一共有哪些数据类型? 
    - 空值：nil
    - 整型类型： int(取决于操作系统), int8, int16, int32(rune), int64, 以及对应的uint
    - 浮点数类型：float32, float64
    - 字节类型：byte (等价于uint8)
    - 字符串类型：string
    - 布尔值类型：boolean，(true 或 false)

    值得注意的是:
    1. 字符串使用UTF8编码，UTF8的好处在于，如果基本是英文，每个字符占1字节，和ASCII编码是一样的，非常节省空间，如果是中文，一般占3字节。字符串单个字节的类型是`uint8`,想要表示中文,就需要转换为`rune`(int32)
    2. int可能是int32或者int64,但是int不等于int32或者int64;一般情况下，int优先，需要明确bit数才需要int32或int64.int与uint可以用来判断操作系统位数.   

6. 简述你对duck typing的理解, go语言是如何实现封装、继承、多态的?

    duck typing是只要某个模块实现了一个接口定义的所有方法,我们就称这个模块是这个接口的一种实现;

    - go通过结构体实现封装,结构体中小写的字段无法被包外部访问;
    - 通过组合实现继承;在结构体B里直接加入结构体A,就继承了结构体A的所有属性和方法(外部包只能继承公有属性和方法)
    - 通过interface实现多态:只要一个结构体实现了接口声明的所有方法,就称这个结构体实现了这个接口;

7. 结构体的最佳实践是什么? 对结构体的内存分析?
    
    尽量将成员变量按照类型从大到小排序;因为结构体内存对齐,


9. defer是怎么执行的? return和defer哪个最后执行?
    
    defer的执行顺序: 后入先出;
    
    执行顺序: 函数体->defer->return;*注意*如果返回值是临时变量(`func foo() int`这种格式),则defer对其修改无效;如果返回值是指针或者非临时变量(`func foo() (a int)`这种格式),则defer对其修改有效;


10. 写网站的时候，浏览器写入url之后的全过程，包括后端的逻辑代码;

    浏览器向某个域名发起http请求,
    1. 先找当前系统的dns服务器,请求域名解析;dns服务器里如果没有域名记录,会往它上一级继续请求,直到找到域名解析记录;然后dns服务器会本地存一份解析,并将ip地址返回给系统;
    2. 系统向ip地址发送请求,服务器接收请求并完成指定逻辑,返回数据给浏览器;
    3. 浏览器渲染页面;


11. go 调度模型。发生网络io,会怎么调度。发生阻塞的IO会怎么调度。epoll详解;

    // todo


13. int int64的区别,占多少位?
    
    > [golang 里面为什么要设计 int 这样一个数据类型？](https://www.v2ex.com/t/744921)
    
    int在不同平台上有不同表现,一般来说64位系统int占64位,32位系统占32位;而int64固定占64位;

    int与机器的字长一致,可以保证最大的运行效率;在不关心数值范围的场景下使用int;而int32与int64一般用于编解码、底层硬件相关或者数值范围敏感的场景.


15. []byte{}  string 的区别

    string底层是一个指向字符串的地址与字符串的长度,而`[]byte`是一个切片slice,底层是由一个指向array的数组、长度len与数组容量cap这三个元素构成;

    string底层指向的字符串是不可更改的,每次更改字符串就需要重新分配一次内存;而`[]byte`底层数组如果cap足够,更改是不需要重新分配内存的,只有当cap不够了才需要重新申请一个array


16. Go 和 PHP 在运行的时候有什么区别和优势?

    go是静态编译语言,而php是动态解释型语言;go由于运行的是编译后的二进制可执行文件,运行效率要比php快很多很多;但是go需要根据不同平台不同架构编译出不同的二进制文件,并且编译后的文件不可更改,因此php相对更灵活.


17. Golang 长连接的时候是怎样做心跳机制的？

    // todo


18. `=`与`:=`的区别

    `:=` 声明+赋值
    `=` 仅赋值

19. 指针的作用

    指针用来保存变量的地址。`*`运算符，也称为解引用运算符，用于访问地址中的值。`＆`运算符，也称为地址运算符，用于返回变量的地址。

21. Go 有异常类型吗？

    Go 没有异常类型，只有错误类型Error.如果程序出现异常panic,则使用defer + recover的方式处理异常.

22. 如何高效地拼接字符串

    Go 语言中，字符串是只读的，也就意味着每次修改操作都会创建一个新的字符串。如果需要拼接多次，应使用 `strings.Builder`，最小化内存拷贝次数。

23. 什么是 rune 类型

    ASCII码只需要7 bit就可以完整地表示，但只能表示英文字母在内的128个字符，为了表示世界上大部分的文字系统，发明了Unicode，它是ASCII的超集，在Go语言中称之为rune，是int32类型的别名。

    Go语言中，字符串的底层表示是byte(8 bit)序列，而非rune(32 bit)序列。例如下面的例子中`语`和`言`使用UTF-8编码后各占3个byte，因此len("Go语言")等于8，当然我们也可以将字符串转换为rune序列。
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

26. defer的执行顺序
    - 1. 多个defer语句，遵从后进先出的原则，最后声明的defer语句，最先得到执行。
    - 2. defer在return语句之后执行，但在函数退出之前，defer可以修改返回值(注意修改的是引用类型而不是值类型);

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
    
    // todo

34. `var a []int`和`a := []int{}`是否有区别？如果有的话，具体有什么区别？在开发过程中使用哪个更好，为什么？

    - `var a []int`申明一个类型为int切片的变量;这个切片底层指针指向的是nil;len和cap都为0;判断`a == nil`结果为true;
    - `a := []int{}`申明一个类型为int切片的变量,并赋值了一个空切片;这个切片的len与cap都为0;判断`a == nil`结果为false;
    - 开发过程中用`var a []int`更好,因为`a := []int{}`多一步空切片的申请逻辑;

    验证代码:
    ```go
    var b []int
	a := []int{}

	structb := (*struct {
		p uintptr
		len int
		cap int
	})(unsafe.Pointer(&b))
	fmt.Println(structb)    // &{0 0 0}

	structa := (*struct {
		p uintptr
		len int
		cap int
	})(unsafe.Pointer(&a))
	fmt.Println(structa)   // &{18500656 0 0}
    ```

35. Go中，如何复制切片内容？如何复制map内容？如何复制接口内容？编程时会如何操作实现。

    (都是值复制而不是引用复制)
    - 复制切片内容: copy
    - 复制map内容: for range循环
    - 复制接口内容: 值类型的接口直接通过赋值来复制;指针类型的接口需要解引用;
    
37. 如何调试一个go程序？

    - 本地打印(log或者fmt);
    - goland有debug功能;
    - gdb调试二进制程序;

38. 如何写单元测试和基准测试？

    // todo

39. cap和len分别获取的是什么？

    cap是capacity,获取的是内建容器的容量;len获取的是内建容器的长度.

41. 什么是interface？

    interface定义了一系列方法的集合,任何其他类型实现了这些方法就相当于实现了这个接口;还有一种空接口,因为没有任何限定的方法,因此可以用任何类型来表示;

42. select可以用于什么

    // todo

43. client如何实现长连接

    // todo

44. 主协程如何等其余协程完再操作

    主协程等待其余协程完成需要解决两个问题:
    - 使用`sync.WaitGroup`或者channel同步等待;
    - 需要安全关闭channel:
        1. 单worker输出的channel由生产者自己关闭;
        2. 多worker输出的channel由第三方主动关闭,保证不能出panic(关闭一个已经关闭的通道会导致panic)
        3. 下游g程使用for-range等待channel关闭;
        4. worker中select监控多个channel,等待done信号;
        
        关闭ch最佳实践: (out channel = <-chan; in channel=chan<-)
        1. 在worker内部定义out channel;
        2. worker保证g程结束后关闭out channel;
        3. worker如果有多个in channel,可以使用sync.waitgroup,等待所有in处理完毕后再关闭out channel;
        4. 使用done channel close信号通知上游g程;
        5. 最终的消费者/主G程检查out是否被关闭;


45. slice，len，cap，共享，扩容

    slice底层结构为指向数组的指针、len、cap;len、cap在初始化的时候根据数组的大小决定,或者通过make来直接指定大小;

    slice共享是指当使用`a[0:1]`这种方式新创建一个slice时,新的slice底层指针与旧slice的底层指针指向的仍然是同一个数组;防止共享的方式是使用`copy()`来新建slice,新slice底层数组也是新建的;

    slice扩容是指当使用append方法增加切片内容时,如果增加内容后的数组容量大于cap,则触发扩容机制;扩容策略是:
    1. 如果新容量大于两倍旧容量,最终容量就是新申请的容量;
    2. 如果旧切片长度小于1024字节,则最终容量是旧容量的两倍;
    3. 如果旧切片长度大于1024字节,则循环增加1/4容量,直到大于最终容量;
    4. 如果最终容量计算值溢出,则最终容量就是新申请的容量;

46. map如何顺序读取

    map底层是hashmap实现的,特点是无序;想要顺序读取只要增加对key的约束规则即可,例如将key按顺序存入slice,或者key的命令规则有一定顺序;


47. Go的反射包怎么找到对应的方法

    `reflect.TypeOf(fn1).NumMethod`获取到方法数量,再通过`reflect.type`的method方法获取到reflect.method结构体;


49. go的new和make区别

    make只用于map、slice和channel的初始化,并且不会返回指针,返回的数据会根据类型有个初值.`new(T)`和`&T{}`是等价的,会返回一个指针,指向新分配的类型为`T`的零值(可以直接调用type的方法).

50. go怎么从源码编译到二进制文件

    - `go build`进行编译;

    - 交叉编译,在`go build`前面增加参数: `CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build hello.go`,其中`CGO_ENABLED=0`指关闭c语言版本的编译器(使用go自己的编译器);`GOOS=linux`代表目标系统是linux;`GOARCH=amd64`代表目标系统的架构是amd64


52. go什么情况下会发生内存泄漏？

    > [实战Go内存泄露](https://segmentfault.com/a/1190000019222661)
    // todo



53. go struct能不能比较

    如果struct的成员变量是可比较的,则两个struct可以进行比较;如果存在不可比较的成员变量(slice、map、func),则不能比较;

    可以用`reflect.DeepEqual`比较slice、map,尤其是多层嵌套的那种;

    扩展:
    1. struct可以作为map的key么

    struct必须是可比较的,才能作为map的key

    2. 两个不同的struct的实例能不能比较

    如果成员变量中含有不可比较成员变量，即使可以强制转换，也不可以比较


54. go defer（for defer）

    defer的参数值是在声明时确定的,而不是在调用时确定的;

56. context包的用途

    context是并发控制和超时控制的标准做法;context定义上下文类型，它跨API边界和进程之间传递截止日期、取消信号和其他请求范围的值。

64. 当go服务部署到线上了，发现有内存泄露，该怎么处理

    // todo

65. go map 的线程安全问题

    就是并发读写要挂掉，recover都救不回来的那种，解决方式，粗暴的用rwmtext，go官方推荐用 sync.Map(没有分段锁，性能要求更高，请使用第三方实现)

66. 反射能获得类里面方法的名称吗?参数名称呢?参数类型呢?

    可以获得公有方法的名称、参数名称和参数类型;不能获得私有方法;

67. 关于接口和类的说法，下面说法正确的是?

    ABD. 特别注意C, 实现接口时不需要导入接口所在的包,但是
    1. 如果接口里面有私有方法,这个接口便无法在包外实现;
    2. 调用类时给类定义的类型是接口的类型,此时会导入接口所在的包;
    ```go
    // package1
    type T1 interface {
        Kill()
    }

    // package2
    type T2 struct{}

    func (t T2) Kill() {}

    // package3,需要import package1与package2
    func NewT1() T1 {
        return &T2{}
    }
    ```

68. golang中大多数数据类型都可以转化为有效的JSON文本，下面几种类型除外（）

    BCD

69. 关于cap函数的适用类型，下面说法正确的是（）

    ABD

70. go日常开发常用命令有哪些?

    // todo

71. init() 函数是什么时候执行的
    `init()`函数是 Go 程序初始化的一部分。Go 程序初始化先于 main 函数，由`runtime`初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。
    每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的`init()`函数。同一个包，甚至是同一个源文件可以有多个 `init()`函数。`init()`函数没有入参和返回值，不能被其他函数调用，同一个包内多个`init()`函数的执行顺序不作保证。

    一句话总结： import –> const –> var –> init() –> main()


72. Go 语言的局部变量分配在栈上还是堆上？
    由编译器决定。Go 语言编译器会自动决定把一个变量放在栈还是放在堆，编译器会做逃逸分析(escape analysis)，当发现变量的作用域没有超出函数范围，就可以在栈上，反之则必须分配在堆上。
    ```go
    func foo() *int {
        v := 11
        return &v
    }

    func main() {
        m := foo()
        println(*m) // 11
    }
    ```
    `foo()`函数中，如果v分配在栈上，foo函数返回时，&v 就不存在了，但是这段函数是能够正常运行的。Go 编译器发现v的引用脱离了 foo 的作用域，会将其分配在堆上。因此，main 函数中仍能够正常访问该值。


73. 2个interface可以比较吗 ？
    Go语言中，interface的内部实现包含了2个字段，类型T和值V，interface可以使用`==`或`!=`比较。2个interface相等有以下2种情况
    1. 两个interface均等于nil(此时V和T都处于unset状态）
    2. 类型`T`相同，且对应的值`V`相等。
    ```go
    type Stu struct {
        Name string
    }

    type StuInt interface{}

    func main() {
        var stu1, stu2 StuInt = &Stu{"Tom"}, &Stu{"Tom"}
        var stu3, stu4 StuInt = Stu{"Tom"}, Stu{"Tom"}
        fmt.Println(stu1 == stu2) // false
        fmt.Println(stu3 == stu4) // true
    }
    ```
    `stu1`和`stu2`对应的类型是`*Stu`，值是`Stu`结构体的地址，两个地址不同，因此结果为false。
    `stu3`和`stu3`对应的类型是`Stu`，值是`Stu`结构体，且各字段相等，因此结果为true。

74. 2个nil可能不相等吗？
    可能。
    接口(interface)是对非接口值(例如指针，struct等)的封装，内部实现包含 2 个字段，类型 T 和 值 V。一个接口等于 nil，当且仅当 T 和 V 处于unset状态（T=nil，V is unset）。
    - 两个接口值比较时，会先比较 T，再比较 V。
    - 接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较。
    ```go
    func main() {
        var p *int = nil
        var i interface{}
        fmt.Printf("%+v %T \n", i, i)   // <nil> <nil> 

        i = p
        fmt.Println(i == p) // true
        fmt.Println(p == nil) // true
        fmt.Println(i == nil) // false
        fmt.Printf("%+v %T \n", i, i)   // <nil> *int
    }
    ```
    上面这个例子中，将一个 nil 非接口值 p 赋值给接口 i，此时，i 的内部字段为(T=*int, V=nil)，i 与 p 作比较时，将 p 转换为接口后再比较，因此 i == p，p 与 nil 比较，直接比较值，所以 p == nil。
    但是当 i 与 nil 比较时，会将 nil 转换为接口 (T=nil, V=nil)，与i (T=*int, V=nil) 不相等，因此 i != nil。因此 V 为 nil ，但 T 不为 nil 的接口不等于 nil。

75. 函数返回局部变量的指针是否安全？
    这在 Go 中是安全的，Go 编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上。

76. 非接口的任意类型 T() 都能够调用 *T 的方法吗？反过来呢？
    - 一个T类型的值可以调用为`*T`类型声明的方法，但是仅当此T的值是可寻址(addressable) 的情况下。编译器在调用指针属主方法前，会自动取此T值的地址。因为不是任何T值都是可寻址的，所以并非任何T值都能够调用为类型`*T`声明的方法。
    - 反过来，一个`*T`类型的值可以调用为类型T声明的方法，这是因为解引用指针总是合法的。事实上，你可以认为对于每一个为类型`T`声明的方法，编译器都会为类型`*T`自动隐式声明一个同名和同签名的方法。

    哪些值是不可寻址的呢？
    - 字符串中的字节；
    - map 对象中的元素（slice 对象中的元素是可寻址的，slice的底层是数组）；
    - 常量；
    - 包级别的函数等。

    举一个例子，定义类型 T，并为类型`*T`声明一个方法 hello()，变量 t1 可以调用该方法，但是常量 t2 调用该方法时，会产生编译错误。
    ```go
    type T string

    func (t *T) hello() {
        fmt.Println("hello")
    }

    func main() {
        var t1 T = "ABC"
        t1.hello() // hello
        const t2 T = "ABC"
        t2.hello() // error: cannot call pointer method on t
    }
    ```


77. Golang 里的逃逸分析是什么？怎么避免内存逃逸？

    // todo

78. Go runtime 了解吗?讲一下调度。发生文件IO的时候 G 怎么调度的？

    // todo

79. netgo，cgo有什么区别？

    // todo

80. go语言的内存管理机制

    // todo

81. go语言并发编程需要注意些什么? 

    // todo

82. 你对Scalability理解是什么

    // todo

83. sync.Pool用过吗，为什么使用，对象池，避免频繁分配对象（GC有关），那里面的对象是固定的吗？

    // todo

84. http包的基本使用; 如何获取url以及其参数分析; 如何获取header、auth; 如何获取form data; 如何进行文件上传与下载? 如何获取cookies与sessions? 

    // todo

85. context包的具体使用环境是什么?简述context包的设计理念

    context包是golang并发控制与超时控制的标准做法; context包代表协程运行的上下文,它是一棵goroutine调用树, 协程之间可以用context传递通知与元数据,主要目的是退出通知或者超时通知.


86. 你有什么常用的包? 简述你对其的理解

    // todo

87. 简述你对go rpc的理解;简述你对grpc的理解;简述rest与rpc的对比

    // todo


