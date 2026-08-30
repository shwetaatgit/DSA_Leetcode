# H-Index (LeetCode Medium)

## Problem

`citations[i]` = citations for a researcher's `i`-th paper. Return the h-index: the largest `h` such that at least `h` papers have `≥ h` citations each, and the rest have `≤ h`.

```
citations = [3,0,6,1,5]
-> 3   (three papers with ≥3 citations: 3,5,6; the rest are ≤3)
```

## How to explain it out loud

*"Brute force: for each candidate h from 1 to n, count how many papers have at least h citations; if that count reaches h, h is achievable. Keep the largest achievable h — the moment a candidate fails, stop, since achievability only gets harder as h grows. That's O(n²). The cleaner way: sort descending, then walk the sorted array — position i (0-indexed) can support h = i+1 as long as citations[i] is at least that big, since everything before position i is at least as large. First position where that fails caps the answer. O(n log n) for the sort."*

## Approach — Brute force

For each `h` from `1` to `n`, count papers with citations `≥ h`. If the count reaches `h`, that `h` is achievable — record it. If some `h` isn't achievable, no larger `h` will be either (the requirement only gets stricter), so stop immediately.

Time: O(n²) · Space: O(1)

## Approach — Sort (optimal)

Sort citations descending. At sorted position `i` (0-indexed), the papers `0..i` are the `i+1` most-cited papers. If `citations[i] >= i+1`, all `i+1` of those papers have at least `i+1` citations — `h = i+1` is achievable. The first position where this fails caps the answer, since citations only get smaller from there.

Time: O(n log n) · Space: O(n) or O(1) depending on sort

## Solution

### C++ (Brute force)
```cpp
class Solution {
public:
    int hIndex(vector<int>& citations) {
        int n = citations.size();
        int h = 0;
        for (int i = 1; i <= n; i++) {
            int cnt = 0;
            for (int j = 0; j < n; j++) {
                if (citations[j] >= i) {
                    cnt++;
                }
                if (cnt == i) {
                    h++; break;
                }
            }
            if (h != i) break;
        }
        return h;
    }
};
```

### C++ (Sort, optimal)
```cpp
class Solution {
public:
    int hIndex(vector<int>& citations) {
        sort(citations.rbegin(), citations.rend());
        int h = 0;
        for (int i = 0; i < (int)citations.size(); i++) {
            if (citations[i] >= i+1) h = i+1;
            else break;
        }
        return h;
    }
};
```

### Python (Sort)
```python
class Solution:
    def hIndex(self, citations: List[int]) -> int:
        citations = sorted(citations, reverse=True)
        h = 0
        for i in range(len(citations)):
            if citations[i] >= i+1:
                h = i+1
            else:
                break
        return h
```

### Java (Sort)
```java
class Solution {
    public int hIndex(int[] citations) {
        Integer[] boxed = new Integer[citations.length];
        for (int i = 0; i < citations.length; i++) boxed[i] = citations[i];
        Arrays.sort(boxed, Collections.reverseOrder());
        int h = 0;
        for (int i = 0; i < boxed.length; i++) {
            if (boxed[i] >= i+1) h = i+1;
            else break;
        }
        return h;
    }
}
```

Both approaches verified against 7 cases (classic, all papers qualify for h=n, all zeros, single zero-citation paper, single well-cited paper, one dominant paper among low ones, tied high values) — C++ additionally clean under AddressSanitizer.

## Bug log

- `if(cnt==i) h++; continue;` — missing braces meant `continue` ran unconditionally every iteration, not just when the `if` fired. Turned out harmless in this exact spot (it was already the last statement in the loop), but revealed the real bug: `cnt==i` was being checked as "does cnt currently equal i" rather than "did cnt just become i." Since `cnt` can plateau at the same value across multiple `j` when citations don't meet the threshold, this fired `h++` repeatedly for a single candidate `h`, wildly overcounting.
- Fixed by adding a `break` right after `h++`, so the inner loop stops the instant `cnt` reaches `i` — can only fire once per candidate.
- Outer loop bound `i < n` never let `i` reach `n` itself, but `h` can legitimately equal `n` (every paper has at least `n` citations). Fixed with `i <= n`. Caught with `[5,5,5]` returning `2` instead of `3`.
