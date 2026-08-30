# Remove Element (LeetCode Easy)

## Problem

Given `nums` and a value `val`, remove every occurrence of `val` in-place (order doesn't matter). Return `k`, the count of elements left that aren't `val` — `nums[0..k-1]` must actually contain those elements; anything past index `k-1` is irrelevant.

```
nums = [3,2,2,3], val = 3
-> k = 2, nums = [2,2,_,_]
```

## Approach

Two pointers, one pass: `i` scans every element, `k` tracks the next write position for a kept element. Whenever `nums[i] != val`, set `nums[k]` to `nums[i]` to store the non-target element at the current `k`th position, then increment `k`. Elements equal to `val` are simply never written — they get silently overwritten by later kept elements, or left as leftover garbage past index `k-1`, which the problem doesn't care about.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int k = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != val) {
                nums[k] = nums[i];
                k++;
            }
        }
        return k;
    }
};
```

### Python
```python
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        k = 0
        for i in range(len(nums)):
            if nums[i] != val:
                nums[k] = nums[i]
                k += 1
        return k
```

### Java
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int k = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != val) {
                nums[k] = nums[i];
                k++;
            }
        }
        return k;
    }
}
```

Verified against 5 cases (mixed, mixed with duplicates, no matches, all matches, empty) — C++ additionally clean under AddressSanitizer.

## Bug log

- First attempt counted occurrences *of* `val` instead of elements *not equal to* `val` — backwards metric, caught with a case where the answer differs from a naive count.
- `i < nums.size()-1` off-by-one, silently skipped the last element — `.size()` returns an unsigned type, so `size()-1` on an empty vector underflows to a huge number rather than going negative. Avoid `size()-1` in loop bounds; use `size()` directly, or check emptiness first if subtraction is unavoidable.
- First correct-count version still never touched `nums` itself — LeetCode's checker inspects the array contents, not just the return value.
