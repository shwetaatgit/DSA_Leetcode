# Summary Ranges (LeetCode Easy)

## Problem

Given a sorted, unique array of integers `nums`, return the smallest sorted list of ranges that exactly covers all the numbers in the array. Each element is either a single number (formatted as `"x"`) or a consecutive run (formatted as `"x->y"`).

```
nums = [0,1,2,4,5,7]     -> ["0->2","4->5","7"]
nums = [0,2,3,4,6,8,9]   -> ["0","2->4","6","8->9"]
```

## How to explain it out loud

*"Since the array's sorted and every value is unique, a 'run' is just a maximal stretch where each next element is exactly one more than the last. So for each starting index `i`, I extend a second pointer `j` forward as long as `nums[j+1] == nums[j]+1` — that finds the end of the current run in one pass. If `j` moved past `i`, it's a real range, format it as `x->y`; otherwise it's a lone value. Then jump `i` straight to `j` so the next iteration starts fresh at the next unconsumed element. Every element gets visited by exactly one of the inner `while` steps across the whole scan, so it's still linear time even with the nested loop."*

## Approach — Two pointers, run-length scan

For each `i` from `0` to `n-1`: let `j = i`, then extend `j` forward while the next element continues the run (`nums[j+1] == nums[j] + 1`). Once the inner loop stops, `[i, j]` is the maximal run starting at `i`. If `j > i`, format as `"nums[i]->nums[j]"`; otherwise (a run of length 1) format as `"nums[i]"`. Push the formatted string, then set `i = j` so the outer `for` loop's own `i++` lands on the first index of the *next* unconsumed run.

Time: O(n) — although there's a nested loop, every array index is examined by the inner `while` exactly once across the entire run of the algorithm (each index either starts a run as `i`, or gets consumed as part of extending `j`, never both in a way that revisits it), so total work across all iterations is linear, not quadratic. Space: O(1) extra (not counting the output list itself).

**Overflow pitfall**: `nums[j] + 1` is computed as a plain `int` addition. If `nums[j] == INT_MAX` (a value LeetCode's own constraints explicitly allow, `-2^31 <= nums[i] <= 2^31-1`) and there's a next element to compare against, `nums[j] + 1` overflows a 32-bit signed int — undefined behavior in C++, and in practice (with wraparound) it silently corrupts the comparison, producing a wrong merged range rather than just crashing. Fix: compute the comparison in a wider type, e.g. `(long long)nums[j+1] == (long long)nums[j] + 1`, or equivalently guard with `nums[j] != INT_MAX &&`.

## Solution

### C++
```cpp
class Solution {
public:
    vector<string> summaryRanges(vector<int>& nums) {
        int n = nums.size();
        vector<string> ans;

        string temp = "";
        for (int i = 0; i < n; i++) {
            int j = i;
            while (j + 1 < n && (long long)nums[j+1] == (long long)nums[j] + 1) j++;

            // if j > i, that means we got our range more than one element
            if (j > i) {
                temp += to_string(nums[i]);
                temp += "->";
                temp += to_string(nums[j]);
            } else {
                temp += to_string(nums[i]);
            }

            ans.push_back(temp);
            temp = "";
            i = j;
        }

        return ans;
    }
};
```

### Python
```python
class Solution:
    def summaryRanges(self, nums: list[int]) -> list[str]:
        n = len(nums)
        ans = []
        i = 0
        while i < n:
            j = i
            while j + 1 < n and nums[j+1] == nums[j] + 1:
                j += 1
            if j > i:
                ans.append(f"{nums[i]}->{nums[j]}")
            else:
                ans.append(str(nums[i]))
            i = j + 1
        return ans
```
*(Python ints don't overflow, so no wider-type guard is needed there — noted for completeness, not because Python needs the fix.)*

### Java
```java
class Solution {
    public List<String> summaryRanges(int[] nums) {
        int n = nums.length;
        List<String> ans = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            int j = i;
            while (j + 1 < n && (long) nums[j+1] == (long) nums[j] + 1) j++;
            if (j > i) {
                ans.add(nums[i] + "->" + nums[j]);
            } else {
                ans.add(String.valueOf(nums[i]));
            }
            i = j;
        }
        return ans;
    }
}
```

Verified against 10 cases: the two classic LeetCode examples, an empty array, a single element, two non-consecutive elements, three consecutive elements, `{INT_MAX}` alone, `{INT_MAX-1, INT_MAX}` as a valid two-element range, `{INT_MIN, INT_MIN+1}` as a valid range at the other extreme, and the bug-triggering case `{INT_MAX-1, INT_MAX, INT_MIN}` — all correct after the fix, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python is the same run-scan logic (no overflow guard needed, Python ints are arbitrary precision). Java mirrors the `long`-cast fix (not independently compiled in this environment — no JDK available).

## Bug log

- Original submission computed `nums[j+1] == nums[j]+1` in plain `int` arithmetic. Confirmed as a real bug, not just theoretical UB: with `nums = {INT_MAX-1, INT_MAX, INT_MIN}`, `nums[j]+1` overflows once `j` reaches the `INT_MAX` entry, and the wraparound (`INT_MAX + 1` wrapping to `INT_MIN`) made the comparison against the next element (`INT_MIN`) spuriously succeed — the buggy version merged all three values into a single wrong range `"2147483646->-2147483648"` instead of the correct two ranges `["2147483646->2147483647", "-2147483648"]`. Fixed by casting both sides of the comparison to `long long` before the addition, verified afterward against all 10 cases including this one.
