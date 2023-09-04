# 实现阻塞读且并发安全的 map

GO里面MAP如何实现key不存在 get操作等待 直到key存在或者超时，保证并发安全，且需要实现以下接口：

```go
type sp interface {
    Put(key string, val interface{})  //存入key /val，如果该key读取的goroutine挂起，则唤醒。此方法不会阻塞，时刻都可以立即执行并返回
    Get(key string, timeout time.Duration) interface{}  //读取一个key，如果key不存在阻塞，等待key存在或者超时
}
```

**解析：**

看到阻塞协程第一个想到的就是`channel`，题目中要求并发安全，那么必须用锁，还要实现多个`goroutine`读的时候如果值不存在则阻塞，直到写入值，那么每个键值需要有一个阻塞`goroutine` 的 `channel`。

```go
package main

import (
	"sync"
	"time"
)

type SafeMap struct {
	mu    sync.Mutex
	data  map[string]interface{}
	chans map[string]chan struct{}
}

type sp interface {
	Put(key string, val interface{})                   //存入key /val，如果该key读取的goroutine挂起，则唤醒。此方法不会阻塞，时刻都可以立即执行并返回
	Get(key string, timeout time.Duration) interface{} //读取一个key，如果key不存在阻塞，等待key存在或者超时
}

var _ sp = (*SafeMap)(nil)

func NewSafeMap() *SafeMap {
	return &SafeMap{
		data:  make(map[string]interface{}),
		chans: make(map[string]chan struct{}),
	}
}

func (sm *SafeMap) Put(key string, val interface{}) {
	sm.mu.Lock()
	sm.data[key] = val
	if ch, ok := sm.chans[key]; ok {
		close(ch)
		delete(sm.chans, key)
	}
	sm.mu.Unlock()
}

func (sm *SafeMap) Get(key string, timeout time.Duration) interface{} {
	sm.mu.Lock()
	if val, ok := sm.data[key]; ok {
		sm.mu.Unlock()
		return val
	}

	ch, ok := sm.chans[key]
	if !ok {
		ch = make(chan struct{})
		sm.chans[key] = ch
	}
	sm.mu.Unlock()

	select {
	case <-ch:
		sm.mu.Lock()
		val := sm.data[key]
		sm.mu.Unlock()
		return val
	case <-time.After(timeout):
		return nil
	}
}

func main() {
	safeMap := NewSafeMap()

	go func() {
		time.Sleep(2 * time.Second)
		safeMap.Put("key1", "value1")
	}()

	result := safeMap.Get("key1", 5*time.Second)
	if result != nil {
		println("Found key1:", result.(string))
	} else {
		println("Timeout waiting for key1")
	}
}

```

