# Trapping Rain Water (LeetCode Hard)

## Problem

Given `n` non-negative integers representing an elevation map where each bar has width 1, compute how much water it can trap after raining.

```
height = [0,1,0,2,1,0,1,3,2,1,2,1] -> 6
```

## How to explain it out loud

*"Per column, the water sitting above bar i is bounded by the shorter of the tallest wall to its left and the tallest wall to its right — water leaks out over whichever side is lower. Brute force: precompute the max height to the left of every index and the max height to the right of every index, then for each index the trapped water is min(leftMax[i], rightMax[i]) - height[i], floored at zero. That's O(n) time but O(n) extra space for the two arrays. The optimized version collapses this to two pointers from both ends, tracking just the running max seen so far from each side — no full arrays needed. Move whichever pointer currently points at the shorter bar; you can commit to using just that side's max because the other side is guaranteed to have something at least that tall already, so it can't be the limiting wall."*

## Approach — Prefix/suffix max arrays (brute force)

Build `leftMax[i]` = the tallest bar anywhere in `height[0..i]`, and `rightMax[i]` = the tallest bar anywhere in `height[i..n-1]`, via one forward pass and one backward pass. For each index `i`, the water trapped there is `min(leftMax[i], rightMax[i]) - height[i]` (this is always `>= 0`, since `leftMax[i]` and `rightMax[i]` both include `height[i]` itself). Sum over all `i`.

Time: O(n) · Space: O(n) for the two arrays

## Approach — Two pointers (optimal)

Same idea, but collapse both arrays into two running variables. Start `left=0`, `right=n-1`, `leftMax` and `rightMax` both at negative infinity (or the smallest possible value). While `left < right`, compare `height[left]` and `height[right]`:

- If `height[left] <= height[right]`: the left side is the shorter (or equal) wall. That means whatever the *true* max on the right side turns out to be, it's already guaranteed to be at least `height[right] >= height[left]` — so the right side can't possibly be the limiting wall for position `left`. It's safe to resolve `left` using only `leftMax`: if `height[left] < leftMax`, add `leftMax - height[left]` to the water total; otherwise `height[left]` itself is a new high, so update `leftMax = height[left]`. Then advance `left++`.
- Otherwise (symmetric): resolve `right` using only `rightMax`, then `right--`.

Because you only ever move the pointer on the side proven *not* to be the limiting wall, every index gets resolved correctly using just one running max — no need to ever know the other side's exact max.

Time: O(n), each index visited once · Space: O(1) extra

## Solution

### C++ (Prefix/suffix max arrays — brute force)
```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        if (n == 0) return 0;   // guard: without this, prefix[0]=height[0] and
                                 // suffix[n-1]=height[n-1] both index an empty vector
        vector<int> prefix(n);
        vector<int> suffix(n);

        prefix[0] = height[0];
        suffix[n-1] = height[n-1];

        for (int i = 1; i < n; i++) {
            prefix[i] = max(prefix[i-1], height[i]);
        }
        for (int i = n-2; i >= 0; i--) {
            suffix[i] = max(suffix[i+1], height[i]);
        }

        int water = 0;
        for (int i = 0; i < n; i++) {
            water += min(prefix[i], suffix[i]) - height[i];
        }

        return water;
    }
};
```

### C++ (Two pointers — optimal)
```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        int l = 0, r = n-1, water = 0;
        int lmax = INT_MIN, rmax = INT_MIN;

        while (l < r) {
            if (height[l] <= height[r]) {
                if (height[l] < lmax) {
                    water += lmax - height[l];
                } else {
                    lmax = height[l];
                }
                l++;
            } else {
                if (height[r] < rmax) {
                    water += rmax - height[r];
                } else {
                    rmax = height[r];
                }
                r--;
            }
        }
        return water;
    }
};
```

### Python
```python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        l, r = 0, n - 1
        water = 0
        lmax, rmax = float('-inf'), float('-inf')
        while l < r:
            if height[l] <= height[r]:
                if height[l] < lmax:
                    water += lmax - height[l]
                else:
                    lmax = height[l]
                l += 1
            else:
                if height[r] < rmax:
                    water += rmax - height[r]
                else:
                    rmax = height[r]
                r -= 1
        return water
```

### Java
```java
class Solution {
    public int trap(int[] height) {
        int n = height.length;
        int l = 0, r = n-1, water = 0;
        int lmax = Integer.MIN_VALUE, rmax = Integer.MIN_VALUE;

        while (l < r) {
            if (height[l] <= height[r]) {
                if (height[l] < lmax) {
                    water += lmax - height[l];
                } else {
                    lmax = height[l];
                }
                l++;
            } else {
                if (height[r] < rmax) {
                    water += rmax - height[r];
                } else {
                    rmax = height[r];
                }
                r--;
            }
        }
        return water;
    }
}
```

Verified against 9 hand-picked cases (classic example, a second standard case, empty input, single bar, two equal bars, strictly increasing/no trapping, strictly decreasing/no trapping, all-flat, and a symmetric basin) plus a 1000-trial randomized stress test against an independent prefix/suffix-max reference implementation — 0 mismatches, both C++ versions additionally clean under AddressSanitizer. Python (two-pointer) matches on all 9 hand-picked cases. Java is the two-pointer logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- Two-pointer version: correct on the first attempt, including correctly reasoning through why `INT_MIN` initialization for `lmax`/`rmax` is safe (the very first comparison for each side always falls into the "new high" branch rather than the subtraction branch, since no real height is less than `INT_MIN`, so there's no underflow risk) and why `<=` vs `<` at the tie-breaking comparison (`height[l] <= height[r]`) doesn't affect correctness either way.
- Prefix/suffix version: correct on all normal inputs, but crashed on empty input (`height = []`) — confirmed as a segfault under AddressSanitizer. `prefix[0] = height[0]` and `suffix[n-1] = height[n-1]` both unconditionally index into `height` before any size check, so when `n == 0` these read/write invalid memory. Fixed by adding `if (n == 0) return 0;` as the very first line. The two-pointer version never had this problem, since its `while (l < r)` loop condition is naturally false when `n == 0` (`l=0, r=-1`), so it never touches `height` at all in that case — a good example of how a loop-condition-driven approach can sidestep an edge case that an array-preallocation approach has to guard explicitly.
