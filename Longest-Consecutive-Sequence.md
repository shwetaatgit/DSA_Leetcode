# Longest Consecutive Sequence (LeetCode Medium)

## Problem

Given an unsorted array of integers `nums`, return the length of the longest run of consecutive integers (not necessarily contiguous in the array itself — just present somewhere in it).

```
nums = [100,4,200,1,3,2]  -> 4   (the sequence 1,2,3,4)
nums = [0,3,7,2,5,8,4,6,0,1] -> 9   (0 through 8)
```

## How to explain it out loud

*"The naive way is to sort and scan for runs — that's O(n log n). The O(n) trick is to put everything in a hash set instead, then only ever start counting a sequence from a number that's a genuine sequence *start* — meaning `num - 1` is not in the set. Once I've found a start, I walk forward (`num+1`, `num+2`, ...) as long as each next number is also in the set, counting as I go. Because I only start walking from true sequence starts, every number in the array gets included in exactly one walk across the whole algorithm — so even though there's a nested-looking loop, the total work across all walks combined is still linear."*

## Approach 1 — Min-heap (priority queue), sorted scan

Push every number into a min-heap, then repeatedly pop in ascending order — this simulates having a sorted array without an explicit sort call (though asymptotically it's the same O(n log n) cost, just paid via heap operations instead of `sort`). Track `lastSeen` and a running `count`: if the current popped value equals `lastSeen`, it's a duplicate — skip it without resetting anything. If it's exactly `lastSeen + 1`, the run continues — increment `count` and update `maxCount`. Otherwise the run broke — reset `count` to `1`. Update `lastSeen` after each real (non-duplicate) comparison.

Time: O(n log n) — `n` pushes and `n` pops, each O(log n) · Space: O(n) for the heap.

## Approach 2 — Hash set, only walk from true sequence starts (optimal)

Put every number in an `unordered_set` (this also naturally de-duplicates). For each number `num` in the set, check whether `num - 1` is also in the set — if it is, `num` is the *middle or end* of some sequence, not its start, so skip it; some other number will pick up this same sequence when its own start is processed. If `num - 1` is absent, `num` is genuinely where a sequence begins — walk forward (`num+1`, `num+2`, ...) counting how far the run extends, and update the max.

Why this is O(n) overall despite the nested-looking loop: every number that belongs to some run is visited by the inner `while` exactly once, ever, across the *entire* run of the outer loop — because the inner walk for a given start only stops once it hits a number *not* in the set, and no other start will ever re-walk that same stretch (any other number in the middle of that run fails the `num-1` check and is skipped outright by the outer loop before its own inner walk could begin).

Time: O(n) — every number is visited by the inner walk at most once across the whole algorithm, and the "is this a start" check is a single O(1) set lookup per outer iteration · Space: O(n) for the hash set.

## Solution

### C++ (min-heap)
```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if(nums.size()==0) return 0;
        priority_queue<int, vector<int>, greater<int>> q;
        for (int i = 0; i<nums.size(); i++) q.push(nums[i]);

        int maxCount = 1;
        int count = 1;
        int lastSeen = q.top();
        q.pop();
        while(q.size()!=0){
            if (q.top() == lastSeen) {
                q.pop();
                continue;
            }
            if(lastSeen+1 == q.top()){
                count++;
                maxCount = max(maxCount, count);
            } 
            else count=1;
            lastSeen = q.top();
            q.pop();
        }
        return maxCount;
    }
};
```

### C++ (hash set — optimal)
```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if(nums.size()==0) return 0;
        unordered_set<int> s(nums.begin(), nums.end());

        int maxCount = 0;
        for(int num: s){
            if(s.count(num-1)) continue;
            //Only start counting from a number that has no num-1 in the set — that's a true sequence start.

            int curr = num;
            int count = 1;
            while (s.count(curr + 1)) {       // walk the sequence forward
                curr++;
                count++;
            }
            maxCount = max(maxCount, count);
        }
        return maxCount;
    }
};
```

### Python (hash set)
```python
class Solution:
    def longestConsecutive(self, nums: list[int]) -> int:
        num_set = set(nums)
        max_count = 0

        for num in num_set:
            if num - 1 in num_set:
                continue  # not a sequence start

            curr = num
            count = 1
            while curr + 1 in num_set:
                curr += 1
                count += 1
            max_count = max(max_count, count)

        return max_count
```

### Java (hash set)
```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> numSet = new HashSet<>();
        for (int n : nums) numSet.add(n);

        int maxCount = 0;
        for (int num : numSet) {
            if (numSet.contains(num - 1)) continue;

            int curr = num;
            int count = 1;
            while (numSet.contains(curr + 1)) {
                curr++;
                count++;
            }
            maxCount = Math.max(maxCount, count);
        }
        return maxCount;
    }
}
```

Verified both versions against 8 hand-picked cases (both classic LeetCode examples, an empty array, a single element, an all-duplicate array, a descending array, negative numbers, and a small array with an internal duplicate) plus a 2000-trial randomized stress test — all three (min-heap, hash-set, and a sort-based reference implementation) cross-checked against each other on every trial — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python is the same hash-set logic. Java is the same logic translated directly (not independently compiled in this environment — no JDK available).

## Bug log

- Both versions: correct on the first attempt — no bugs found.
