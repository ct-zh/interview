# GC垃圾回收
## 前置知识
### go里的堆栈
栈: 由操作系统自动分配释放，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈(先进后出, 自顶向下)。在程序中，每个函数块都会有自己的内存区域用来存自己的局部变量（内存占用少）、返回地址、返回值之类的数据，这一块内存区域有特定的结构和寻址方式，大小在编译时已经确定，寻址起来也十分迅速，开销很少。这一块内存地址称为栈。栈是线程级别的，大小在创建的时候已经确定，所以当数据太大的时候，就会发生"stack overflow"。栈使用的是一级缓存，他们通常都是被调用时处于存储空间中，调用完毕立即释放。

堆：(先进先出)在程序中，全局变量、内存占用大的局部变量、发生了逃逸的局部变量存在的地方就是堆，这一块内存没有特定的结构，也没有固定的大小，可以根据需要进行调整。简单来说，有大量数据要存的时候，就存在堆里面。堆是进程级别的。当一个变量需要分配在堆上的时候，开销会比较大，对于go这种带GC的语言来说，也会增加gc压力，同时也容易造成内存碎片。堆存放在二级缓存中，生命周期由gc来决定。所以调用这些对象的速度要相对来得低一些。

> [godoc FQA](https://golang.org/doc/faq#stack_or_heap)上有写: 
> Go 编译器自行决定变量分配在堆栈或堆上，以保证程序的正确性。

#### 变量到底分配在堆上还是栈上?
> 一般来说:
> 引用类型的全局变量内存分配在堆上，值类型的全局变量分配在栈上
> 局部变量内存分配可能在栈上也可能在堆上

> However, if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors. Also, if a local variable is very large, it might make more sense to store it on the heap rather than the stack.
> 但是，如果编译器在函数返回后无法证明变量未被引用，则编译器必须在gc heap上分配变量以避免空指针错误。此外，如果局部变量非常大，将它存储在堆而不是堆栈上可能更有意义。

下面是一个常见的指针逃逸代码, 函数`f1`返回了局部变量a的指针, 导致a产生逃逸,被分配在堆上面
```go
func f1() *string {
	a := "gggg"
	return &a
}
func main() {
	f1()
}
```
执行`go run -gcflags '-m -l' test.go`, 可以看到a确实被分配在堆上面:
```
# command-line-arguments
./test.go:5:9: &a escapes to heap
./test.go:4:2: moved to heap: a
```
打印汇编代码`go tool compile -S test.go`,在某一行看到`runtime.newobject(SB)`说明变量a是分配在堆上的
```
0x0000 00000 (test.go:3)	TEXT	"".f1(SB), $24-8
0x0000 00000 (test.go:3)	MOVQ	(TLS), CX
0x0009 00009 (test.go:3)	CMPQ	SP, 16(CX)
0x000d 00013 (test.go:3)	JLS	107
0x000f 00015 (test.go:3)	SUBQ	$24, SP
0x0013 00019 (test.go:3)	MOVQ	BP, 16(SP)
0x0018 00024 (test.go:3)	LEAQ	16(SP), BP
0x001d 00029 (test.go:3)	FUNCDATA	$0, gclocals·2a5305abe05176240e61b8620e19a815(SB)
0x001d 00029 (test.go:3)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
0x001d 00029 (test.go:4)	LEAQ	type.string(SB), AX
0x0024 00036 (test.go:4)	MOVQ	AX, (SP)
0x0028 00040 (test.go:4)	PCDATA	$0, $0
0x0028 00040 (test.go:4)	CALL	runtime.newobject(SB)
0x002d 00045 (test.go:4)	MOVQ	8(SP), DI
0x0032 00050 (test.go:4)	MOVQ	$4, 8(DI)
0x003a 00058 (test.go:4)	MOVL	runtime.writeBarrier(SB), AX
0x0040 00064 (test.go:4)	TESTL	AX, AX
0x0042 00066 (test.go:4)	JNE	93
0x0044 00068 (test.go:4)	LEAQ	go.string."gggg"(SB), AX
```

稍微改一改代码, 不返回局部变量的指针, 此时的a明显分配在栈上,随着`f1()`函数执行完毕被回收
```go
func f1() string {
	a = "5555"
	return a
}
func main() {
	f1()
}
```
执行逃逸分析:`go run -gcflags '-m -l' test.go`,没有任何输出,说明没有执政逃逸;再打印汇编代码:`go tool compile -S test.go`
```
0x0000 00000 (test.go:3)	TEXT	"".f1(SB), $8-16
0x0000 00000 (test.go:3)	MOVQ	(TLS), CX
0x0009 00009 (test.go:3)	CMPQ	SP, 16(CX)
0x000d 00013 (test.go:3)	JLS	88
0x000f 00015 (test.go:3)	SUBQ	$8, SP
0x0013 00019 (test.go:3)	MOVQ	BP, (SP)
0x0017 00023 (test.go:3)	LEAQ	(SP), BP
0x001b 00027 (test.go:3)	FUNCDATA	$0, gclocals·aef1f7ba6e2630c93a51843d99f5a28a(SB)
0x001b 00027 (test.go:3)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
0x001b 00027 (test.go:3)	MOVQ	"".a+16(SP), DI
0x0020 00032 (test.go:4)	MOVQ	$4, 8(DI)
0x0028 00040 (test.go:4)	MOVL	runtime.writeBarrier(SB), CX
......
```
可以看到`runtime.newobject`那一行消失了,说明变量a确实是分配在栈上面的.


### 逃逸分析
「逃逸」是指子程序某个对象本来应该分配在栈中,却因为在外部可能被引用,因此被分配在堆上;

如果一个子程序分配一个对象并返回一个该对象的指针，该对象可能在程序中的任何一个地方被访问到——这样指针就成功“逃逸”了。如果指针存储在全局变量或者其它数据结构中，它们也可能发生逃逸，这种情况是当前程序中的指针逃逸。逃逸分析需要确定指针所有可以存储的地方，保证指针的生命周期只在当前进程或线程中。
golang逃逸分析最基本的原则是：*如果一个函数返回的是一个（局部）变量的地址，那么这个变量就发生逃逸*。

#### 逃逸分析的用处（为了性能）
1. 减少gc的压力，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要gc标记清除。减少内存碎片的产生。
2. 因为逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好。

一般情况下，对于需要修改原对象值，或占用内存比较大的结构体，选择传指针。对于只读的占用内存较小的结构体，直接传值能够获得更好的性能。

#### 如何进行逃逸分析
run或者build时加上`-gcflags '-m -l'`,
- -m 会打印出逃逸分析的优化策略，实际上最多总共可以用 4 个 -m，但是信息量较大，一般用 1 个就可以了.
- -l 会禁用函数内联，在这里禁用掉内联能更好的观察逃逸情况，减少干扰。(也可以在func上面加上`//go:noinline`标记)

也可以直接看汇编代码:`go tool compile -S `
> go汇编快速指南 http://blog.rootk.com/post/golang-asm.html

#### 常见的几种逃逸
1. 指针逃逸:
```go
func GetUserInfo(userInfo UserData) *UserData {
  return &userInfo
}
```
返回的是一个局部变量的地址,这个局部变量可能在函数外被调用,因此产生逃逸;

2. interface逃逸:
```go
func main() {
	a := "aaa"
	fmt.Println(a)
}
```
`fmt.Println`的参数是interface,在函数`fmt.Println`内部编译器无法确认interface的类型,因此产生逃逸;

3. 栈空间不足:
对 Go 编译器而言，超过一定大小的局部变量将逃逸到堆上, 不同的 Go 版本的大小限制可能不一样。

4. 闭包:
子程序返回的闭包, 内部变量会一直存在,因此会被分配到堆上.


## gc
### 常见的gc
#### 引用计数
对每个对象维护一个引用计数，当引用该对象的对象被销毁时，引用计数减1，当引用计数器为0是回收该对象。
- 优点：对象可以很快的被回收，不会出现内存耗尽或达到某个阀值时才回收。
- 缺点：不能很好的处理循环引用，而且实时维护引用计数有一定的代价。
- 代表语言：Python、PHP、Swift

#### 标记-清除
从根变量开始遍历所有引用的对象，引用的对象标记为"被引用"，没有被标记的进行回收。
- 优点：解决了引用计数的缺点(不能处理循环引用，需要维护指针)。
- 缺点：需要STW。
- 代表语言：Golang(其采用三色标记法)

#### 分代收集
分代垃圾回收算法将对象按生命周期长短存放到堆上的两个（或者更多）区域，这些区域就是分代（generation）.对于新生代的区域的垃圾回收频率要明显高于老年代区域。分配对象的时候从新生代里面分配，如果后面发现对象的生命周期较长，则将其移到老年代，这个过程叫做 promote。随着不断 promote，最后新生代的大小在整个堆的占用比例不会特别大。收集的时候集中主要精力在新生代就会相对来说效率更高，STW 时间也会更短。
- 优点：回收性能好
- 缺点：算法复杂
- 代表语言：JAVA


### GC(垃圾回收)的工作原理
最常见的垃圾回收算法有标记清除(Mark-Sweep) 和引用计数(Reference Count)，Go 语言采用的是标记清除算法。并在此基础上使用了三色标记法和写屏障技术，提高了效率。
标记清除收集器是跟踪式垃圾收集器，其执行过程可以分成标记（Mark）和清除（Sweep）两个阶段：
- 标记阶段: 从根对象出发查找并标记root中所有存活的对象；
- 清除阶段: 遍历root中的全部对象，回收未被标记的垃圾对象并将回收的内存加入空闲链表。

> 所有垃圾回收都是针对堆的

标记清除算法的一大问题是在标记期间，需要暂停程序（Stop the world，STW），标记结束之后，用户程序才可以继续执行。为了能够异步执行，减少STW的时间，Go语言采用了三色标记法。

三色标记算法将程序中的对象分成白色、黑色和灰色三类。
- 白色：不确定对象。
- 灰色：存活对象，子对象待处理。
- 黑色：存活对象。
标记开始时，所有对象加入白色集合（这一步需STW）。首先将根对象标记为灰色，加入灰色集合，垃圾搜集器取出一个灰色对象，将其标记为黑色，并将其指向的对象标记为灰色，加入灰色集合。重复这个过程，直到灰色集合为空为止，标记阶段结束。那么白色对象即可需要清理的对象，而黑色对象均为根可达的对象，不能被清理。

三色标记法因为多了一个白色的状态来存放不确定对象，所以后续的标记阶段可以并发地执行。当然并发执行的代价是可能会造成一些遗漏，因为那些早先被标记为黑色的对象可能目前已经是不可达的了。所以三色标记法是一个false negative（假阴性）的算法。

三色标记法并发执行仍存在一个问题，即在 GC 过程中，对象指针发生了改变。比如下面的例子：
```
A (黑) -> B (灰) -> C (白) -> D (白)
```
正常情况下，D 对象最终会被标记为黑色，不应被回收。但在标记和用户程序并发执行过程中，用户程序删除了 C 对 D 的引用，而 A 获得了 D 的引用。标记继续进行，D 就没有机会被标记为黑色了（A 已经处理过，这一轮不会再被处理）。
```
A (黑) -> B (灰) -> C (白) 
↓
D (白)
```
为了解决这个问题，Go 使用了内存屏障技术，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，类似于一个钩子。垃圾收集器使用了写屏障（Write Barrier）技术，当对象新增或更新时，会将其着色为灰色。这样即使与用户程序并发执行，对象的引用发生改变时，垃圾收集器也能正确处理了。

### 一次完整的 GC 分为四个阶段
<img width="500px" src="http://legendtkl.com/img/uploads/2017/gc.png"></img>
1. Stack scan：收集根对象（全局变量，和G stack），开启写屏障。全局变量、开启写屏障需要STW，G stack只需要停止该G就好，时间比较少。
2. Mark: 扫描所有根对象, 和根对象可以到达的所有对象, 标记它们不被回收
3. Mark Termination: 完成标记工作, 重新扫描部分根对象(要求STW),关闭写屏障.
4. Sweep: 按标记结果清扫span(并发)

需要注意的是, 不是所有根对象的扫描都需要STW, 例如扫描栈上的对象只需要停止拥有该栈的G.

> root到底是什么?  
> root包括全局指针和G栈上的指针


## gc源码分析
> 源码基于go 1.14.15,

### 在什么时候触发gc?
#### 阈值触发
go从堆分配对象时会调用[newobject](https://github.com/golang/go/blob/go1.9.2/src/runtime/malloc.go#L840)函数, 先从这个函数看起:
```go
// implementation of new builtin
// compiler (both frontend and SSA backend) knows the signature
// of this function
func newobject(typ *_type) unsafe.Pointer {
	return mallocgc(typ.size, typ, true)
}
```

在堆新建对象时会调用([mallocgc()](https://github.com/golang/go/tree/go1.14.15/src/runtime/mgc.go#L1229),`gcTrigger{kind: gcTriggerHeap}`),新建对象是否触发gc由变量`shouldhelpgc`判断:
```go
// 分配一个字节大小的对象。
// 小对象是从P缓存的空闲列表中分配的。
// 大对象（>32KB）直接从堆中分配。
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	// 初值为false
	shouldhelpgc := false
	// ...
	// 如果对象小于32字节
	if size <= maxSmallSize {
		v := nextFreeFast(span)
		if v == 0 {
			// nextFree返回缓存范围中的下一个可用对象（如果有）。
			// 否则，它将使用一个具有可用对象的范围来重新填充缓存，并
			// 返回该对象以及一个标志，指示这是一个重的
			// 重量分配。如果是重权重分配，则调用者必须
			// 确定是否需要启动新的GC循环或GC是否处于活动状态
			// 这个goroutine是否需要协助GC。
			v, _, shouldhelpgc = c.nextFree(tinySpanClass)
		}
	} else {
		// 对象大于32字节, 直接执行gc
		shouldhelpgc = true
	}

	// 根据shouldhelpgc判断是否gc
	if shouldhelpgc {
		if t := (gcTrigger{kind: gcTriggerHeap}); t.test() {
			gcStart(t)
		}
	}
}
```

#### 主动触发
[runtime.GC()函数](https://github.com/golang/go/tree/go1.14.15/src/runtime/mgc.go#L1096), `gcTrigger{kind: gcTriggerCycle, n: n + 1}`


#### 两分钟定时触发
默认2min触发一次gc; 时间变量:[forcegcperiod](https://github.com/golang/go/tree/go1.14.15/src/runtime/proc.go#L4516) 执行函数:[forcegchelper()](https://github.com/golang/go/tree/go1.14.15/src/runtime/proc.go#L259)
```go
// The main goroutine.
func main() {
	// ...
	if GOARCH != "wasm" { // no threads on wasm yet, so no sysmon
		systemstack(func() {
			newm(sysmon, nil, -1)	// 在主routine里面加入sysmon监控
		})
	}
	// ... 
}

// forcegcperiod is the maximum time in nanoseconds between garbage
// collections. If we go this long without a garbage collection, one
// is forced to run.
// This is a variable for testing purposes. It normally doesn't change.
// forcegcperiod是垃圾回收之间的最长时间（以纳秒为单位）。
// 如果我们在没有垃圾收集的情况下走了这么长时间，他就会被迫运行。
// 这是一个用于测试目的的变量。它通常不会改变。
var forcegcperiod int64 = 2 * 60 * 1e9

func sysmon() {
	// 检测是否需要force gc
}

func forcegchelper() {
	// 检测force gc的锁,判断是否需要gc
	gcStart(gcTrigger{kind: gcTriggerTime, now: nanotime()})
}
```

### 具体流程与源码解析
#### gcstart
源码位置[gcStart()](https://github.com/golang/go/tree/go1.14.15/src/runtime/mgc.go#L1229)

```go
// gcStart starts the GC. It transitions from _GCoff to _GCmark (if
// debug.gcstoptheworld == 0) or performs all of GC (if
// debug.gcstoptheworld != 0).
// This may return without performing this transition in some cases,
// such as when called on a system stack or with locks held.
// gcStart启动GC。它从_GCoff转换到 _GCmark（如果debug.gcstoptheworld==0）或执行所有GC（如果debug.gcstoptheworld!= 0).
// 在某些情况下，这可能会不执行此转换直接返回，例如在系统堆栈上调用或持有锁时。
func gcStart(trigger gcTrigger) {
	// ...
}
```

#### 第一步
// todo



## 参考资料
> 1. [Golang源码探索](https://www.cnblogs.com/zkweb/p/7880099.html)
> 2. [Golang 垃圾回收剖析](http://legendtkl.com/2017/04/28/golang-gc/)
> 3. [Golang GC算法](https://www.jianshu.com/p/a5b84b16a6c9?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)