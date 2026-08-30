# Remove Duplicates from Sorted Array II (LeetCode Medium)

## Problem

Sorted array `nums`. Remove duplicates in-place so each value appears **at most twice**. Return `k`, the new length — `nums[0..k-1]` must hold the result in order.

```
nums = [1,1,1,2,2,3]
-> k = 5, nums = [1,1,2,2,3,_]

nums = [0,0,1,1,1,1,2,3,3]
-> k = 7, nums = [0,0,1,1,2,3,3,_,_]
```

## How to explain it out loud

*"This is the same write-pointer pattern as basic Remove Duplicates, just relaxed from 'compare one back' to 'compare two back.' I keep a write pointer, `insertInd`, and a read pointer, `currInd`. The first two elements are always safe to keep — no array can have three copies within its first two slots. From index 2 onward, before writing `nums[currInd]`, I check it against whatever is sitting two slots behind the write pointer, `nums[insertInd-2]`. If they're equal, that means the two most recently kept elements are already both this value — writing a third would break the 'at most twice' rule, so I skip it. If they differ, it's safe, I write it and advance. One pass, O(n) time, O(1) space."*

## Step-by-step (why `nums[insertInd-2]`, specifically)

1. **Base case**: if `nums.size() <= 2`, every element trivially satisfies "at most 2 copies" — return the size as-is, nothing to do.
2. **Seed the write pointer at 2**: the first two elements of any array can never violate the rule on their own, so they're kept unconditionally. `insertInd` starts at `2`, meaning "the next write goes here."
3. **Scan `currInd` from index 2 to the end.** At each step, ask: *if I wrote `nums[currInd]` right now, would it be a third copy in a row of the same value?*
4. **The answer is exactly `nums[insertInd-2]`.** Everything written so far lives in `nums[0..insertInd-1]`, and it's sorted, so the two *most recently written* values are `nums[insertInd-2]` and `nums[insertInd-1]`. If `nums[currInd]` equals `nums[insertInd-2]`, that means both of those last two slots — and now this candidate — would all be the same value: three in a row. Skip it.
5. **If it's different, it's always safe** — writing it can add at most one more copy of that value next to whatever's already there, never a third.
6. **`insertInd` at the end is the answer**, and `nums[0..insertInd-1]` is the array, rearranged in place.

### Worked trace on `[0,0,1,1,1,1,2,3,3]`

| currInd | nums[currInd] | nums[insertInd-2] | equal? | action | insertInd after |
|---|---|---|---|---|---|
| 2 | 1 | nums[0]=0 | no | write nums[2]=1 | 3 |
| 3 | 1 | nums[1]=0 | no | write nums[3]=1 | 4 |
| 4 | 1 | nums[2]=1 | **yes** | skip | 4 |
| 5 | 1 | nums[3]=1 | **yes** | skip | 4 |
| 6 | 2 | nums[2]=1 | no | write nums[4]=2 | 5 |
| 7 | 3 | nums[3]=1 | no | write nums[5]=3 | 6 |
| 8 | 3 | nums[4]=2 | no | write nums[6]=3 | 7 |

Final: `insertInd = 7`, `nums[0..6] = [0,0,1,1,2,3,3]`. Matches expected.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n = nums.size();
        if (n <= 2) return n;

        int insertInd = 2;
        for (int currInd = 2; currInd < n; currInd++) {
            if (nums[currInd] != nums[insertInd-2]) {
                nums[insertInd] = nums[currInd];
                insertInd++;
            }
        }
        return insertInd;
    }
};
```

### Python
```python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        n = len(nums)
        if n <= 2:
            return n
        insert_ind = 2
        for curr_ind in range(2, n):
            if nums[curr_ind] != nums[insert_ind-2]:
                nums[insert_ind] = nums[curr_ind]
                insert_ind += 1
        return insert_ind
```

### Java
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        if (n <= 2) return n;
        int insertInd = 2;
        for (int currInd = 2; currInd < n; currInd++) {
            if (nums[currInd] != nums[insertInd-2]) {
                nums[insertInd] = nums[currInd];
                insertInd++;
            }
        }
        return insertInd;
    }
}
```

Verified against 6 cases (classic 3-in-a-row, longer mixed run, exactly 2 elements, single element, all-same 4-in-a-row, no duplicates at all) — C++ additionally clean under AddressSanitizer.

## Bug log

None — correct on first attempt. (An earlier hash-map version was proposed first — correct, O(n) time, but O(k) extra space for the frequency map; this write-pointer version gets it to O(1) space.)
