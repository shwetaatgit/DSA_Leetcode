# Climbing Stairs (LeetCode Easy)

## Problem

You're climbing a staircase of `n` steps. Each move you can climb either 1 or 2 steps. Return the number of distinct ways to reach the top.

```
n = 2 -> 2   (1+1, 2)
n = 3 -> 3   (1+1+1, 1+2, 2+1)
```

## How to explain it out loud

*"The number of ways to reach step `n` depends only on how you could have arrived there: either you were at step `n-1` and took one more single step, or you were at step `n-2` and took one more double step. Those are the only two options and they're mutually exclusive, so `ways(n) = ways(n-1) + ways(n-2)` — that's Fibonacci. I build it bottom-up: `dp[0]=1` (one way to 'reach' the ground — do nothing), `dp[1]=1` (one step, one way), then each `dp[i]` is just the sum of the two before it, up to `dp[n]`."*

## Approach 1 — Brute force recursion

Directly recurse: `climb(n) = climb(n-1) + climb(n-2)`, with base cases `climb(0) = 1` and negative `n` contributing `0` (can't land exactly on a negative step). Correct, but recomputes the same subproblems exponentially many times — e.g. `climb(5)` calls `climb(3)` twice, `climb(2)` three times, and so on.

Time: O(2ⁿ) — the recursion tree branches into two calls per level, `n` levels deep, giving a Fibonacci-shaped call tree with roughly `2ⁿ` total calls · Space: O(n) for the recursion stack depth.

## Approach 2 — Top-down memoization

Same recursion, but cache each `climb(n)` result the first time it's computed, so repeat calls for the same `n` return instantly instead of re-branching.

Time: O(n) — each distinct `n` from `0` to the input is computed exactly once · Space: O(n) for the memo table plus O(n) recursion stack.

## Approach 3 — Bottom-up DP, full array (your version)

Same recurrence as Fibonacci, built iteratively instead of recursively: a `dp` array where `dp[i]` holds the number of ways to reach step `i`, filled left to right from the base cases `dp[0]=dp[1]=1` up to `dp[n]`. No recursion, no repeated subproblem calls — each `dp[i]` is computed exactly once, directly from the two entries before it.

Time: O(n) — one pass, one O(1) addition per step · Space: O(n) for the full `dp` array — more than strictly necessary, since only the last two values are ever read at any point.

## Approach 4 — Bottom-up DP, O(1) space (optimal)

Since `dp[i]` only ever depends on `dp[i-1]` and `dp[i-2]`, there's no need to keep the whole array — just carry the last two values forward in two variables, updating them each step.

Time: O(n) — same single pass · Space: O(1) — only two integers in flight at any point, regardless of `n`.

## Solution

### C++ (your version — bottom-up DP, full array)
```cpp
class Solution {
public:
    int climbStairs(int n) {
        if (n == 0) return 1;
        if (n == 1) return 1;

        vector<int> dp (n+1,0);
        dp[0] = 1; dp[1] = 1;

        for(int i = 2; i<=n; i++){
            dp[i] = dp[i-1]+dp[i-2];
        }

        return dp[n];
    }
};
```

### C++ (O(1) space — optimal)
```cpp
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1) return 1;
        int prev2 = 1, prev1 = 1;
        for (int i = 2; i <= n; i++) {
            int cur = prev1 + prev2;
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
};
```

### C++ (top-down memoization)
```cpp
class Solution {
public:
    int climbStairs(int n) {
        vector<int> memo(n + 1, -1);
        return helper(n, memo);
    }
private:
    int helper(int n, vector<int>& memo) {
        if (n == 0) return 1;
        if (n == 1) return 1;
        if (memo[n] != -1) return memo[n];
        return memo[n] = helper(n - 1, memo) + helper(n - 2, memo);
    }
};
```

### Python (O(1) space)
```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 1:
            return 1
        prev2, prev1 = 1, 1
        for _ in range(2, n + 1):
            prev2, prev1 = prev1, prev1 + prev2
        return prev1
```

### Java (O(1) space)
```java
class Solution {
    public int climbStairs(int n) {
        if (n <= 1) return 1;
        int prev2 = 1, prev1 = 1;
        for (int i = 2; i <= n; i++) {
            int cur = prev1 + prev2;
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
}
```

Verified: bottom-up array version checked against brute-force recursion for `n = 1..25` — 0 mismatches. All three DP variants (full-array, O(1)-space, memoized) cross-checked against each other for every `n` from `0` to `45` (LeetCode's stated max) — 0 mismatches, and confirmed `climbStairs(45) = 1836311903` doesn't overflow a 32-bit `int` (max value stays under `INT_MAX`). Clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python O(1)-space version independently verified against the same array-based reference for `n = 0..45` — 0 mismatches. Java is the same O(1)-space logic translated directly (not independently compiled in this environment — no JDK available).

## Bug log

- Your bottom-up DP (full array): correct on the first attempt — no bugs found.
