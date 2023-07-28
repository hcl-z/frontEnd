---
nav:
  title: LeetCode
  order: 3
group:
  title: 动态规划
  order: 23
title: 122 - 买卖股票的最佳时机 II
order: 122
---

# 买卖股票的最佳时机 II

`动态规划`

给定一个数组，它的第 $i$ 个元素是一支给定股票第 $i$ 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```plain
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

**示例 2:**

```plain
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
  注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
  因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

**示例  3:**

```plain
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

**提示：**

- `1 <= prices.length <= 3 \* 10 ^ 4`
- `0 <= prices[i] <= 10 ^ 4`

## 解题思路

我们必须确定通过交易能够获得的最大利润（对于交易次数没有限制）。为此，我们需要找出那些共同使得利润最大化的买入及卖出价格。

### 暴力搜索

### 贪心算法

贪心算法在每步总是做出在当前看来最好的选择。

- 贪心算法、动态规划和鹅回溯算法一样，完成一件事情，是分步决策的
- 贪心算法在每一步总是做出在当前看来最好的选择，**最好** 这两字的理解
  1. **最好** 的意思往往根据题目而来，可能是 **最小**，也可能是 **最大**
  2. 贪心算法和动态规划相比，它既不看前面（也就是说它不需要从前面的状态转移过来），也不看后面（无后效性，后面的选择不会对前面的选择有影响），因此贪心算法时间复杂度一般是线性的，空间复杂度是常数级别的
  3. 这道题 **贪心** 的地方在于，对于 `今天的股价 - 昨天的股价`，得到的结果有 3 种可能：正数、0 和负数

```js
const maxProfit = function(prices) {
  let profit = 0;
  for (let i = 1; i < pricies.length; i++) {
    if (prices[i] > prices[i - 1]) {
      profit += prices[i] - prices[i - 1];
    }
  }
  return profit;
};
```

复杂度分析：

- 时间复杂度 $O(N)$：只需遍历一次 `prices`
- 空间复杂度 $O(1)$：变量使用常数额外空间

### 动态规划

用到贪心算法解决的问题，一般情况下都可以用动态规划。因此，不妨从 **状态**、**状态转移方程** 的角度考虑。

**第一步：定义状态**

状态 `dp[i][j]` 定义如下：

- $i$：表示前 $i$ 天能获得的最大利润
- $j$：表示第 $i$ 天是否持有股票的状态，$0$ 表示没有持有股票，$1$ 表示持有股票

**第二步：状态转移方程**

- 状态从没有持有股票开始，到最后一天我们关心的状态依然是没有持有股票 $dp[i][0]$
- 每天状态可以发生转移，也可以不做任何操作

说明：

- 因为不限制交易次数，除了最后一天，每天的状态可能不变化，也可能转移
- 写代码的时候，可以不用对最后一天单独处理，输出最后一天，状态为 $0$ 的时候的值即可

**第三步：确定起始**

起始的时候：

- 如果什么都不做：`dp[0][0] = 0`
- 如果买入股票，当前收益为负数，即 `dp[0][1] = -prices[i]`

**第四步：确定终止**

终止的时候，上面也分析了，输出 `dp[len - 1][0]`，因为一定有 `dp[len - 1][0] > dp[len - 1][1]`

```js
const maxProfit = function(prices) {
  let len = prices.length;
  if (len < 2) return 0;

  let dp = [[0, -prices[0]]];

  for (let i = 1; i < len; i++) {
    if (!dp[i]) dp[i] = [];
    dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
  }

  return dp[len - 1][0];
};
```

复杂度分析：

- 时间复杂度 $O(N)$，这里 $N$ 表示股价数组的长度
- 空间复杂度 $O(N)$，虽然是二维数组，但是第二维是常数，与问题规模无关

我们也可以将状态数组分开设置，语义更明确。

```js
const maxProfit = function(prices) {
  let len = prices.length;
  if (len < 2) return 0;

  let cash = Array(len),
    stock = Array(len);

  cash[0] = 0;
  stock[0] = -prices[0];

  for (let i = 1; i < len; i++) {
    cash[i] = Math.max(cash[i - 1], stock[i - 1] + prices[i]);
    stock[i] = Math.max(stock[i - 1], cash[i - 1] - prices[i]);
  }

  return cash[len - 1];
};
```

优化空间：由于当前行只参考上一行，每行就两个值，因此可以考虑使用 **滚动变量**（滚动数组的技巧）。

```js
const maxProfit = function() {
  let len = prices.length;
  if (len < 2) return 0;
};
```

---

**参考资料：**

- [暴力搜索、贪心算法、动态规划](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/solution/tan-xin-suan-fa-by-liweiwei1419-2/)