# Container With Most Water (LeetCode Medium)

## Problem

Given `n` non-negative integers `height[i]`, each representing a vertical line at position `i`, find two lines that together with the x-axis form a container holding the most water.

```
height = [1,8,6,2,5,4,8,3,7] -> 49   (lines at index 1 and 8: min(8,7) * (8-1) = 7*7 = 49)
```

## How to explain it out loud

*"Start with the two pointers at both ends — that's the widest possible container. The area is min(height[l], height[r]) * (r-l). Now think about which pointer to move: whichever side is shorter is the bottleneck, since the container's height is capped by the shorter wall. If I move the taller side inward, the width shrinks and the height can only stay the same or get worse (since the shorter wall is still the cap) — that can never improve the area. But if I move the shorter side inward, the width still shrinks, but there's a chance the new wall is taller, which could raise the height enough to beat the previous area. So moving the shorter pointer is the only move that has any chance of improving things — moving the taller one is provably never better. Track the max area seen at each step as the pointers close in."*

## Approach — Brute force

Try every pair of lines `(i, j)` with `i < j`, compute the area `(j-i) * min(height[i], height[j])` for each, and track the maximum. Straightforward, but re-examines every pair regardless of what's already been ruled out.

Time: O(n²) · Space: O(1)

## Approach — Two pointers (optimal)

Two pointers, `l=0` and `r=n-1`. At each step, compute the area `(r-l) * min(height[l], height[r])` and update a running max. Then move whichever pointer points at the **shorter** line, inward.

Why this greedy move is safe: the area is bottlenecked by the shorter of the two walls. If you move the taller pointer inward instead, the width strictly decreases, and the height is still capped by the (unchanged) shorter wall — so the new area can never exceed the current one. Moving the shorter pointer, on the other hand, decreases width too, but gives the height a chance to improve (the new wall might be taller than the old shorter one) — it's the only move that could possibly find something better. So every container that could beat the current best gets a chance to be considered as the pointers close in, and nothing is skipped.

Time: O(n) · Space: O(1)

## Solution

### C++ (Brute force)
```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int n = height.size();
        int best = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i+1; j < n; j++) {
                best = max(best, (j-i) * min(height[i], height[j]));
            }
        }
        return best;
    }
};
```

### C++ (Two pointers — optimal)
```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int n = height.size();
        int l = 0, r = n - 1;
        int maxWater = INT_MIN;

        while (l < r) {
            maxWater = max(maxWater, (r-l) * min(height[l], height[r]));
            if (height[l] < height[r]) l++;
            else r--;
        }

        return maxWater;
    }
};
```

### Python (Brute force)
```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        n = len(height)
        best = 0
        for i in range(n):
            for j in range(i+1, n):
                best = max(best, (j-i) * min(height[i], height[j]))
        return best
```

### Python (Two pointers — optimal)
```python
class Solution:
    def maxArea(self, height: List[int]) -> int:
        l, r = 0, len(height) - 1
        best = 0
        while l < r:
            best = max(best, (r-l) * min(height[l], height[r]))
            if height[l] < height[r]:
                l += 1
            else:
                r -= 1
        return best
```

### Java
```java
class Solution {
    public int maxArea(int[] height) {
        int l = 0, r = height.length - 1;
        int maxWater = 0;
        while (l < r) {
            maxWater = Math.max(maxWater, (r-l) * Math.min(height[l], height[r]));
            if (height[l] < height[r]) l++;
            else r--;
        }
        return maxWater;
    }
}
```

Verified against 4 hand-picked cases (classic example, two equal minimal lines, a symmetric peak-valley shape, a small case with a repeated value) — both brute force and two-pointer versions match on all 4 in both C++ and Python, plus the two-pointer version additionally cross-checked against the brute force as ground truth on a 1000-trial randomized stress test (0 mismatches). C++ (both versions) clean under AddressSanitizer. Java is the two-pointer logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the first attempt, including the greedy "move the shorter pointer" insight, verified against 1000 random trials.
