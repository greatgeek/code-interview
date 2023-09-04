# LFU 缓存（最不经常使用）

缓存算法需要维护三个映射结构：

1. keys：保存每个关键键之间的映射关系；
2. frequency：保存每个频率对应的键值；
3. keyValue：保存键与值之间的映射。

然后，最为重要的就是维护这三个映射结构之间的关系。代码如下：

```go
type LFUCache struct {
    minFreq int
    cap int
    key2Item map[int]*list.Element
    key2Freq map[int]int
    freq2Items map[int]*list.List
}

type Item struct{
    key int
    value int
}


func Constructor(capacity int) LFUCache {
    return LFUCache{
        cap: capacity,
        key2Item: make(map[int]*list.Element),
        key2Freq: make(map[int]int),
        freq2Items: make(map[int]*list.List),
    }
}


func (this *LFUCache) Get(key int) int {
    if item, ok := this.key2Item[key]; ok{
        this.IncreaseFrequency(key)
        return item.Value.(*Item).value
    }
    return -1
}


func (this *LFUCache) Put(key int, value int)  {
    if this.cap == 0{
        return
    }
    if _, ok := this.key2Item[key]; ok {
        this.key2Item[key].Value.(*Item).value = value
        this.IncreaseFrequency(key)
    }else{
        if len(this.key2Item) == this.cap{
            this.RemoveMinFreqNode()
        }
        item := &Item{key: key, value:value}
        if this.freq2Items[1] == nil {
            this.freq2Items[1] = list.New()
        }
        this.key2Item[key] = this.freq2Items[1].PushFront(item)
        this.key2Freq[key] = 1
        this.minFreq = 1
    }
}

func (this *LFUCache) IncreaseFrequency(key int) {
    freq := this.key2Freq[key]
    this.freq2Items[freq].Remove(this.key2Item[key])

    this.key2Freq[key]++
    if this.freq2Items[this.key2Freq[key]] == nil{
        this.freq2Items[this.key2Freq[key]] = list.New()
    }
    this.key2Item[key] = this.freq2Items[this.key2Freq[key]].PushFront(this.key2Item[key].Value.(*Item))

    if this.freq2Items[freq].Len() == 0{
        delete(this.freq2Items, freq)
        if this.minFreq == freq {
            this.minFreq++
        }
    }
}

func (this *LFUCache) RemoveMinFreqNode(){
    item := this.freq2Items[this.minFreq].Back()
    this.freq2Items[this.minFreq].Remove(item)
    delete(this.key2Item, item.Value.(*Item).key)
    delete(this.key2Freq, item.Value.(*Item).key)
}


/**
 * Your LFUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```

