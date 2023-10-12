[123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

### 方法一：动态规划

**思路与算法**

在任意一天结束后，你可以处于以下五个状态之一：

1. 未进行过任何操作；
2. 只进行过一次买操作；
3. 完成了一笔交易（买入并卖出）；
4. 在完成了一笔交易后，进行了第二次买操作；
5. 完成了两笔交易。

我们可以使用动态规划来描述状态转移过程，用 `buy1, sell1, buy2, sell2` 分别表示上述的四个状态的最大利润。

- 对于 `buy1`，你可以选择不操作，或者买入股票，所以状态转移为：
  ```markdown
  buy1 = max(buy1, -prices[i])
  ```

- 对于 `sell1`，你可以选择不操作，或者卖出股票（前提是你在之前买入过），所以状态转移为：
  ```markdown
  sell1 = max(sell1, buy1 + prices[i])
  ```

- 对于 `buy2`，你可以选择不操作，或者在完成了一笔交易后再买入股票，所以状态转移为：
  ```markdown
  buy2 = max(buy2, sell1 - prices[i])
  ```

- 对于 `sell2`，你可以选择不操作，或者完成第二次的股票交易，所以状态转移为：
  ```markdown
  sell2 = max(sell2, buy2 + prices[i])
  ```

**边界条件**：

当 `i=0` 时（第一天），`buy1` 代表买入股票，因此 `buy1 = -prices[0]`；`sell1` 代表在同一天买入并卖出，所以 `sell1 = 0`；同样的道理，`buy2 = -prices[0]` 和 `sell2 = 0`。

进行动态规划后，`sell2` 将会给出进行两次交易所能获得的最大利润。由于你可以选择交易少于两次，所以最终的答案即为 `sell2`。



```go
func maxProfit(prices []int) int {
    buy1, sell1 := -prices[0], 0
    buy2, sell2 := -prices[0], 0
    for i := 1; i < len(prices); i++ {
        buy1 = max(buy1, -prices[i])
        sell1 = max(sell1, buy1+prices[i])
        buy2 = max(buy2, sell1-prices[i])
        sell2 = max(sell2, buy2+prices[i])
    }
    return sell2
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

