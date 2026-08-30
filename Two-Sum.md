j# Two Sum (LeetCode Easy)

## Problem

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one valid answer exists, and you can't reuse the same element twice. Return the indices in any order.

**Example**
```
nums = [2, 7, 11, 15], target = 9
-> [0, 1]        (nums[0] + nums[1] = 2 + 7 = 9)
```

## Approaches

**1. Brute force** — check every pair `(i, j)` with a nested loop until one sums to `target`. Simple, but O(n²).

**2. Hash map, one pass (optimal)** — walk the array once; before inserting the current number, check whether its *complement* (`target - num`) has already been seen. Checking before inserting means an element can never pair with itself, so no extra index check is needed.

Time: O(n) · Space: O(n)

## Solution

### Python
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}  # value -> index
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i
        return []
```

### C++
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> seen; // value -> index
        for (int i = 0; i < (int)nums.size(); i++) {
            int complement = target - nums[i];
            if (seen.find(complement) != seen.end()) {
                return {seen[complement], i};
            }
            seen[nums[i]] = i;
        }
        return {};
    }
};
```

### Java
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>(); // value -> index
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (seen.containsKey(complement)) {
                return new int[] { seen.get(complement), i };
            }
            seen.put(nums[i], i);
        }
        return new int[0];
    }
}
```

All three verified against: `[2,7,11,15]/9 -> [0,1]`, `[3,2,4]/6 -> [1,2]`, `[3,5]/6 -> []` (no valid pair), `[3,3]/6 -> [0,1]` (duplicates).
