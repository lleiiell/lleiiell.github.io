## Go Interface 示例

### hello interface

```go
type error interface {
    Error() string
}

type MyError struct {
    When time.Time
    What string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("at %v, %s",
        e.When, e.What)
}

func TestInterfaceError(t *testing.T) {

    var err error
    err = &MyError{When: time.Now(), What: "something's wrong"}

    if err != nil {
        // at 2022-11-24 09:43:24.774546757 +0800 CST m=+0.000640449, something's wrong
        fmt.Println(err)
    }

}
```

### 断言

```go
func TestAssert(t *testing.T) {
    var itf interface{}

    itf = 1

	// i 1
    fmt.Println("i", itf.(int))

    // panic: interface conversion: interface {} is int, not int64
    // fmt.Println("i64", itf.(int64))

    s, ok := itf.(string)
    // s  true false
    fmt.Println("s", s, s == "", ok)

    // int
    switch itf.(type) {
    case int:
        fmt.Println("int")
    case string:
        fmt.Println("string")
    }
}
```

### nil

- 每个interface变量包括（type, value）两部分，value是实际变量值，type是实际变量的类型。
- interface类型变量包含了2个指针，一个指针指向值的类型，另外一个指针指向实际的值。


```go
func TestInterfaceNil(t *testing.T) {

	type helloInterface interface {
		Hello()
	}

	var i helloInterface
	// true
	fmt.Println("i == nil", i == nil)

	var u *User
	i = u

	// false
	fmt.Println("i == nil", i == nil)
	// true
	fmt.Println("u == nil", u == nil)
}
```

### interface 继承

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}

type Reader interface {
    Read(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}
```

### 参考

- https://gobyexample.com/interfaces
- https://go.dev/tour/methods/19
- https://www.geeksforgeeks.org/interfaces-in-golang/