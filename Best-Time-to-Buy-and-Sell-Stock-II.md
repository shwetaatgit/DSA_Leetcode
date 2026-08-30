# Best Time to Buy and Sell Stock II (LeetCode Medium)

## Problem

Same as the basic version, but now unlimited transactions are allowed — buy and sell repeatedly, just can't hold more than one share at a time (must sell before buying again). Return max total profit.

```
prices = [7,1,5,3,6,4]
-> 7   (buy@1 sell@5 = +4, buy@3 sell@6 = +3, total 7)
```

## How to explain it out loud

*"Since I can transact as many times as I want, I don't need to find specific buy/sell days at all — I just sum up every day-to-day price increase. Any time tomorrow's price is higher than today's, that gap is profit I could have captured by buying today and selling tomorrow. Adding up every positive gap across the whole array gives the same total as optimally choosing buy/sell pairs, because a multi-day uphill run and 'buy at the bottom, sell at the top of that run' produce identical profit — summing daily deltas is just a different way to compute the same number. One pass, O(n) time, O(1) space."*

## Approach

Track the previous day's price. Each day, if today's price is higher than the previous day's, add the difference to the running total — that's profit as if bought yesterday and sold today. Move the "previous price" pointer forward regardless.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int maxProfitVal = 0;
        int start = prices[0];
        int len = prices.size();
        for (int i = 1; i < len; i++) {
            if (start < prices[i]) {
                maxProfitVal += prices[i] - start;
            }
            start = prices[i];
        }
        return maxProfitVal;
    }
};
```

### Python
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        start = prices[0]
        for i in range(1, len(prices)):
            if start < prices[i]:
                max_profit += prices[i] - start
            start = prices[i]
        return max_profit
```

### Java
```java
class Solution {
    public int maxProfit(int[] prices) {
        int maxProfitVal = 0;
        int start = prices[0];
        for (int i = 1; i < prices.length; i++) {
            if (start < prices[i]) {
                maxProfitVal += prices[i] - start;
            }
            start = prices[i];
        }
        return maxProfitVal;
    }
}
```

Verified against 5 cases (multiple uphill runs, continuous rise, all-decreasing/no profit, single day, mixed run with a big spike) — C++ additionally clean under AddressSanitizer.

## Bug log

- Correct on first attempt algorithmically. Only fix: the variable was originally named `max`, which shadows `std::max` from `<algorithm>` — same class of naming mistake as `floor` (Majority Element) and Python's `dict` (Two Sum), now the third time this pattern has come up. Renamed to `maxProfitVal`.
