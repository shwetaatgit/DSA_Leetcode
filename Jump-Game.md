# Jump Game (LeetCode Medium)

## Problem

`nums[i]` is the max jump length from index `i`. Starting at index `0`, determine if you can reach the last index.

```
nums = [2,3,1,1,4] -> true
nums = [3,2,1,0,4] -> false
```

## How to explain it out loud

*"Two ways to approach this. Brute force / DP: work backward from the end, marking each index as 'good' if any jump from it lands on an already-good index. Greedy: work forward, tracking the farthest index reachable so far — if you ever reach a position beyond that farthest point, you're stuck; otherwise keep extending it. The greedy version is O(n) time O(1) space; the DP version is O(n²) time O(n) space, but it's the natural starting point before spotting the greedy optimization."*

## Approach — DP (brute force)

At any index `i`, `nums[i]` is a **maximum**, not a fixed distance — you can jump anywhere from `1` step up to `nums[i]` steps. So for index `i` to be "good" (able to reach the end), it's enough for **any one** of those jump lengths to land on an already-good index — not just the longest jump.

Working backward from the last index (trivially good):
1. For each index `i` from `n-2` down to `0`, try every jump length `j` from `1` to `nums[i]`.
2. Check `reached[i+j]` — if it's already `true` for *any* `j` in that range, then index `i` can reach the end too (via that specific jump). Mark `reached[i] = true` and stop checking further `j` values for this `i`.
3. The answer is `reached[0]`.

Checking only the longest jump isn't enough — a shorter jump can land on a working stepping stone while the longest one overshoots into a dead end. Example: `nums = [4,0,0,3,0,0,0]` — from index 0, the max jump (4) lands on a dead index, but jump 3 lands on index 3, which itself jumps 3 to reach the end. Missing that shorter option would wrongly mark index 0 as bad.

Time: O(n²) worst case · Space: O(n)

## Approach — Greedy (optimal)

Track `maxIndex`, the farthest index reachable so far, starting at `0`. Walk forward: if the current index `i` is already beyond `maxIndex`, you could never have stood there — return `false`. Otherwise extend `maxIndex = max(maxIndex, i + nums[i])`. This has to be a running maximum, not an overwrite — an earlier index's jump can reach farther than a later one's, and overwriting would forget that.

Time: O(n) · Space: O(1)

## Solution

### C++ (Greedy)
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maxIndex = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (i > maxIndex) return false;
            maxIndex = max(maxIndex, i + nums[i]);
            if (maxIndex > nums.size()) return true;
        }
        return true;
    }
};
```

### C++ (DP)
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        vector<bool> reached(nums.size(), false);
        reached[nums.size()-1] = true;
        for (int i = nums.size()-2; i >= 0; i--) {
            for (int j = 1; j <= nums[i]; j++) {
                if (reached[i+j] == true) {
                    reached[i] = true;
                    break;
                }
            }
        }
        return reached[0];
    }
};
```

### Python (Greedy)
```python
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        max_index = 0
        for i in range(len(nums)):
            if i > max_index:
                return False
            max_index = max(max_index, i + nums[i])
            if max_index >= len(nums) - 1:
                return True
        return True
```

### Java (Greedy)
```java
class Solution {
    public boolean canJump(int[] nums) {
        int maxIndex = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > maxIndex) return false;
            maxIndex = Math.max(maxIndex, i + nums[i]);
            if (maxIndex >= nums.length - 1) return true;
        }
        return true;
    }
}
```

Both approaches verified against 7 cases (classic true, classic false, shorter-jump-beats-max-jump, single element, immediate dead end, exact-landing on last index, alternating zeros) across all languages — C++ additionally clean under AddressSanitizer.

## Bug log

- Greedy version's early-return check `maxIndex > nums.size()` is looser than necessary (should be `>= nums.size()-1`, the actual last index) — doesn't cause wrong answers, since the loop still falls through to the correct `return true` when it doesn't fire early, but it's an unnecessary missed optimization. Tightened in the Python/Java versions.
- DP version correct on first attempt — the key insight (check *every* jump length, not just the max) was internalized before writing any code, from working through the `[4,0,0,3,0,0,0]` counterexample first.
