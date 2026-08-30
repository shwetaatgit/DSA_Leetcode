# Merge Sorted Array (LeetCode Easy)

## Problem

`nums1` and `nums2` are sorted non-decreasing. `nums1` has length `m + n` — the first `m` slots hold real data, the last `n` are `0` placeholders reserved for `nums2`'s elements. `nums2` has length `n`. Merge `nums2` into `nums1` in place so the result is sorted.

```
nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
-> [1,2,2,3,5,6]
```

## Approach

Fill from the **back**, largest values first — three pointers: `i1` at the last slot of `nums1` (the write position), `i2` at the last real element of `nums1`, `i3` at the last element of `nums2`. Each step, place the bigger of `nums1[i2]`/`nums2[i3]` at `nums1[i1]` and step that pointer back. If `i2` runs out first, the rest of `nums2` copies straight in; if `i3` runs out first, `nums1`'s remaining elements are already sitting in their correct final spots (since `i1 == i2` at that point) — no separate pass needed either way, one loop with two guard branches handles both.

Time: O(m+n) · Space: O(1), no sort needed.

## Solution

Refined form — loop on `i3` alone (stop once `nums2` is exhausted; if `nums1`'s own elements run out first, they're already sitting in their correct final spots, so nothing left to do). The `i2 >= 0 &&` short-circuit replaces the need for a separate `i2 < 0` branch.

### C++
```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int i1 = m + n - 1, i2 = m - 1, i3 = n - 1;
        while (i3 >= 0) {
            if (i2 >= 0 && nums1[i2] > nums2[i3]) {
                nums1[i1] = nums1[i2]; i2--;
            } else {
                nums1[i1] = nums2[i3]; i3--;
            }
            i1--;
        }
    }
};
```

### Python
```python
class Solution:
    def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:
        i1, i2, i3 = m + n - 1, m - 1, n - 1
        while i3 >= 0:
            if i2 >= 0 and nums1[i2] > nums2[i3]:
                nums1[i1] = nums1[i2]; i2 -= 1
            else:
                nums1[i1] = nums2[i3]; i3 -= 1
            i1 -= 1
```

### Java
```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i1 = m + n - 1, i2 = m - 1, i3 = n - 1;
        while (i3 >= 0) {
            if (i2 >= 0 && nums1[i2] > nums2[i3]) {
                nums1[i1] = nums1[i2]; i2--;
            } else {
                nums1[i1] = nums2[i3]; i3--;
            }
            i1--;
        }
    }
}
```

All three verified against 5 cases (classic merge, nums1-values-all-larger, nums2-values-all-larger, `m=0`, `n=0`) — C++ additionally clean under AddressSanitizer.

## Bug log (real mistakes made getting here)

- Off-by-one indexing into `nums2` when copying without adjusting for the `m` offset.
- `sort()`-after-copy works but is O((m+n) log(m+n)), not optimal.
- **While-loop condition inverted three times in a row**: wrote the *stopping* condition (`i1 < 0`, `i2 < 0`) where C++ needs the *continuing* condition (`i1 >= 0`). `while(cond)` always means "keep going while true," never "stop when true."
- `continue` inside a branch skipped the `i1--` at the loop's end, which would have caused an effectively infinite loop.
- Missing guard for `i2 < 0` and `i3 < 0` individually caused a real heap-buffer-overflow (confirmed via AddressSanitizer) — C++ doesn't throw on out-of-bounds `vector` access via `[]`, it's undefined behavior that can silently "look correct" by luck.
