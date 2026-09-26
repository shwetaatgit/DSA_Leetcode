# House Robber (LeetCode Medium)

## Problem

Given an array `nums` where `nums[i]` is the money in house `i`, and you can't rob two adjacent houses (a security constraint — robbing two in a row trips the alarm), return the maximum amount you can rob.

```
nums = [1,2,3,1]   -> 4   (rob house 0 and house 2: 1+3=4)
nums = [2,7,9,3,1]  -> 12  (rob house 0, 2, and 4: 2+9+1=12)
```

## How to explain it out loud

*"At every house, I have exactly two options: rob it, or skip it. If I rob house `i`, I get `nums[i]` plus whatever the best answer was up through house `i-2` — because I *can't* have robbed `i-1`. If I skip it, I just carry forward the best answer through `i-1` unchanged. So the best-through-`i` is `max(best-through-(i-1), nums[i] + best-through-(i-2))`. That's a linear DP, and since each step only looks back two positions, I don't need the whole history — just the last two running answers, updated as I go."*

## Approach 1 — Brute force recursion

At each index `i`, branch into the two choices explicitly: `rob(i) = max(nums[i] + rob(i+2), rob(i+1))` — either take this house and skip to `i+2`, or skip this house and move to `i+1`. Base case: once `i` is past the end, there's nothing left to rob, return `0`.

Time: O(2ⁿ) — two branches per call, `n` deep · Space: O(n) recursion stack.

## Approach 2 — Top-down memoization

Same recursion, caching each `rob(i)` the first time it's computed — each index only has one "best answer from here to the end," so recomputing it on every path through the tree is wasted work.

Time: O(n) — each index computed once · Space: O(n) for memo + O(n) recursion stack.

## Approach 3 — Bottom-up DP, full array

Build `dp[i]` = best amount robbable using houses `0..i`. `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])` (only one house or two — take the bigger), then `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` for `i >= 2`. Answer is `dp[n-1]`.

Time: O(n) · Space: O(n) for the array — more than needed, since `dp[i]` only ever looks back two entries.

## Approach 4 — Bottom-up DP, O(1) space (your version, optimal)

Same recurrence, but instead of a full array, track just the two running values needed: `prev` = best through two houses ago, `curr` = best through the previous house. At each step, the candidate for robbing the current house is `nums[i] + prev` (skip-one-back is baked into using `prev`, not `curr`); the new `curr` is the max of not robbing (`curr` stays) and robbing (`nums[i] + prev`). Then `prev` catches up to the old `curr` before the next iteration.

Time: O(n) — one pass · Space: O(1) — two integers, regardless of `n`.

## Solution

### C++ (your version — O(1) space, optimal)
```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.empty()) return 0;

        int prev = 0, curr = nums[0];
        for (int i = 1; i < nums.size(); ++i) {
            int newmax = nums[i] + prev;
            prev = curr;
            curr = max(curr, newmax);
        }
        return curr;
    }
};
```

### C++ (bottom-up DP, full array)
```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        if (n == 1) return nums[0];

        vector<int> dp(n);
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < n; i++) {
            dp[i] = max(dp[i-1], dp[i-2] + nums[i]);
        }
        return dp[n-1];
    }
};
```

### C++ (top-down memoization)
```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        vector<int> memo(nums.size(), -1);
        return helper(nums, 0, memo);
    }
private:
    int helper(vector<int>& nums, int i, vector<int>& memo) {
        if (i >= (int)nums.size()) return 0;
        if (memo[i] != -1) return memo[i];
        return memo[i] = max(nums[i] + helper(nums, i + 2, memo), helper(nums, i + 1, memo));
    }
};
```

### Python (O(1) space)
```python
class Solution:
    def rob(self, nums: list[int]) -> int:
        if not nums:
            return 0
        prev, curr = 0, nums[0]
        for i in range(1, len(nums)):
            new_max = nums[i] + prev
            prev = curr
            curr = max(curr, new_max)
        return curr
```

### Java (O(1) space)
```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 0) return 0;
        int prev = 0, curr = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int newMax = nums[i] + prev;
            prev = curr;
            curr = Math.max(curr, newMax);
        }
        return curr;
    }
}
```

Verified: your O(1)-space version checked against 9 hand-picked cases (both classic examples, empty array, single house, two houses in both orders, an alternating small case, all-zero houses, and a case where the optimal skips two houses in a row) plus a 2000-trial randomized stress test against a brute-force recursive reference — 0 mismatches. All three DP variants (full-array, O(1)-space, memoized) cross-checked against each other on the same 9 cases — 0 mismatches. Clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python is the same O(1)-space logic. Java is the same logic translated directly (not independently compiled in this environment — no JDK available).

## Bug log

- Your O(1)-space version: correct on the first attempt — no bugs found.
