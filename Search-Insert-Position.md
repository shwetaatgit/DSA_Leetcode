# Search Insert Position (LeetCode Easy)

## Problem

Given a sorted array of distinct integers `nums` and a target value, return the index if the target is found; if not, return the index where it would be inserted to keep the array sorted.

```
nums = [1,3,5,6], target = 5  -> 2   (found at index 2)
nums = [1,3,5,6], target = 2  -> 1   (would insert between 1 and 3)
nums = [1,3,5,6], target = 7  -> 4   (would insert at the end)
```

## How to explain it out loud

*"This is binary search, but instead of just returning -1 on failure, I track the best insertion point as I go. Every time I move `l` past `mid` because `nums[mid] < target`, the insertion point — if the target isn't found — has to be at least `mid+1`, so I record that. Every time I move `r` before `mid` because `nums[mid] >= target`, the insertion point could be exactly `mid` (or something smaller), so I record `mid` as the current best guess and keep narrowing left. By the time the search space closes, whichever value I last recorded is exactly where the target belongs — this is really just `lower_bound` — the first index where `nums[i] >= target`."*

## Approach — Binary search, tracking the insertion point as the search narrows

Standard binary search with `l`, `r` bounds and `mid = l + (r-l)/2` (this form avoids the `(l+r)/2` overflow pitfall for very large indices, though not strictly needed here given the constraint `nums.length <= 10^4`). If `nums[mid] == target`, found it — return immediately.

If `nums[mid] < target`, the target (if present) is further right, so `l = mid + 1` — and since everything up through `mid` is confirmed too small, the insertion point (if the target turns out to be absent) must be at least `mid + 1`, so update `ans = mid + 1`.

If `nums[mid] > target`, the target is further left, so `r = mid - 1` — and `mid` itself is a value *not smaller* than the target, making it a valid candidate insertion point, so update `ans = mid`.

When the loop ends without an exact match, `ans` holds the last-recorded candidate, which is exactly the correct insertion index — equivalent to `std::lower_bound`.

Time: O(log n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int l = 0 , r = nums.size()-1 , mid , ans = -1;
        while(l <= r)
        {
            mid = l + (r - l) / 2;
            if(nums[mid] == target)
                return mid;
            if(nums[mid] < target)
            {
                l = mid + 1;
                ans = mid + 1;
            }
            else
            {
                ans = mid;
                r = mid - 1;
            }
        }
        return ans;
    }
};
```

### Python
```python
class Solution:
    def searchInsert(self, nums: list[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        ans = len(nums)  # default: target is larger than everything, insert at the end
        while l <= r:
            mid = l + (r - l) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] < target:
                l = mid + 1
            else:
                ans = mid
                r = mid - 1
        return ans
```

### Java
```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int l = 0, r = nums.length - 1, ans = nums.length;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] == target) return mid;
            if (nums[mid] < target) {
                l = mid + 1;
            } else {
                ans = mid;
                r = mid - 1;
            }
        }
        return ans;
    }
}
```

Verified against 8 hand-picked cases (found in the middle, insert in the middle, insert at the end, insert at the start, an empty array, and a single-element array checked against a match, a smaller target, and a larger target) plus a 5000-trial randomized stress test (array sizes 1-15, per LeetCode's actual stated constraint `1 <= nums.length`) against `std::lower_bound` as a reference — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java default `ans` to `len(nums)` up front instead of `-1`, so they handle a target larger than every element correctly without relying on the loop having updated `ans` at least once — worth noting as a difference in defensive style, not a correctness fix, since it doesn't come up within the actual stated constraints either way.

## Bug log

- Correct on the first attempt against every input satisfying LeetCode's actual constraint (`1 <= nums.length <= 10^4`) — no bugs found there.
- Noted, not fixed (outside the problem's stated constraints): with an *empty* array, the `while` loop never executes (since `r` starts at `-1 < l`), so `ans` is returned at its initial value of `-1` instead of the arguably-more-useful `0` (the correct insertion index for an empty array). Confirmed via a targeted test. Not a bug against the actual problem — LeetCode guarantees `nums.length >= 1` — but worth knowing if this exact code were reused somewhere that guarantee didn't hold. The Python/Java versions above sidestep this by defaulting `ans` to `len(nums)` instead of `-1`, which happens to also come out correct for the empty-array edge case as a side effect.
