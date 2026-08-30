# Best Time to Buy and Sell Stock (LeetCode Easy)

## Problem

`prices[i]` is the stock price on day `i`. Buy once, sell once, sell must come after buy. Return the max profit, or `0` if no profit is possible.

```
prices = [7,1,5,3,6,4]
-> 5   (buy at 1, sell at 6)
```

## How to explain it out loud

*"Brute force is try every buy day against every later sell day — O(n²). But you don't actually need to compare every pair: at any given day, all that matters is the best price on the other side of it. I scan once from the back, keeping a running max of 'best price seen so far to the right.' At each day, the best possible profit if I bought today is that running max minus today's price. I take the best of those across the whole scan, and update the running max as I go. One pass, O(n) time, O(1) space."*

That's the whole pitch — running extremum tracked in one direction, profit checked against it before it's updated with the current element.

## Approaches

**1. Brute force** — nested loop, every `(buy day i, sell day j>i)` pair, track the best `prices[j] - prices[i]`. O(n²) time, O(1) space.

**2. Running max from the back (optimal)** — walk right to left. `maxPrice` holds the highest price seen so far *to the right* of the current day. At each day, check `maxPrice - prices[i]` (selling later, buying today) *before* folding today's price into `maxPrice` — order matters, otherwise you'd let a day "sell to itself."

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int profit = 0;
        int maxPrice = 0;
        for (int i = prices.size() - 1; i >= 0; i--) {
            profit = max(profit, maxPrice - prices[i]);
            maxPrice = max(maxPrice, prices[i]);
        }
        return profit;
    }
};
```

### Python
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        max_price = 0
        for i in range(len(prices) - 1, -1, -1):
            profit = max(profit, max_price - prices[i])
            max_price = max(max_price, prices[i])
        return profit
```

### Java
```java
class Solution {
    public int maxProfit(int[] prices) {
        int profit = 0;
        int maxPrice = 0;
        for (int i = prices.length - 1; i >= 0; i--) {
            profit = Math.max(profit, maxPrice - prices[i]);
            maxPrice = Math.max(maxPrice, prices[i]);
        }
        return profit;
    }
}
```

Verified against 5 cases (classic, small rise, all-decreasing/no profit, two-day, dip-then-rise) — C++ additionally clean under AddressSanitizer.

## Bug log

- First attempt at the O(n²) brute force had the subtraction backwards (`prices[i]-prices[j]` instead of `prices[j]-prices[i]`) — this doesn't error or look obviously wrong, it silently computes the best price *drop* (as if short-selling) instead of the best rise. Caught by tracing a specific `(i,j)` pair by hand and seeing sell-before-buy.
- Outer loop bound `i < n-2` cut off the last valid buy day; needed `i < n-1`.
- A converging two-pointer ("sliding window," high/low closing inward) doesn't apply to this problem — there's no rule for safely discarding a boundary the way there is in problems like Container With Most Water, since the best buy/sell days can be anywhere in the middle.
