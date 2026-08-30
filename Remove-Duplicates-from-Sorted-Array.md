# Remove Duplicates from Sorted Array (LeetCode Easy)

## Problem

Given a **sorted** array `nums`, remove duplicates in-place so each unique element appears once. Return `k`, the count of unique elements — `nums[0..k-1]` must hold them in order.

```
nums = [0,0,1,1,1,2,2,3,3,4]
-> k = 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

## Approaches

**1. Hash set (works on any array, sorted or not)** — track every value seen in a `set`, write to `nums[k]` only on first sight. Correct, but doesn't exploit sortedness: O(n log n) time (`set` operations are O(log n)), O(n) extra space.

```cpp
int removeDuplicates(vector<int>& nums) {
    int k = 0;
    set<int> s;
    for (int i = 0; i < nums.size(); i++) {
        if (s.find(nums[i]) == s.end()) {
            nums[k] = nums[i];
            s.insert(nums[i]);
            k++;
        }
    }
    return k;
}
```

**2. Adjacent comparison (optimal, sorted-only)** — since the array is sorted, duplicates are always adjacent. Compare each element only to its immediate predecessor; no set needed at all.

Time: O(n) · Space: O(1)

## Solution (optimal)

### C++
```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        int j = 1;
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] != nums[i-1]) {
                nums[j] = nums[i];
                j += 1;
            }
        }
        return j;
    }
};
```

### Python
```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        if len(nums) == 0:
            return 0
        j = 1
        for i in range(1, len(nums)):
            if nums[i] != nums[i-1]:
                nums[j] = nums[i]
                j += 1
        return j
```

### Java
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;
        int j = 1;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[i-1]) {
                nums[j] = nums[i];
                j += 1;
            }
        }
        return j;
    }
}
```

All three verified against 5 cases (mixed duplicates, pair, single, no duplicates, all-same) — C++ additionally clean under AddressSanitizer.
