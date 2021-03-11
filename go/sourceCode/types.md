# go语言的类型
> 此文章受限于笔者的经验与理解,不能保证其正确性.

## 基本类型
### 基本类型的实现
在go1.4或之前,go还是基于c实现的.基本类型申明在[runtime.h](https://github.com/golang/go/blob/go1.4/src/runtime/runtime.h)上,网上很多文章都是基于这个分析的;

go1.5开始,golang实现了`自举`, 自己实现了编译器,所以从1.5开始golang的编译器与运行完全用go写了(带一点汇编),c语言不再参与实施;

> [types/type.go(1.14)](https://github.com/golang/go/tree/go1.14.15/src/go/types/type.go)

> [cmd/compile/internal/types/type.go(1.14)](https://github.com/golang/go/blob/go1.14.15/src/cmd/compile/internal/types/type.go)

go内部变量的申明在`src/builtin/builtin.go`(?这里存疑,builtin.go文件仅仅是用于文档说明用)



#### int到底是32位还是64位/取int的最大值
```go
tpm1 := uint(0)     // 0
// 对uint(0)取反,即可获得uint最大值
tpm2 := ^uint(0)    // 18446744073709551615, 可知是 2^64 即uint与int为64位
// 因为int的二进制第一位是符号位,所以对uint最大值右移一位,即 11...111 => 011...111 即int的最大值
tpm3 := ^uint(0) >> 1   // 9223372036854775807
```
见注释,对uint(0)取反即可得到当前最大的uint值,即可得uint的位数;

### interface
接口有两种底层结构,一种是空接口:eface,申明方法是`a := interface{}`;一种是有方法的接口:iface,申明方法是
```go
b := interface{
    Error()
}
```

> eface源码见src/runtime/runtime2.go
eface的定义是:
```go
type eface struct {
	_type *_type
	data  unsafe.Pointer
}
```

> iface源码见src/runtime/runtime2.go
iface的定义是:
```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```
eface有`_type`字段;而iface有内容更丰富的`itab`字段,其中就包含了`_type`字段;

#### []interface
> 原文见: https://github.com/golang/go/wiki/InterfaceSlice
以下语句将会报错:
```go
dataSlice := []int{1, 2, 3, 4, 5}
var iFaceSlice []interface{}
iFaceSlice = dataSlice
```
interface不应该什么类型都能表示吗?为什么这里无法将dataSlice赋给interface切片呢?

因为`[]interface{}`的类型不是interface,而是slice;在slice中,每个interface类型占两个字,而int类型只占一个字,它们的底层结构是不相同的;

正确的写法应该是:
```go
// method 1, 直接赋给interface
var iFaceSlice interface{}
iFaceSlice = dataSlice

// method 2, for循环赋值
iFaces := make([]interface{}, len(dataSlice))
for k, v := range dataSlice {
    iFaces[k] = v
}
```


### nil是什么?
> [理解nil](https://www.youtube.com/watch?v=ynoY2xz-F8s)

> nil is a predeclared identifier representing the zero value for a pointer, channel, func, interface, map, or slice type.
> nil是一个预先声明的标识符，表示指针、通道、函数、接口、映射或切片类型的零值。

nil并不是go的关键字之一,甚至你可以设定一个变量名字就要`nil`(但是最好不要这样做);

#### 两个为nil的变量未必相等
```go
type MagicError struct{}

func (MagicError) Error() string {
	return "[Magic]"
}
func Generate() *MagicError {
	return nil
}

func Test() error {
	return Generate()
}
func main() {
	fmt.Printf("%T %+v \n", Test(), Test())          // *main.MagicError nil
	fmt.Printf("%T %+v \n", Generate(), Generate())  // *main.MagicError nil
    fmt.Println(Generate() == nil)                   // true
	fmt.Println(Test() == Generate())                // true
	fmt.Println(Test() == nil)                       // false
}
```
main函数里,为什么`Generate()`与`Test()`返回的都是`<*main.MagicError nil>`, 为何Generate就能等于nil,而Test不等于nil呢?

> (此处存疑)这个问题讲的其实是interface的实现,使用`unsafe.Sizeof`看到Test的大小是16字节,而Generate的大小是8字节;因为Test返回的是实现了`error接口`的`MagicError`,而Generate返回的是`MagicError`对象,对象的值是nil;

> 众所周知,对象类型零值为nil,是等于nil的;而interface内部有Type和Value,interface判断相等必须type和value都相等,在这里Test的值是`<Type:MagicError Value:nil>`,而nil是`<Type:nil Value:nil>`,因此Test!=nil;


### struct



## map
1. map是hash table实现,无序;
2. hash冲突常用*线性探测*或者*拉链法*
    开放定址（线性探测）和拉链的优缺点
    - 拉链法比线性探测处理简单
    - 线性探测查找是会被拉链法会更消耗时间
    - 线性探测会更加容易导致扩容，而拉链不会
    - 拉链存储了指针，所以空间上会比线性探测占用多一点
    - 拉链是动态申请存储空间的，所以更适合链长不确定的

### 源码分析
位置: https://github.com/golang/go/tree/go1.14.15/src/runtime/map.go#L149

map同样也是数组存储的的，每个数组下标处存储的是一个bucket,这个bucket的类型见下面代码，每个bucket中可以存储8个kv键值对，当每个bucket存储的kv对到达8个之后，会通过overflow指针指向一个新的bucket，从而形成一个链表,看bmap的结构，我想大家应该很纳闷，没看见kv的结构和overflow指针啊，事实上，这两个结构体并没有显示定义，是通过指针运算进行访问的。

```go
// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
    // tophash通常包含此bucket中每个键的哈希值的顶部字节。如果tophash[0]<minTopHash，则tophash[0]是一个bucket撤离状态。
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
    // 接着是bucketCnt键，然后是bucketCnt元素。
    // 
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
    // 注意：将所有keys打包在一起，然后将所有元素打包在一起，这使得代码比交替的key/elem/key/elem/要复杂一些。。。但它允许我们消除需要的填充，例如map[int64]int8。后跟溢出指针。
}
```
我们能得到bucket中存储的kv是这样的，tophash用来快速查找key值是否在该bucket中，而不同每次都通过真值进行比较；还有kv的存放，为什么不是k1v1，k2v2..... 而是k1k2...v1v2...，我们看上面的注释说的`map[int64]int8`,key是int64（8个字节），value是int8（一个字节），kv的长度不同，如果按照kv格式存放，则考虑内存对齐v也会占用int64，而按照后者存储时，8个v刚好占用一个int64,从这个就可以看出go的map设计之巧妙。

最后我们分析一下go的整体内存结构，阅读一下map存储的源码，如下图所示，当往map中存储一个kv对时，通过k获取hash值，hash值的低八位和bucket数组长度取余，定位到在数组中的那个下标，hash值的高八位存储在bucket中的tophash中，用来快速判断key是否存在，key和value的具体值则通过指针运算存储，当一个bucket满时，通过overfolw指针链接到下一个bucket。


## array
> cmd/compile/internal/types/type.go:Array
> go/types/type.go:Array
数组是一块固定大小的连续的内存空间。
```go
type Array struct {
	Elem  *Type //元素类型
	Bound int64 //元素的个数(长度)
}
```
> 如果数组使用`[...]int{1,2,3}`初始化,则会调用:`cmd/compile/internal/gc/typecheck.go:typecheckcomplit`来计算长度;

### 新建array
```go
// cmd/compile/internal/types/type.go:473
// NewArray returns a new fixed-length array Type.
func NewArray(elem *Type, bound int64) *Type {
	if bound < 0 {
		Fatalf("NewArray: invalid bound %v", bound)
	}
	t := New(TARRAY)
	t.Extra = &Array{Elem: elem, Bound: bound}
	t.SetNotInHeap(elem.NotInHeap())
	return t
}
```
可以看到在新建array时就会判断array是分配在堆上还是分配在栈上;


## slice
> 源码分析 [slice](./slice.md)


## map、array和slice的区别与注意点



## reference
> [go源码](https://github.com/golang/go)
> [深入解析 Go 中 Slice 底层实现](https://halfrost.com/go_slice/#toc-0)
> [Go之读懂interface的底层设计](https://zhuanlan.zhihu.com/p/109964497)

