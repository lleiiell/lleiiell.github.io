## Go context 示例

### 介绍

context 主要用来在 goroutine 之间传递上下文信息，包括：取消信号、超时时间、截止时间、k-v 等。

随着 context 包的引入，标准库中很多接口因此加上了 context 参数，例如 database/sql 包。context 几乎成为了并发控制和超时控制的标准做法。

```go
type Context interface {
    // 返回 context.Context 取消的时间，也就是完成工作的截止日期；
    Deadline() (deadline time.Time, ok bool)

    // 在Context超时或取消时（即结束了）返回一个关闭的channel
    // 即如果当前Context超时或取消时,Done方法会返回一个channel，
    // 然后其他地方就可以通过判断Done方法是否有返回（channel）,如果有则说明Context已结束
    // 故其可以作为广播通知其他相关方本Context已结束,请做相关处理.
    Done() <-chan struct{}

    // 返回Context取消的原因
    // 1、如果 context.Context 取消，会返回 Canceled 错误；
    // 2、如果 context.Context 超时，会返回 DeadlineExceeded 错误；
    Err() error

    // 返回Context相关数据
    Value(key interface{}) interface{}
}
```

### 特点、实践

特点

- 可发送取消信号
- 可设置超时时间、截止时间
- 并发安全
- 接口方法幂等，多次调用，结果相同。

实践

- 不要将 Context 塞到结构体里。直接将 Context 类型作为函数的第一参数，一般命名为 ctx
- 不要向函数传入 nil 的 context，备选 context.TODO(), context.Background
- 不要把本应作为函数参数的类型塞到context中，context 存储的应该是一些共同的数据。例如：token等
- 同一个 context 可能被传到多个 goroutine，context是并发安全的

### 超时取消 WithTimeout()

```go
func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()

    fmt.Println("pre handle", "time", time.Now())

    go handle(ctx, 500*time.Millisecond)
    go handle(ctx, 1500*time.Millisecond)

    select {
    case <-ctx.Done():
        fmt.Println("main", ctx.Err())
    }

    fmt.Println("pre sleep", "time", time.Now())
    time.Sleep(1 * time.Second)
    fmt.Println("all done", "time", time.Now())
}

func handle(ctx context.Context, duration time.Duration) {
    select {
    case <-ctx.Done():
        dl, ok := ctx.Deadline()
        fmt.Println("handle", ctx.Err(), "deadline", dl, "ok", ok)
    case <-time.After(duration):
        dl, ok := ctx.Deadline()
        fmt.Println("process request with", duration, "deadline", dl, "ok", ok)
    }
}


// 输出：

// pre handle time 2022-06-06 11:10:53.55862 +0800 CST m=+0.003721901
// process request with 500ms deadline 2022-06-06 11:10:54.55862 +0800 CST m=+1.003721901 ok true
// main context deadline exceeded
// pre sleep time 2022-06-06 11:10:54.5592465 +0800 CST m=+1.004337601
// handle context deadline exceeded deadline 2022-06-06 11:10:54.55862 +0800 CST m=+1.003721901 ok true
// all done time 2022-06-06 11:10:55.5594567 +0800 CST m=+2.004537001
```

- context 开始时间：2022-06-06 11:10:53.55862
- context 截至时间：2022-06-06 11:10:54.55862
- main 先于 goroutine 收到关闭的channel

### TODO() 和 Background()

- TODO() 和 Background() 返回 emptyCtx
- emptyCtx 没有截止时间

```go
func Background() Context {
    return background
}

func TODO() Context {
    return todo
}

type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}

func (e *emptyCtx) String() string {
    switch e {
    case background:
        return "context.Background"
    case todo:
        return "context.TODO"
    }
    return "unknown empty Context"
}

var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

```

### WithCancel()

```go
func TestWithCancel(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    go func(ctx context.Context) {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("ctxDone...", time.Now())
                return
            default:
                fmt.Println("defaultContinue...", time.Now())
                time.Sleep(1 * time.Second)
            }
        }
    }(ctx)

    time.Sleep(5 * time.Second)
    fmt.Println("preCancel", time.Now())
    cancel()
    fmt.Println("Canceled", time.Now())

    time.Sleep(1 * time.Second)
    fmt.Println("done", time.Now())
}


// defaultContinue... 2022-06-06 19:54:48.5304652 +0800 CST m=+0.012911901
// defaultContinue... 2022-06-06 19:54:49.5521356 +0800 CST m=+1.034574301
// defaultContinue... 2022-06-06 19:54:50.5529818 +0800 CST m=+2.035412701
// defaultContinue... 2022-06-06 19:54:51.5536748 +0800 CST m=+3.036097901
// defaultContinue... 2022-06-06 19:54:52.5538833 +0800 CST m=+4.036298601
// preCancel 2022-06-06 19:54:53.5306726 +0800 CST m=+5.013080301
// Canceled 2022-06-06 19:54:53.5306726 +0800 CST m=+5.013080301
// ctxDone... 2022-06-06 19:54:53.5559938 +0800 CST m=+5.038401301
// done 2022-06-06 19:54:54.5311194 +0800 CST m=+6.013519301
```

### WithValue()

```go
func TestWithValue(t *testing.T) {
    ctx := context.WithValue(context.Background(), "myKey", "myVal")

    go func(ctx context.Context) {
        fmt.Println("ctx", "myKey", ctx.Value("myKey"), time.Now())
    }(ctx)

    time.Sleep(1 * time.Second)
    fmt.Println("done", time.Now())
}


// ctx myKey myVal 2022-06-06 20:04:42.9522581 +0800 CST m=+0.015779801
// done 2022-06-06 20:04:43.9535771 +0800 CST m=+1.017094001
```


## 参考

- [深度解密Go语言之context](https://zhuanlan.zhihu.com/p/68792989)
- [上下文 Context - draveness.me](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/)
- [如何在 Go 中使用上下文 - digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-use-contexts-in-go)