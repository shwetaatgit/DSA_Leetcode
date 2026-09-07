# Candy (LeetCode Hard)

## Problem

`n` children stand in a line, each with a rating `ratings[i]`. Every child gets at least 1 candy. Any child with a higher rating than a neighbor must receive more candies than that neighbor. Return the minimum total candies needed.

```
ratings = [1,0,2] -> 5   (candies: [2,1,2])
ratings = [1,2,2] -> 4   (candies: [1,2,1] — equal ratings, no strict requirement between them)
```

## How to explain it out loud

*"Each child has two independent constraints — 'more than my left neighbor if I'm rated higher' and 'more than my right neighbor if I'm rated higher' — and a single left-to-right scan can only naturally enforce one of those at a time. So do it in two passes. Left-to-right: give everyone 1 candy to start; whenever ratings[i] > ratings[i-1], set candies[i] = candies[i-1]+1 — this correctly satisfies every 'left' constraint. Right-to-left: whenever ratings[i] > ratings[i+1], the child needs more than their right neighbor too — but I can't just overwrite what the left pass already gave them, since that might violate the left constraint I already satisfied. So I take candies[i] = max(candies[i], candies[i+1]+1) — keep whichever requirement is stricter. Sum everything at the end."*

## Approach

Initialize a `candies` array to all `1`s (everyone gets at least one).

**Left-to-right pass:** for each `i` from `1` to `n-1`, if `ratings[i] > ratings[i-1]`, set `candies[i] = candies[i-1] + 1`. This guarantees every child rated higher than their *left* neighbor has strictly more candies than that neighbor.

**Right-to-left pass:** for each `i` from `n-2` down to `0`, if `ratings[i] > ratings[i+1]`, the child needs more than their *right* neighbor. Don't just overwrite — take `candies[i] = max(candies[i], candies[i+1] + 1)`, so whichever constraint (left-pass or right-pass) demanded more candies wins, and neither gets violated by the other.

Sum the final `candies` array.

Why `max` and not a plain overwrite: consider a rising-then-falling sequence like `[1,3,2,2,1]`. The left pass already gave the peak (`3`) enough candy to beat its left neighbor. The right pass, scanning backward, separately determines how much candy each position needs to beat its right neighbor. Taking the max of the two independently-computed requirements is what guarantees both directions are satisfied simultaneously without one pass's fix undoing the other's.

Time: O(n) (two linear passes) · Space: O(n) for the candies array

## Solution

### C++
```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        if (n == 0) return 0;

        vector<int> candies(n, 1);

        for (int i = 1; i < n; i++) {
            if (ratings[i] > ratings[i-1]) {
                candies[i] = candies[i-1] + 1;
            }
        }

        for (int i = n-2; i >= 0; i--) {
            if (ratings[i] > ratings[i+1]) {
                candies[i] = max(candies[i], candies[i+1] + 1);
            }
        }

        int totalCandies = 0;
        for (int i = 0; i < n; i++) {
            totalCandies += candies[i];
        }

        return totalCandies;
    }
};
```

### Python
```python
class Solution:
    def candy(self, ratings: List[int]) -> int:
        n = len(ratings)
        if n == 0:
            return 0
        candies = [1] * n
        for i in range(1, n):
            if ratings[i] > ratings[i-1]:
                candies[i] = candies[i-1] + 1
        for i in range(n-2, -1, -1):
            if ratings[i] > ratings[i+1]:
                candies[i] = max(candies[i], candies[i+1] + 1)
        return sum(candies)
```

### Java
```java
class Solution {
    public int candy(int[] ratings) {
        int n = ratings.length;
        if (n == 0) return 0;

        int[] candies = new int[n];
        Arrays.fill(candies, 1);

        for (int i = 1; i < n; i++) {
            if (ratings[i] > ratings[i-1]) {
                candies[i] = candies[i-1] + 1;
            }
        }

        for (int i = n-2; i >= 0; i--) {
            if (ratings[i] > ratings[i+1]) {
                candies[i] = Math.max(candies[i], candies[i+1] + 1);
            }
        }

        int total = 0;
        for (int c : candies) total += c;
        return total;
    }
}
```

Verified against 7 hand-picked cases (both problem examples, strictly increasing, strictly decreasing, a rise-then-fall peak, all-equal ratings, single child) plus a 500-trial randomized stress test against an independent constraint-relaxation reference implementation (repeatedly fixes any local violation until no changes remain, a different and definitely-correct — if slower — algorithm) — 0 mismatches. C++ additionally clean under AddressSanitizer. Python matches on all 7 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- Correct on the first attempt — the two-pass left/right with `max` combine was written and verified with no fixes needed.
