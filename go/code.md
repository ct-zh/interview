可以用来回答: go有踩过什么坑?

## go基础问题
1. 以下代码有什么问题，说明原因
    ```go
    type student struct {
        Name string
        Age  int
    }

    func pase_student() {
        m := make(map[string]*student)
        stus := []student{
            {Name: "zhou", Age: 24},
            {Name: "li", Age: 23},
            {Name: "wang", Age: 22},
        }
        for _, stu := range stus {
            m[stu.Name] = &stu
        }

    }
    ```

    问题: 打印变量m:
    ```
    k: zhou value: {Name:zhou Age:24}
    k: li value: {Name:li Age:23}
    k: wang value: {Name:wang Age:22}
    ```
    因为m的值`&stu`是指向stu的,而最后stu的值是`{Name:wang Age:22}`


2. for-range 里的 go-routine 闭包捕获问题

3. 手写循环队列; 写的循环队列是不是线程安全，不是，怎么保证线程安全，加锁，效率有点低啊，然后面试官就提醒Go推崇原子操作和channel

4. 生产者消费者模式，手写代码; channel缓冲长度怎么决定，怎么控制上游生产速度过快

5. 下面的程序的运行结果是:
    ```go
    type Slice []int
    func NewSlice() Slice {
            return make(Slice, 0)
    }
    func (s* Slice) Add(elem int) *Slice {
            *s = append(*s, elem)
            fmt.Print(elem)
            return s
    }
    func main() {  
            s := NewSlice()
            defer s.Add(1).Add(2)
            s.Add(3)
    }
    ```
    132


