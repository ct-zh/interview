## go面试问题:代码相关
### 以下代码有什么问题，说明原因
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

### 下面的程序的运行结果是:
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


### 判断下面函数的输出
```go
func f1() (r int) {
	defer func() {
		r++
	}()
	return 0
}

func f2() (r int) {
	t := 5
	defer func() {
		r += 5
	}()
	return t
}

func f3() (t int) {
	t = 5
	defer func() {
		t += 5
	}()
	return t
}

func f4() (r int) {
	r = 1
	defer func(r int) {
		r = r + 5
	}(r)
	return
}

func DeferFunc1(i int) (t int) {
	t = i
	defer func() {
		t += 3
	}()
	return t
}

func DeferFunc2(i int) int {
	t := i
	defer func() {
		t += 3
	}()
	return t
}

func DeferFunc3(i int) (t int) {
	defer func() {
		t += i
	}()
	return 2
}

func DeferFunc4() (t int) {
	defer func(i int) {
		fmt.Println(i)
		fmt.Println(t)
	}(t)
	t = 1
	return 2
}
```