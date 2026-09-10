# 3Sum (LeetCode Medium)

## Problem

Given an integer array `nums`, find all unique triplets `[nums[i], nums[j], nums[k]]` (distinct indices) that sum to zero. The solution set must not contain duplicate triplets.

```
nums = [-1,0,1,2,-1,-4] -> [[-1,-1,2],[-1,0,1]]
```

## How to explain it out loud

*"Brute force: try every triplet of indices, check if they sum to zero, and dedupe using a set since the same triplet of values can come from different index combinations when there are duplicate numbers — O(n³). The optimized version sorts the array first, then reduces the problem to something already solved: fix one number, and find two others that sum to its negative — that's exactly Two Sum II, solvable with two pointers in O(n) once sorted. So the outer loop picks a fixed index i, and the inner two-pointer search covers everything after it. The tricky part is avoiding duplicate triplets without a set: since the array's sorted, duplicates sit next to each other, so I just skip past repeated values — for the fixed index, skip if it matches the previous one; after finding a valid pair, skip the two pointers past any further occurrences of their own values before continuing."*

## Approach — Brute force

Try every triplet of distinct indices `(i,j,k)`. If they sum to zero, sort that triplet's values and insert into a set to naturally dedupe (the same value-triplet can arise from multiple index combinations when the array has duplicate values). Convert the set to the final result list.

Time: O(n³) for the triple loop, plus O(log(count)) per set insertion · Space: O(n³) worst case for candidate triplets before dedup

## Approach — Sort + two pointers (optimal)

Sort `nums`. Loop a fixed index `i` from `0` to `n-3`; `nums[i]` is treated as one of the three numbers, and the other two must sum to `-nums[i]` — exactly the Two Sum II two-pointer search, run on the sorted subarray `i+1 .. n-1`.

Two separate dedup rules, both possible without a set because sorting puts duplicate values adjacent to each other:
- **Fixed-element dedup:** if `nums[i] == nums[i-1]`, skip this `i` — any triplet startable from this value was already found on the previous (identical) `i`.
- **Pair dedup:** once a valid `l`/`r` pair is found, before moving on, skip `l` forward past any run of equal values, and skip `r` backward past any run of equal values, *then* move each one step further. Without this, the same triplet gets re-recorded as the pointers step through a run of duplicates one at a time.

Time: O(n²) — O(n log n) for the sort, dominated by O(n²) for the outer loop times the linear two-pointer scan · Space: O(1) extra beyond the output (or O(n)/O(log n) depending on the sort's implementation)

## Solution

### C++ (Brute force)
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n = nums.size();
        set<vector<int>> uniqueTriplets;

        for (int i = 0; i < n; i++) {
            for (int j = i+1; j < n; j++) {
                for (int k = j+1; k < n; k++) {
                    if (nums[i] + nums[j] + nums[k] == 0) {
                        vector<int> triplet = {nums[i], nums[j], nums[k]};
                        sort(triplet.begin(), triplet.end());
                        uniqueTriplets.insert(triplet);
                    }
                }
            }
        }

        return vector<vector<int>>(uniqueTriplets.begin(), uniqueTriplets.end());
    }
};
```

### C++ (Sort + two pointers — optimal)
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;

        for (int i = 0; i < n-2; i++) {
            if (i > 0 && nums[i-1] == nums[i]) continue;

            int l = i + 1;
            int r = n - 1;
            while (l < r) {
                if (nums[i] + nums[l] + nums[r] == 0) {
                    vector<int> v;
                    v.push_back(nums[i]);
                    v.push_back(nums[l]);
                    v.push_back(nums[r]);
                    result.push_back(v);

                    while (l < n-1 && nums[l] == nums[l+1]) l++;
                    while (r > 0 && nums[r] == nums[r-1]) r--;
                    l++;
                    r--;
                }
                else if (nums[i] + nums[l] + nums[r] > 0) {
                    while (r > 0 && nums[r] == nums[r-1]) r--;
                    r--;
                }
                else {
                    while (l < n-1 && nums[l] == nums[l+1]) l++;
                    l++;
                }
            }
        }
        return result;
    }
};
```

### Python (Sort + two pointers)
```python
class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums = sorted(nums)
        n = len(nums)
        result = []
        for i in range(n-2):
            if i > 0 and nums[i-1] == nums[i]:
                continue
            l, r = i+1, n-1
            while l < r:
                total = nums[i] + nums[l] + nums[r]
                if total == 0:
                    result.append([nums[i], nums[l], nums[r]])
                    while l < n-1 and nums[l] == nums[l+1]:
                        l += 1
                    while r > 0 and nums[r] == nums[r-1]:
                        r -= 1
                    l += 1
                    r -= 1
                elif total > 0:
                    while r > 0 and nums[r] == nums[r-1]:
                        r -= 1
                    r -= 1
                else:
                    while l < n-1 and nums[l] == nums[l+1]:
                        l += 1
                    l += 1
        return result
```

### Java (Sort + two pointers)
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        int n = nums.length;
        List<List<Integer>> result = new ArrayList<>();

        for (int i = 0; i < n-2; i++) {
            if (i > 0 && nums[i-1] == nums[i]) continue;

            int l = i + 1, r = n - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[l], nums[r]));
                    while (l < n-1 && nums[l] == nums[l+1]) l++;
                    while (r > 0 && nums[r] == nums[r-1]) r--;
                    l++;
                    r--;
                } else if (sum > 0) {
                    while (r > 0 && nums[r] == nums[r-1]) r--;
                    r--;
                } else {
                    while (l < n-1 && nums[l] == nums[l+1]) l++;
                    l++;
                }
            }
        }
        return result;
    }
}
```

Verified the optimized version against a brute-force set-based reference (order-independent comparison) on 8 hand-picked cases (classic example, no valid triplet, all-zeros, more zeros than needed, empty array, two-element array, a case with multiple duplicate values, and a larger array with heavy duplication producing 6 distinct triplets) plus a 500-trial randomized stress test using a narrow value range to force frequent duplicates — 0 mismatches. C++ (both versions) clean under AddressSanitizer. Python matches the C++ two-pointer version exactly on all 8 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — the two-pointer version with dedup logic was correct on the first attempt, including both dedup rules (fixed-element skip and post-match pointer skip), verified against 500 random stress trials with heavy duplication.
