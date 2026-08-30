# Product of Array Except Self (LeetCode Medium)

## Problem

Given an integer array `nums`, return `answer` where `answer[i]` is the product of every element except `nums[i]`. Must run in O(n) time and **cannot use division**. Follow-up: do it with O(1) extra space (the output array itself doesn't count).

```
nums = [1,2,3,4] -> [24,12,8,6]
nums = [-1,1,0,-3,3] -> [0,0,9,0,0]
```

## How to explain it out loud

*"answer[i] is just two pieces multiplied together — the product of everything to the left of i, and the product of everything to the right of i. So first pass: build a left array where left[i] is the product of everything before index i, left[0]=1 since there's nothing to the left. Second pass, going right to left: build a right array the same way. Multiply them together, that's the brute-force O(n) space answer. To get to O(1) extra space, notice the left array can double as the output array — instead of a separate right array, keep a single running right_prod variable, walk right to left, multiply it into left[i] first, then fold nums[i] into right_prod for the next index. Order matters: you have to use right_prod before updating it, otherwise you'd be including nums[i] in its own answer."*

## Approach — O(n) extra space

Build `left[i]` = product of `nums[0..i-1]` via a left-to-right pass (`left[0]=1`). Build `right[i]` = product of `nums[i+1..end]` via a right-to-left pass (`right[n-1]=1`). `answer[i] = left[i] * right[i]`.

Time: O(n) · Space: O(n) extra (two arrays)

## Approach — O(1) extra space (optimal)

Same idea, but reuse the output array as `left` and collapse the right-side product into a single running variable `right_prod`, walked right to left. At each index: multiply `right_prod` into the already-stored left-product (`answer[i] *= right_prod`) **before** extending `right_prod` to include `nums[i]` (`right_prod *= nums[i]`). Doing it in that order keeps `right_prod` correctly representing "everything strictly to the right of the current index" at the moment it's used — updating it first would fold `nums[i]` into its own answer, effectively multiplying by the whole array instead of excluding the current element.

Time: O(n) · Space: O(1) extra (output array not counted)

## Solution

### C++
```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> left(n);
        left[0] = 1;
        for (int i = 1; i < n; i++) {
            left[i] = left[i-1] * nums[i-1];
        }
        int right_prod = 1;
        for (int i = n-1; i >= 0; i--) {
            left[i] *= right_prod;
            right_prod *= nums[i];
        }
        return left;
    }
};
```

### Python
```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        left = [1] * n
        for i in range(1, n):
            left[i] = left[i-1] * nums[i-1]
        right_prod = 1
        for i in range(n-1, -1, -1):
            left[i] *= right_prod
            right_prod *= nums[i]
        return left
```

### Java
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] left = new int[n];
        left[0] = 1;
        for (int i = 1; i < n; i++) {
            left[i] = left[i-1] * nums[i-1];
        }
        int rightProd = 1;
        for (int i = n-1; i >= 0; i--) {
            left[i] *= rightProd;
            rightProd *= nums[i];
        }
        return left;
    }
}
```

Verified against 4 cases (classic, mix of negatives/zero/multiple-zeros, two elements, two zeros) — C++ and Python outputs match exactly on all four; C++ additionally clean under AddressSanitizer. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the O(n)-space version first try (clean prefix/suffix split), and correct on the O(1)-space collapse on the first try too, including correctly reasoning through *why* the multiply-before-update ordering matters (right_prod must represent "right of i" at time of use, not "right of i" including i itself).
