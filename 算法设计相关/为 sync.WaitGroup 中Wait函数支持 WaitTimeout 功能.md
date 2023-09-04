# 为 sync.WaitGroup 中Wait函数支持 WaitTimeout 功能

```go
import (
	"fmt"
	"sync"
	"time"
)

func waitTimeout(wg *sync.WaitGroup, timeout time.Duration) bool {
	// 要求手写代码
	// 要求 sync.WaitGroup 支持 timeout 功能
	// 若 timeout 到了超时时间返回 true
	// 若 WaitGroup 自然结束返回 false

	ch := make(chan bool)
	go time.AfterFunc(timeout, func() {
		ch <- true
	})
	go func() {
		wg.Wait()
		ch <- false
	}()
	return <-ch
}

func WaitTimeoutTest() {
	wg := sync.WaitGroup{}
	ch := make(chan struct{})
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(num int, close <-chan struct{}) {
			defer wg.Done()
			<-close
			fmt.Println(num)
		}(i, ch)
	}

	if waitTimeout(&wg, time.Second*5) {
		close(ch)
		fmt.Println("timeout exit")
	}
	time.Sleep(time.Second * 10)
}

```

