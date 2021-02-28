## 深度
1. go语言的内存管理机制

2. go语言垃圾回收机制

3. go语言实现RSA与AES加密

4. go语言并发编程需要注意些什么? 

5. 你对 Scalability 理解是什么

6. 如何看待go-kit/iris/gin的架构设计

7. 如何看待你之前写过的项目的架构设计? 如何看待bilibili的go架构设计?

8. go后段接口与nginx转发配置

9. 简述你对锁的理解, 乐观锁悲观锁 


10. init() 函数是什么时候执行的
    `init()`函数是 Go 程序初始化的一部分。Go 程序初始化先于 main 函数，由`runtime`初始化每个导入的包，初始化顺序不是按照从上到下的导入顺序，而是按照解析的依赖关系，没有依赖的包最先初始化。
    每个包首先初始化包作用域的常量和变量（常量优先于变量），然后执行包的`init()`函数。同一个包，甚至是同一个源文件可以有多个 `init()`函数。`init()`函数没有入参和返回值，不能被其他函数调用，同一个包内多个`init()`函数的执行顺序不作保证。

    一句话总结： import –> const –> var –> init() –> main()


11. Go 语言的局部变量分配在栈上还是堆上？
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


12. 2 个 interface 可以比较吗 ？
    Go 语言中，interface 的内部实现包含了 2 个字段，类型 T 和 值 V，interface 可以使用 == 或 != 比较。2 个 interface 相等有以下 2 种情况
    1. 两个 interface 均等于 nil（此时 V 和 T 都处于 unset 状态）
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


13. 2 个 nil 可能不相等吗？
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


14. 函数返回局部变量的指针是否安全？
    这在 Go 中是安全的，Go 编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上。

15. 非接口的任意类型 T() 都能够调用 *T 的方法吗？反过来呢？
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

16. Golang 里的逃逸分析是什么？怎么避免内存逃逸？

17. sync.Pool用过吗，为什么使用，对象池，避免频繁分配对象（GC有关），那里面的对象是固定的吗？


