# LRU 缓存（最近最少使用）

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

```go
type LRUCache struct {
    capacity int
    lru *list.List
    items map[int]*list.Element
}

type item struct{
    key int
    value int
}

func Constructor(capacity int) LRUCache {
    return LRUCache{
        capacity: capacity,
        lru: list.New(),
        items: make(map[int]*list.Element),
    }
}

func (this *LRUCache) Get(key int) int {
    element, ok := this.items[key]
    if !ok{
        return -1
    }
    // 将刚访问过的元素移动到队首
    this.lru.MoveToFront(element)
    // 先进行断言，再把 value 返回
    return element.Value.(*item).value
}

func (this *LRUCache) Put(key int, value int)  {
    element, exists := this.items[key]
    if exists{
        this.lru.MoveToFront(element)
        // 进行断言后重新赋值
        element.Value.(*item).value = value
    }else{
        if this.lru.Len() >= this.capacity{
            // 驱逐最近最不常用的元素，链表最后一个元素
            delete(this.items, this.lru.Back().Value.(*item).key)
            this.lru.Remove(this.lru.Back())
        }
        element = this.lru.PushFront(&item{key:key, value:value})
        this.items[key] = element
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```