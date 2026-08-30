# Jump Game II (LeetCode Medium)

## Problem

Same array, `nums[i]` = max jump from index `i`. Return the **minimum number of jumps** to reach the last index (always guaranteed reachable).

```
nums = [2,3,1,1,4] -> 2   (index 0 → 1 → 4)
```

## How to explain it out loud

*"Forward DP first: minSteps[i] holds the fewest jumps to reach index i. Starting from index 0, for every index I've already resolved, I try every jump length up to nums[i] and fill in minSteps for anything not yet reached — first time you reach a cell is always the minimum, since you're filling in increasing distance order. That's O(n²) worst case. The greedy optimization tracks the boundary of the current jump and the farthest reachable within the next one: every time I reach the edge of what the current jump count can cover, that's forced — I have to take another jump, and I already know the best next boundary because I've been tracking it. O(n), one pass."*

## Approach — Forward DP (brute force)

`minSteps[i]` = fewest jumps to reach index `i`, all initialized to `0` (which also happens to be the correct value for index `0` itself — no ambiguity, since nothing ever writes back to index `0`). Walk `ind` from `0` upward; for each `ind`, try every jump length `1..nums[ind]`, and for any landing index not yet reached, mark it with `minSteps[ind]+1`. Stop as soon as the last index is reached.

Time: O(n²) worst case · Space: O(n)

## Approach — Greedy (optimal)

Track three values: `jumps` (count so far), `currentEnd` (the farthest index reachable with the jumps taken so far), and `farthest` (the farthest index reachable with one more jump from anywhere visited so far). Walk forward, continuously updating `farthest`. Whenever `i` reaches `currentEnd` — meaning you've exhausted everywhere the current jump count could reach — that forces another jump: increment `jumps` and extend `currentEnd` to `farthest`.

Time: O(n) · Space: O(1)

## Solution

### C++ (Forward DP)
```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        if (n==0 || n==1) return 0;
        vector<int> minSteps(n, 0);
        int ind = 0;
        while (ind < n && minSteps[n-1]==0) {
            for (int i = 1; i <= nums[ind]; i++) {
                if (ind+i < n && minSteps[ind+i]==0) minSteps[ind+i] = minSteps[ind]+1;
            }
            ind++;
        }
        return minSteps[n-1];
    }
};
```

Cleaner variant of the same idea — same forward DP, but the bound is baked directly into the inner loop via `min(n-1, i+nums[i])`, so there's no way to ever generate an out-of-range index at all (nothing to short-circuit correctly or incorrectly):

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n, INT_MAX);
        dp[0] = 0;
        for (int i = 0; i < n; ++i) {
            for (int j = i+1; j <= min(n-1, i+nums[i]); ++j) {
                dp[j] = min(dp[j], dp[i]+1);
            }
        }
        return dp[n-1];
    }
};
```
`dp[i]` = minimum jumps to reach index `i`, `INT_MAX` meaning "unreached." Safe to trust `dp[i]` when using it to update others: jumps only go forward, `i` is processed in increasing order, so every index that could possibly improve `dp[i]` has already run by the time `i`'s turn comes — same guarantee a topological-order DAG relaxation gets.

### C++ (Greedy)
```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        int jumps = 0, currentEnd = 0, farthest = 0;
        for (int i = 0; i < n - 1; i++) {
            farthest = max(farthest, i + nums[i]);
            if (i == currentEnd) {
                jumps++;
                currentEnd = farthest;
            }
        }
        return jumps;
    }
};
```

### Python (Forward DP)
```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        n = len(nums)
        if n <= 1:
            return 0
        min_steps = [0] * n
        ind = 0
        while ind < n and min_steps[n-1] == 0:
            for i in range(1, nums[ind]+1):
                if ind+i < n and min_steps[ind+i] == 0:
                    min_steps[ind+i] = min_steps[ind] + 1
            ind += 1
        return min_steps[n-1]
```

### Java (Forward DP)
```java
class Solution {
    public int jump(int[] nums) {
        int n = nums.length;
        if (n <= 1) return 0;
        int[] minSteps = new int[n];
        int ind = 0;
        while (ind < n && minSteps[n-1] == 0) {
            for (int i = 1; i <= nums[ind]; i++) {
                if (ind+i < n && minSteps[ind+i] == 0) minSteps[ind+i] = minSteps[ind] + 1;
            }
            ind++;
        }
        return minSteps[n-1];
    }
}
```

Both approaches verified against 8 cases (classic, a zero mid-array, all-ones, one huge first jump, single element, two elements, small array, jump far past the end from a large first value) — C++ additionally clean under AddressSanitizer.

## Bug log

- No bounds check on `ind+i` at all initially — `nums[i]` can exceed the remaining array length (e.g. `nums[0]=5` on a 5-element array), and nothing stops the jump length from reaching past the last index. Heap-buffer-overflow, confirmed via AddressSanitizer.
- First fix attempt (`ind+i < n-1`) overcorrected — that excludes index `n-1` **itself**, the actual target, from ever being marked reachable. `minSteps[n-1]` then never leaves `0`, the loop's own exit condition never fires, and it eventually reaches the same out-of-bounds crash again, now even on the basic example.
- Second fix (`minSteps[ind+i]==0 && ind+i < n`) — correct destination, but wrong **order**. `&&` short-circuits left to right, so the array access on the left (`minSteps[ind+i]`) still executes *before* the bounds check on the right gets a chance to block it. Crashed on `[5,1,1,1,1]` for the same reason. Fixed by swapping the order — `ind+i < n && minSteps[ind+i]==0` — so the bounds check runs first and the array access is skipped entirely when out of range.
