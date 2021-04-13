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

