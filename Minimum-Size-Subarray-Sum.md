# Minimum Size Subarray Sum (LeetCode Medium)

## Problem

Given an array of positive integers `nums` and an integer `target`, return the minimal length of a contiguous subarray whose sum is `>= target`. Return `0` if no such subarray exists.

```
target = 7, nums = [2,3,1,2,4,3] -> 2   (subarray [4,3] sums to 7)
```

## How to explain it out loud

*"Since every number is positive, growing the window only ever increases the sum and shrinking it only ever decreases the sum — that monotonic relationship is exactly what makes a sliding window valid here. Expand the right edge, adding each new element to a running sum. The moment the sum reaches the target, don't just shrink once — keep shrinking from the left for as long as the sum stays at or above target, recording the window length at every step along the way, since a smaller window might still be valid. Once shrinking drops the sum below target, go back to expanding the right edge. Track a flag for whether the target was ever reached at all, since if it never was, the answer is 0, not whatever an uninitialized minimum happens to hold."*

## Approach

Two pointers, `l` and `r`, both starting at `0`, plus a running `sum`. Expand `r` one step at a time, adding `nums[r]` to `sum`. Whenever `sum >= target`, enter a `while` loop (not just an `if`) that keeps shrinking from the left — recording `r-l+1` as a candidate minimum length, then subtracting `nums[l]` from `sum` and advancing `l` — for as long as the window is still valid. This is the key detail: shrinking must continue until the window is no longer valid, not stop after a single step, otherwise a smaller valid window further along could be missed.

Track whether the target was ever actually reached with a boolean flag. If it never was, return `0` (LeetCode's specified answer for "no valid subarray"); otherwise return the tracked minimum length.

Time: O(n) — each element is added to the window once and removed at most once, so both pointers together do O(n) total work · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int minLength = INT_MAX;
        int l = 0, r = 0;
        int sum = 0;
        bool target_achieved = false;

        while (r < nums.size() && l <= r) {
            sum += nums[r];
            while (sum >= target) {
                target_achieved = true;
                minLength = min(minLength, r - l + 1);
                sum -= nums[l];
                l++;
            }
            r++;
        }
        if (!target_achieved) return 0;
        return minLength;
    }
};
```

### Python
```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        min_length = float('inf')
        l = 0
        total = 0
        target_achieved = False
        for r in range(len(nums)):
            total += nums[r]
            while total >= target:
                target_achieved = True
                min_length = min(min_length, r - l + 1)
                total -= nums[l]
                l += 1
        return min_length if target_achieved else 0
```

### Java
```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int minLength = Integer.MAX_VALUE;
        int l = 0;
        int sum = 0;
        boolean targetAchieved = false;

        for (int r = 0; r < nums.length; r++) {
            sum += nums[r];
            while (sum >= target) {
                targetAchieved = true;
                minLength = Math.min(minLength, r - l + 1);
                sum -= nums[l];
                l++;
            }
        }
        return targetAchieved ? minLength : 0;
    }
}
```

Verified against 6 hand-picked cases (the classic example, a case with an exact-match single element, no valid subarray at all, another no-valid-subarray case, a wider array, a single-element array) plus a 2000-trial randomized stress test against a brute-force O(n²) reference — 0 mismatches, within the problem's actual constraints (`target >= 1`, all `nums[i] >= 1`). C++ clean under AddressSanitizer within those constraints. Python matches on all 6 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- First attempt shrank the window with `if (sum>=target)` instead of `while (sum>=target)` — only ever contracted the window by one element before going back to expanding `r`, even when the window could have been shrunk further while remaining valid. Confirmed as a real bug: returned `4` instead of the correct `2` on the classic example, because the true minimal window `[4,3]` was never fully converged on. Fixed by changing to a `while` loop, so shrinking continues until the window actually becomes invalid.
- Second attempt (after the `while` fix) initialized `minLength` to `nums.size()` instead of a proper sentinel like `INT_MAX`, intending it as the "no valid subarray" fallback. This broke the actual "no valid subarray" case: confirmed `target=100`, `nums=[1,1,1,1]` returned `4` (the array's length) instead of the correct `0`, since `minLength` was never touched by the inner loop and just kept its initial value. Fixed with an explicit `target_achieved` boolean flag, set `true` only when the inner loop actually fires at least once, checked before returning.
- Separately (not a bug against the problem's actual constraints, but worth knowing): with `target <= 0` — outside LeetCode's stated `1 <= target <= 10^9` range — `sum >= target` becomes trivially true forever once the window empties, since all array elements are positive and `sum` never goes negative. This causes the inner `while` to keep shrinking `l` past `r` and out of the array's bounds entirely, confirmed as a real out-of-bounds crash (`target=0`, `nums=[1]`) under AddressSanitizer. Not a bug that can be triggered by any valid LeetCode input for this problem, but a reminder that the code implicitly relies on the `target >= 1` guarantee without checking it — worth knowing if this logic were ever reused somewhere that guarantee didn't hold.
