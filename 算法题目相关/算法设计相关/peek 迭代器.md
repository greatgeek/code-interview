# peek 迭代器

请你在设计一个迭代器，在集成现有迭代器拥有的 `hasNext` 和 `next` 操作的基础上，还额外支持 `peek` 操作。

实现 `PeekingIterator` 类：

- `PeekingIterator(Iterator<int> nums)` 使用指定整数迭代器 `nums` 初始化迭代器。
- `int next()` 返回数组中的下一个元素，并将指针移动到下个元素处。
- `bool hasNext()` 如果数组中存在下一个元素，返回 `true` ；否则，返回 `false` 。
- `int peek()` 返回数组中的下一个元素，但 **不** 移动指针。

**注意：**每种语言可能有不同的构造函数和迭代器 `Iterator`，但均支持 `int next()` 和 `boolean hasNext()` 函数。

```go
/*   Below is the interface for Iterator, which is already defined for you.
 *
 *   type Iterator struct {
 *       
 *   }
 *
 *   func (this *Iterator) hasNext() bool {
 *		// Returns true if the iteration has more elements.
 *   }
 *
 *   func (this *Iterator) next() int {
 *		// Returns the next element in the iteration.
 *   }
 */

type PeekingIterator struct {
    iter *Iterator
    _hasNext bool
    _next int
}

func Constructor(iter *Iterator) *PeekingIterator {
    return &PeekingIterator{iter, iter.hasNext(), iter.next()}
}

func (this *PeekingIterator) hasNext() bool {
    return this._hasNext
}

func (this *PeekingIterator) next() int {
    ret := this._next
    this._hasNext = this.iter.hasNext()
    if this._hasNext{
        this._next = this.iter.next()
    }
    return ret
}

func (this *PeekingIterator) peek() int {
    return this._next
}
```

