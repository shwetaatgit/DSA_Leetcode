# Rotate Array (LeetCode Medium)

## Problem

Rotate `nums` to the right by `k` steps (`k` can exceed the array length).

```
nums = [1,2,3,4,5,6,7], k = 3
-> [5,6,7,1,2,3,4]
```

## How to explain it out loud

*"Naive approach is to rotate by one position, k times — O(n·k), quadratic in the worst case. The O(n) trick is three reversals: reverse the first n-k elements, reverse the last k elements, then reverse the whole array. Reversing the two pieces first flips each piece's internal order, and reversing everything afterward puts both pieces in their correct final positions with their internal order corrected back. One helper function, reused three times, O(n) time, O(1) space."*

## Approach

1. Normalize `k` with `k %= n` (rotating by `n` is a no-op, so anything beyond one full lap is wasted work otherwise).
2. Reverse `nums[0 .. n-k-1]` (the part that moves to the back).
3. Reverse `nums[n-k .. n-1]` (the part that moves to the front).
4. Reverse the whole array `nums[0 .. n-1]`.

Time: O(n) · Space: O(1)

## Does C++ already have this?

Yes — `std::reverse(first, last)` from `<algorithm>` reverses a range in place. Note it takes a **half-open** range (`last` is one-past-the-end, not inclusive), unlike the hand-written `reverseInd` here which takes inclusive `start`/`end` indices — so `std::reverse(nums.begin(), nums.begin()+k)` reverses the first `k` elements, not `nums.begin(), nums.begin()+k-1`. Using it would replace all three `reverseInd` calls: `reverse(nums.begin(), nums.end()-k); reverse(nums.end()-k, nums.end()); reverse(nums.begin(), nums.end());`. Python has the equivalent as a slice trick (`nums[:] = nums[-k:] + nums[-k:]`, or more simply `nums[:] = nums[-k:] + nums[:-k]`). Java has `Collections.reverse()` but only for object collections, not primitive arrays — no built-in for `int[]`, hence the manual helper below.

## Solution

### C++
```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k = k % n;
        reverseInd(nums, 0, n-k-1);
        reverseInd(nums, n-k, n-1);
        reverseInd(nums, 0, n-1);
    }

    void reverseInd(vector<int>& nums, int start, int end) {
        while (start <= end) {
            int temp = nums[end];
            nums[end] = nums[start];
            nums[start] = temp;
            start++;
            end--;
        }
    }
};
```

### Python
```python
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        n = len(nums)
        k = k % n
        self.reverse_ind(nums, 0, n-k-1)
        self.reverse_ind(nums, n-k, n-1)
        self.reverse_ind(nums, 0, n-1)

    def reverse_ind(self, nums: List[int], start: int, end: int) -> None:
        while start <= end:
            nums[start], nums[end] = nums[end], nums[start]
            start += 1
            end -= 1
```

### Java
```java
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        k = k % n;
        reverseInd(nums, 0, n-k-1);
        reverseInd(nums, n-k, n-1);
        reverseInd(nums, 0, n-1);
    }

    private void reverseInd(int[] nums, int start, int end) {
        while (start <= end) {
            int temp = nums[end];
            nums[end] = nums[start];
            nums[start] = temp;
            start++;
            end--;
        }
    }
}
```

Verified against 6 cases (classic, negatives, single element with k > n, k=0, k equal to array length, 2-element with k > n) — C++ additionally clean under AddressSanitizer.

## Bug log

- First version used `while (start != end)` in the reversal helper. Works fine when `start` walks up to meet `end`, but breaks completely when `start` starts out *past* `end` — which happens whenever the effective `k` is `0` (passed directly, or via `k % n == 0`, e.g. any `k` on a single-element array). `reverseInd(nums, n, n-1)` gets called: `start=n`, `end=n-1`, and since they're already crossed, `start != end` never becomes false — `start` climbs and `end` falls forever, reading straight off the end of the array. Confirmed as a heap-buffer-overflow via AddressSanitizer, not just a logic slip. Fixed with `start <= end`, which correctly treats an already-crossed range as "nothing to do" and also handles normal ranges (with one harmless redundant self-swap on odd-length ranges).
