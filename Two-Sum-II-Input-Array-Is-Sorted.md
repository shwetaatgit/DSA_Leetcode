# Two Sum II - Input Array Is Sorted (LeetCode Medium)

## Problem

Given a 1-indexed array `numbers` sorted in non-decreasing order, find two numbers that add up to `target`. Return their 1-indexed positions as `[index1, index2]`. Exactly one solution is guaranteed.

```
numbers = [2,7,11,15], target = 9 -> [1,2]   (2+7=9)
```

## How to explain it out loud

*"Since the array is already sorted, I don't need a hashmap like the original Two Sum — two pointers from both ends can do it in O(1) extra space. Start left at index 0, right at the last index. If the sum of the two is too small, the only way to increase it is to move the left pointer right, toward bigger values — moving right wouldn't help, it can only make the sum smaller or equal. Symmetrically, if the sum is too big, move the right pointer left. If they're equal, that's the answer. Because the array is sorted, this greedy narrowing never skips over the actual answer."*

## Approach

Two pointers, `l=0` and `r=n-1`. While `l < r`: compute `numbers[l] + numbers[r]`. If it's less than `target`, the sum needs to grow — since the array is sorted ascending, only advancing `l` (to a larger value) can grow the sum, so `l++`. If it's greater than `target`, only moving `r` inward (to a smaller value) can shrink the sum, so `r--`. If it equals `target`, return `{l+1, r+1}` (converting back to 1-indexed).

This is safe to do greedily (without missing the true answer) precisely because of the sorted order: at any point, if the current sum is too small, no pairing involving the current `l` and *any* index to the left of `r` could ever be large enough either (since all those sums would be even smaller or equal) — wait, more precisely: if `numbers[l]+numbers[r] < target`, then pairing `l` with any index `< r` is even smaller (sorted ascending, so `numbers[l] + numbers[k]` for `k < r` is `<= numbers[l]+numbers[r]`), so `l` itself can never be part of the answer with any of those — the only hope is a larger left value, hence advance `l`.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int n = numbers.size();
        int l = 0, r = n - 1;
        while (l < r) {
            if (numbers[l] + numbers[r] < target) l++;
            else if (numbers[l] + numbers[r] > target) r--;
            else return {l+1, r+1};
        }
        return {};
    }
};
```

### Python
```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            total = numbers[l] + numbers[r]
            if total < target:
                l += 1
            elif total > target:
                r -= 1
            else:
                return [l+1, r+1]
        return []
```

### Java
```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int l = 0, r = numbers.length - 1;
        while (l < r) {
            int sum = numbers[l] + numbers[r];
            if (sum < target) l++;
            else if (sum > target) r--;
            else return new int[]{l+1, r+1};
        }
        return new int[]{};
    }
}
```

Verified against 4 cases (classic example, three-element array, negative numbers, a wider array with a duplicate value) — C++ and Python outputs match exactly on all 4, C++ additionally clean under AddressSanitizer. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the first attempt. The only follow-up was a style question about returning `{l+1, r+1}` / `{}` directly via list-initialization instead of explicit `vector<int>{...}` — confirmed this is already idiomatic, correct modern C++ (the compiler infers the target type from the function's return type), no change needed.
