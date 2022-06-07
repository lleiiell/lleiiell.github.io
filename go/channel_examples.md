## Go Channel 示例

### hello Channel

- chan 发送和接收是阻塞的
- 阻塞 chan 需要并发发送、接收

```go
func TestHelloChan(t *testing.T) {
	msg := make(chan string)

	go func() {
		time.Sleep(100 * time.Millisecond)
		msg <- "hello world"
	}()

	fmt.Println("prePrint", time.Now())
	fmt.Println("msg", <-msg, time.Now())
}


// 输出：
// prePrint 2022-06-07 09:56:32.9302519 +0800 CST m=+0.003704701
// msg hello world 2022-06-07 09:56:33.0307279 +0800 CST m=+0.104180601
```

### 缓冲 Channel

- 缓冲 chan 无需并发接收

```go
func TestBuffer(t *testing.T) {
	c1 := make(chan int, 2)

	c1 <- 1
	c1 <- 2

	fmt.Println("c1", <-c1)
	fmt.Println("c1", <-c1)
}


// 输出：
// c1 1
// c1 2
```

### 单向通道

- `chan<-` 仅发送类型：发送到chan
- `<-chan` 仅接收类型：从chan接收

```go
func TestDirection(t *testing.T) {
	pings := make(chan string, 1)
	pongs := make(chan string, 1)
	go func(pings chan<- string, msg string) {
		pings <- msg

		// 无效运算: <-pings (从仅发送类型 chan<- string 接收
		//<-pings
	}(pings, "hello")

	go func(pings <-chan string, pongs chan<- string) {
		
		// 无效运算: pings <- "world" (发送到仅接收类型 <-chan string)
		//pings <- "world"
		
		msg := <-pings
		pongs <- msg
	}(pings, pongs)

	fmt.Println(<-pongs)
}


// 输出：
// hello
```

### select 

select 等待多个通道操作

```go
func TestSelect(t *testing.T) {
	c1 := make(chan string)
	c2 := make(chan string)
	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "hello"
	}()
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "world"
	}()

	fmt.Println("preFor", time.Now())
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("c1", msg1, time.Now())
		case msg2 := <-c2:
			fmt.Println("c2", msg2, time.Now())

		}
	}

	fmt.Println("done", time.Now())
}


// 输出
// preFor 2022-06-07 14:32:11.9269205 +0800 CST m=+0.005331201
// c1 hello 2022-06-07 14:32:12.9271142 +0800 CST m=+1.005524701
// c2 world 2022-06-07 14:32:13.927697 +0800 CST m=+2.006107301
// done 2022-06-07 14:32:13.927697 +0800 CST m=+2.006107301
```

select 超时

```go
func TestTimeout(t *testing.T) {
	c1 := make(chan string, 1)

	go func() {
		time.Sleep(2 * time.Second)
		c1 <- "hello"
	}()

	fmt.Println("run1", time.Now())
	select {
	case msg := <-c1:
		fmt.Println("c1", msg, time.Now())
	case <-time.After(1 * time.Second):
		fmt.Println("1s超时", time.Now())

	}

	fmt.Println("run2", time.Now())
	select {
	case msg := <-c1:
		fmt.Println("c1", msg, time.Now())
	case <-time.After(1 * time.Second):
		fmt.Println("1s超时", time.Now())
	}
	fmt.Println("done", time.Now())
}


// run1 2022-06-07 14:45:50.5729878 +0800 CST m=+0.003986401
// 1s超时 2022-06-07 14:45:51.5917107 +0800 CST m=+1.022708901
// run2 2022-06-07 14:45:51.5917107 +0800 CST m=+1.022708901
// c1 hello 2022-06-07 14:45:52.573228 +0800 CST m=+2.004225801
// done 2022-06-07 14:45:52.573228 +0800 CST m=+2.004225801
```

select default

```go
func TestDefault(t *testing.T) {
	c1 := make(chan string)

	fmt.Println("run", time.Now())
	msg := "hello"

	// r1
	select {
	case c1 <- msg:
		fmt.Println("r1 c1", <-c1, time.Now())
	default:
		fmt.Println("r1 default", time.Now())
	}

	// r2
	go func() {
		fmt.Println("r2 c1", <-c1, time.Now())
	}()
	for i := 0; i < 2; i++ {
		select {
		case c1 <- msg:
			fmt.Println("r2 msg", msg, "i", i, time.Now())
		default:
			fmt.Println("r2 default", "i", i, time.Now())
			time.Sleep(1 * time.Second)
		}

	}

	// r3
	c1 = make(chan string, 1)
	select {
	case c1 <- msg:
		fmt.Println("r3 c1", <-c1, time.Now())
	default:
		fmt.Println("r3 default", time.Now())
	}

	fmt.Println("done", time.Now())

}


// 输出：
// run 2022-06-07 15:18:33.487881 +0800 CST m=+0.003740701
// r1 default 2022-06-07 15:18:33.5067034 +0800 CST m=+0.022563001
// r2 default i 0 2022-06-07 15:18:33.5067034 +0800 CST m=+0.022563001
// r2 c1 hello 2022-06-07 15:18:34.5075343 +0800 CST m=+1.023385001
// r2 msg hello i 1 2022-06-07 15:18:34.5075343 +0800 CST m=+1.023385001
// r3 c1 hello 2022-06-07 15:18:34.5075343 +0800 CST m=+1.023385001
// done 2022-06-07 15:18:34.5075343 +0800 CST m=+1.023385001
```

- r1：无接收、无缓冲
- r2：有接收
- r3：有缓冲

### close

```go
func TestClose(t *testing.T) {
	jobs := make(chan int, 5)
	done := make(chan bool)

	fmt.Println("run", time.Now())

	go func() {
		for {
			j, more := <-jobs
			if more {
				fmt.Println("received", "job", j, time.Now())
			} else {
				fmt.Println("no more", "job", j, time.Now())

				time.Sleep(1 * time.Second)
				done <- true
				return
			}
		}
	}()

	for i := 0; i < 3; i++ {
		jobs <- i
		fmt.Println("sent job", i, time.Now())
	}

	close(jobs)
	fmt.Println("closed", time.Now())

	fmt.Println("done", <-done, time.Now())
}


// 输出：
// run 2022-06-07 15:33:47.2940415 +0800 CST m=+0.003341701
// sent job 0 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// sent job 1 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// sent job 2 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// closed 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// received job 0 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// received job 1 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// received job 2 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001
// no more job 0 2022-06-07 15:33:47.3114318 +0800 CST m=+0.020732001

// done true 2022-06-07 15:33:48.3116074 +0800 CST m=+1.020906801
```

### range

example1

```go
func TestRange(t *testing.T) {

	queue := make(chan string, 2)

	fmt.Println("run", time.Now())
	queue <- "one"
	queue <- "two"

	// fatal error: all goroutines are asleep - deadlock!
	// for item := range queue {
	// 	fmt.Println("item", item, time.Now())
	// }

	close(queue)
	fmt.Println("closed", time.Now())

	for item := range queue {
		fmt.Println("item", item, time.Now())
	}
	
	fmt.Println("done", time.Now())

}


// 输出：
// run 2022-06-07 15:40:08.9142587 +0800 CST m=+0.005389601
// closed 2022-06-07 15:40:08.9427201 +0800 CST m=+0.033851001
// item one 2022-06-07 15:40:08.9427201 +0800 CST m=+0.033851001
// item two 2022-06-07 15:40:08.9427201 +0800 CST m=+0.033851001
// done 2022-06-07 15:40:08.9427201 +0800 CST m=+0.033851001
```

example2

```go
func TestRange(t *testing.T) {

	queue := make(chan string, 2)
	done := make(chan bool)

	go func() {

		for item := range queue {
			fmt.Println("item", item, time.Now())
			time.Sleep(2 * time.Second)
		}
		done <- true
	}()

	fmt.Println("run", time.Now())
	queue <- "one"
	queue <- "two"

	time.Sleep(1 * time.Second)
	close(queue)
	fmt.Println("closed", time.Now())

	fmt.Println("done", <-done, time.Now())

}


// run 2022-06-07 15:49:26.8360505 +0800 CST m=+0.004001201
// item one 2022-06-07 15:49:26.8552004 +0800 CST m=+0.023150101
// closed 2022-06-07 15:49:27.8562438 +0800 CST m=+1.024144101
// item two 2022-06-07 15:49:28.8554174 +0800 CST m=+2.023268301
// done true 2022-06-07 15:49:30.8561188 +0800 CST m=+4.023870901
```

### 计时器 Timer

```go
func TestTimer(t *testing.T) {
	fmt.Println("run", time.Now())

	timer1 := time.NewTimer(2 * time.Second)
	t1, ok := <-timer1.C
	fmt.Println("timer1 fired", t1, ok)

	timer2 := time.NewTimer(time.Second)
	go func() {
		t2, ok := <-timer2.C
		fmt.Println("timer2 fired", t2, ok)
	}()
	ok = timer2.Stop()
	if ok {
		fmt.Println("timer2 stopped", time.Now())
	}

	time.Sleep(2 * time.Second)
	fmt.Println("done", time.Now())
}


// 输出：
// run 2022-06-07 15:59:56.0461165 +0800 CST m=+0.006866501
// timer1 fired 2022-06-07 15:59:58.0792758 +0800 CST m=+2.039971201 true
// timer2 stopped 2022-06-07 15:59:58.0794867 +0800 CST m=+2.040182101
// done 2022-06-07 16:00:00.0803209 +0800 CST m=+4.040960101
```

### 定时器 Ticker

```go
func TestTicker(t *testing.T) {
	tk := time.NewTicker(500 * time.Millisecond)
	done := make(chan bool)
	go func() {
		for {
			select {
			case t1, ok := <-tk.C:
				fmt.Println("tick at", t1, ok)
			case <-done:
				fmt.Println("go done", time.Now())
				return
			}
		}
	}()

	time.Sleep(1600 * time.Millisecond)
	tk.Stop()
	fmt.Println("tick stopped", time.Now())
	time.Sleep(1 * time.Second)
	done <- true
	fmt.Println("done", time.Now())
}


// 输出：
// tick at 2022-06-07 16:31:01.3723516 +0800 CST m=+0.503823301 true
// tick at 2022-06-07 16:31:01.8720139 +0800 CST m=+1.003485401 true
// tick at 2022-06-07 16:31:02.3729422 +0800 CST m=+1.504413501 true
// tick stopped 2022-06-07 16:31:02.4725571 +0800 CST m=+1.604028301
// done 2022-06-07 16:31:03.4729822 +0800 CST m=+2.604453001
// go done 2022-06-07 16:31:03.47315 +0800 CST m=+2.604620801
```

### 工作池 Worker pool

3个 goroutine 花费 2s 处理总量 5s 的作业

```go
func TestWorkerPool(t *testing.T) {
	const numJobs = 5
	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	fmt.Println("run", time.Now())

	// 3 workers
	for w := 1; w <= 3; w++ {
		go func(id int, jobs <-chan int, results chan<- int) {
			for j := range jobs {
				fmt.Println("worker", id, "started job", j, time.Now())
				time.Sleep(time.Second)
				fmt.Println("worker", id, "finished job", j, time.Now())
				results <- j * 2
			}
		}(w, jobs, results)
	}

	// 作业请求
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)
	fmt.Println("jobs closed", time.Now())

	// 确保 goroutines 全部完成
	for a := 1; a <= numJobs; a++ {
		r := <-results
		fmt.Println("results", r, time.Now())
	}

	fmt.Println("done", time.Now())
}


// 输出：
// run 2022-06-07 19:27:13.7973141 +0800 CST m=+0.002804801
// jobs closed 2022-06-07 19:27:13.8489983 +0800 CST m=+0.054488901
// worker 3 started job 3 2022-06-07 19:27:13.8489983 +0800 CST m=+0.054488901
// worker 1 started job 1 2022-06-07 19:27:13.8489983 +0800 CST m=+0.054488901
// worker 2 started job 2 2022-06-07 19:27:13.849533 +0800 CST m=+0.055023601
// worker 3 finished job 3 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// worker 1 finished job 1 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// results 6 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// results 2 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// worker 3 started job 4 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// worker 1 started job 5 2022-06-07 19:27:14.8491952 +0800 CST m=+1.054684001
// worker 2 finished job 2 2022-06-07 19:27:14.8506233 +0800 CST m=+1.056112101
// results 4 2022-06-07 19:27:14.8506233 +0800 CST m=+1.056112101
// worker 1 finished job 5 2022-06-07 19:27:15.8509527 +0800 CST m=+2.056439701
// results 10 2022-06-07 19:27:15.8509527 +0800 CST m=+2.056439701
// worker 3 finished job 4 2022-06-07 19:27:15.8509527 +0800 CST m=+2.056439701
// results 8 2022-06-07 19:27:15.8509527 +0800 CST m=+2.056439701
// done 2022-06-07 19:27:15.8509527 +0800 CST m=+2.056439701
```

### 限流 Rate Limit

```go
func TestRateLimit(t *testing.T) {
	requests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		requests <- i
	}
	close(requests)

	limiter := time.Tick(200 * time.Millisecond)

	for req := range requests {
		<-limiter
		fmt.Println("request", req, time.Now())
	}

	burstyLimiter := make(chan time.Time, 3)
	for i := 0; i < 3; i++ {
		burstyLimiter <- time.Now()
	}
	go func() {
		for t := range time.Tick(200 * time.Millisecond) {
			burstyLimiter <- t
		}
	}()

	burstyRequests := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		burstyRequests <- i
	}
	close(burstyRequests)
	for req := range burstyRequests {
		<-burstyLimiter
		fmt.Println("request", req, time.Now())
	}
}


// 输出：
// request 1 2022-06-07 19:59:21.3987663 +0800 CST m=+0.205139901
// request 2 2022-06-07 19:59:21.5989678 +0800 CST m=+0.405339601
// request 3 2022-06-07 19:59:21.7997344 +0800 CST m=+0.606104401
// request 4 2022-06-07 19:59:21.9988399 +0800 CST m=+0.805208101
// request 5 2022-06-07 19:59:22.200215 +0800 CST m=+1.006581301

// request 1 2022-06-07 19:59:22.2002936 +0800 CST m=+1.006659901
// request 2 2022-06-07 19:59:22.2002936 +0800 CST m=+1.006659901
// request 3 2022-06-07 19:59:22.2002936 +0800 CST m=+1.006659901

// request 4 2022-06-07 19:59:22.4009163 +0800 CST m=+1.207280801
// request 5 2022-06-07 19:59:22.6009971 +0800 CST m=+1.407359801
```

### 死锁

- chan需要并发生产、消费


deadlock1

```go
func TestDeadlock(t *testing.T) {
	c1 := make(chan int)
	c1 <- 1
}


// 输出：
// fatal error: all goroutines are asleep - deadlock!
```


deadlock2

```go
func TestDeadlock(t *testing.T) {
	c1 := make(chan int)
	<-c1
}


// 输出：
// fatal error: all goroutines are asleep - deadlock!
```

## 参考

- [channels - gobyexample.com](https://gobyexample.com/channels)