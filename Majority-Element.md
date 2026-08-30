# Majority Element (LeetCode Easy)

## Problem

Given an array `nums` of size `n`, return the element that appears more than `⌊n/2⌋` times. A majority element is guaranteed to exist.

```
nums = [2,2,1,1,1,2,2]
-> 2
```

## Approaches

**1. Hash map** — count occurrences of each value, return the one that exceeds `⌊n/2⌋`. O(n) time, O(n) space.

**2. Boyer-Moore voting (optimal)** — a known trick specific to this problem: keep a `candidate` and a `count`. Each element matching the candidate increments count, each mismatch decrements it; when count hits zero, switch candidates. Because the true majority element outnumbers everything else combined, it's guaranteed to survive as the final candidate.

Time: O(n) · Space: O(1)

## Solution (hash map)

### C++
```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int majorityCount = nums.size() / 2;
        int result = 0;
        unordered_map<int,int> m;
        for (int i = 0; i < nums.size(); i++) {
            if (majorityCount == 0) result = nums[i];
            if (m.find(nums[i]) != m.end()) {
                m[nums[i]]++;
                if (m[nums[i]] > majorityCount) result = nums[i];
            } else {
                m[nums[i]] = 1;
            }
        }
        return result;
    }
};
```

## Solution (Boyer-Moore, optimal)

### C++
```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int count = 0, candidate = 0;
        for (int num : nums) {
            if (count == 0) candidate = num;
            count += (num == candidate) ? 1 : -1;
        }
        return candidate;
    }
};
```

### Python
```python
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        count, candidate = 0, 0
        for num in nums:
            if count == 0:
                candidate = num
            count += 1 if num == candidate else -1
        return candidate
```

### Java
```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0, candidate = 0;
        for (int num : nums) {
            if (count == 0) candidate = num;
            count += (num == candidate) ? 1 : -1;
        }
        return candidate;
    }
}
```

Both approaches verified against 5 cases (classic, single element, pair, interleaved, tail-heavy) — Boyer-Moore additionally clean under AddressSanitizer.

## Bug log

- `m[nums[i]]++` used inside the comparison — post-increment evaluates to the count *before* incrementing, so the check compared the old count. Failed exactly when the majority element had the bare-minimum majority count (the common case, not an edge case).
- Returned `m[nums[i]]` (the count) instead of `nums[i]` (the element) — wrong return semantics entirely.
- `floor` as a variable name shadows `std::floor` from `<cmath>` — renamed to `majorityCount`. Same class of mistake as naming a Python dict `dict`.
- First-occurrence branch never checked whether a single occurrence alone already exceeds `majorityCount` — only matters when `majorityCount == 0` (i.e. `n == 1`), but that's a real, valid input. Fixed by checking `majorityCount == 0` explicitly rather than only checking on repeat occurrences.
