## Go Interface 示例

### hello interface

```go
type User struct {
    Name string
}

func (u *User) Hello() {
    fmt.Println("hello ", u.Name)
}

func TestInterfaceHello(t *testing.T) {
    type helloInterface interface {
        Hello()
    }

    var i helloInterface
    i = &User{Name: "lt"}
    // hello  lt
    i.Hello()
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