# Contains Duplicate II (LeetCode Easy)

## Problem

Given an integer array `nums` and an integer `k`, return `true` if there are two *distinct* indices `i` and `j` such that `nums[i] == nums[j]` and `abs(i - j) <= k`.

```
nums = [1,2,3,1], k = 3   -> true   (indices 0 and 3, distance 3)
nums = [1,0,1,1], k = 1   -> true   (indices 2 and 3, distance 1)
nums = [1,2,3,1,2,3], k = 2 -> false (the repeated 1 and 2 are both distance 3 apart, > k)
```

## How to explain it out loud

*"I keep a hash set that represents a sliding window of exactly the `k` elements immediately before the current index — not the current index itself. At each `i`, before doing anything else, I evict whatever element just fell outside that window (the one at `i-k-1`, once `i` is far enough along that such an index exists). Then I check: is the current value already in the set? If so, it appeared somewhere within the last `k` positions, so that's a valid nearby duplicate — return true. Otherwise insert the current value and move on. Since the set only ever holds up to `k` elements, and every element does O(1) work (one erase, one lookup, one insert), the whole thing is O(n) time with O(k) space."*

## Approach — Sliding window of size k, hash set

Maintain an `unordered_set<int>` representing the window of up to `k` elements immediately preceding index `i`. For each `i`: first, if `i > k`, evict `nums[i - k - 1]` — that's the element that has just aged out of the window (once `i` exceeds `k`, the window can no longer include index `i - k - 1`, since it's now more than `k` positions behind `i`). Then check whether `nums[i]` is already present in the set — a hit means some earlier index `j` within the last `k` positions shares this value, so `abs(i - j) <= k` is satisfied — return `true`. Otherwise insert `nums[i]` and continue.

Why checking before inserting is safe (and necessary): the set at the moment of the check contains exactly the elements at indices `[max(0, i-k), i-1]` — never `nums[i]` itself — so a hit is guaranteed to come from a *different* index, satisfying the "distinct indices" requirement without any extra bookkeeping.

Time: O(n) — one erase, one lookup, one insert per index, each O(1) average for `unordered_set` · Space: O(min(n, k)) — the set never holds more than `k` elements at once.

*(Equivalent alternative: a hash map from value to its most recent index, checking `i - lastSeen[nums[i]] <= k` directly — same complexity, arguably simpler to reason about since it doesn't require the eviction step, but the sliding-window-set version generalizes more naturally to problems needing the actual multiset of a window, e.g. Sliding Window Maximum-style problems.)*

## Solution

### C++
```cpp
class Solution {
public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        unordered_set<int> set;
        for (int i = 0; i < nums.size(); ++i) {
            if (i > k) set.erase(nums[i - k - 1]);
            if (set.count(nums[i])) return true;

            set.insert(nums[i]);
        }

        return false;
    }
};
```

### Python
```python
class Solution:
    def containsNearbyDuplicate(self, nums: list[int], k: int) -> bool:
        window = set()
        for i, num in enumerate(nums):
            if i > k:
                window.discard(nums[i - k - 1])
            if num in window:
                return True
            window.add(num)
        return False
```

### Java
```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        Set<Integer> window = new HashSet<>();
        for (int i = 0; i < nums.length; i++) {
            if (i > k) window.remove(nums[i - k - 1]);
            if (window.contains(nums[i])) return true;
            window.add(nums[i]);
        }
        return false;
    }
}
```

Verified against 11 hand-picked cases (both classic true examples, a false example with a large-enough gap, a single element, an empty array, `k=0` with and without adjacent duplicates — confirming `k=0` correctly requires literally the same index, which is impossible for distinct `i,j`, so it's always false — a duplicate far apart under a huge `k`, and two duplicates exactly at/just-over the boundary distance) plus a 2000-trial randomized stress test (small arrays, small value range to force duplicates, random `k` from 0-9) against a hash-map-of-last-seen-index reference — 0 mismatches, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same logic translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- Correct on the first attempt — no bugs found.
