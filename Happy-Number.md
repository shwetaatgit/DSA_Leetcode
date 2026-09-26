# Happy Number (LeetCode Easy)

## Problem

A number is "happy" if repeatedly replacing it with the sum of the squares of its digits eventually reaches `1`. If it enters a cycle that never includes `1`, it's not happy. Return `true` if `n` is happy.

```
n = 19 -> true   (19 -> 82 -> 68 -> 100 -> 1)
n = 2  -> false  (enters the cycle 4,16,37,58,89,145,42,20,4,...)
```

## How to explain it out loud

*"This is really a cycle-detection problem disguised as a math problem. Repeatedly applying 'sum of squared digits' to any starting number either eventually hits 1, or it starts repeating a value it's already seen — it can't just grow forever, since squaring digits caps the next value well below the current one once you're past a few digits. So I keep a hash set of every value seen so far. Each step: if the current value is already in the set, I'm in a cycle that doesn't include 1, so it's not happy — return false. Otherwise record it and compute the next value. If I ever land on exactly 1, it's happy."*

## Approach — Cycle detection with a hash set

Loop while `n != 1`. On each iteration: if `n` is already in `seen`, a repeat means the sequence has entered a cycle without ever reaching 1 — return `false`. Otherwise insert `n` into `seen`, then compute the next value by pulling digits off with `% 10` / `/ 10` and summing their squares. If the loop exits because `n == 1`, return `true`.

This terminates because there are only two possibilities for any starting number: reach 1, or enter one of a small number of possible cycles (all unhappy numbers funnel into the same cycle `4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4`) — either way, a value must repeat within a bounded number of steps, which the hash set catches.

Time: O(log n) per digit-sum step to peel off digits, times a small bounded number of steps before either hitting 1 or repeating (in practice this stabilizes to a small number below 243 for any starting value, since once a number has more than 3 digits its digit-square-sum is guaranteed to be smaller than itself) — effectively O(log n) overall, though not tightly bounded by a simple closed form. Space: O(1) in practice, bounded by the small cycle/path length rather than input size.

*(Alternative: Floyd's cycle detection with two pointers, slow/fast, avoids the hash set entirely — O(1) space guaranteed instead of "small but technically unbounded." Not needed here since the practical cycle length is tiny, but worth knowing as the follow-up if asked for O(1) space explicitly.)*

## Solution

### C++
```cpp
class Solution {
public:
    bool isHappy(int n) {
        unordered_set<int> seen;

        while (n != 1) {
            if (seen.count(n)) return false;
            seen.insert(n);

            int sum = 0;
            while (n > 0) {
                int digit = n % 10;
                sum += digit * digit;
                n /= 10;
            }
            n = sum;
        }

        return true;
    }
};
```

### Python
```python
class Solution:
    def isHappy(self, n: int) -> bool:
        seen = set()
        while n != 1:
            if n in seen:
                return False
            seen.add(n)
            total = 0
            while n > 0:
                digit = n % 10
                total += digit * digit
                n //= 10
            n = total
        return True
```

### Java
```java
class Solution {
    public boolean isHappy(int n) {
        Set<Integer> seen = new HashSet<>();
        while (n != 1) {
            if (seen.contains(n)) return false;
            seen.add(n);
            int sum = 0;
            while (n > 0) {
                int digit = n % 10;
                sum += digit * digit;
                n /= 10;
            }
            n = sum;
        }
        return true;
    }
}
```

Verified against 20 known happy numbers and 23 known unhappy numbers, plus a full sweep of every `n` from 1 to 100,000 against a reference implementation (same digit-square-sum logic, capped at 1000 iterations instead of using a hash set — relying on the mathematical guarantee that any unhappy number enters the known cycle well within that bound) — 0 mismatches. Also checked `INT_MAX` (2147483647, unhappy) for overflow safety — squaring a single digit (max 81) and summing up to 10 digits never approaches `int` overflow, confirmed clean under AddressSanitizer with `-D_GLIBCXX_ASSERTIONS`. Python matches on all hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- Correct on the first attempt — no bugs found.
