# Is Subsequence (LeetCode Easy)

## Problem

Given two strings `s` and `t`, return `true` if `s` is a subsequence of `t` — meaning every character of `s` appears in `t`, in the same relative order, but not necessarily contiguously.

```
s = "abc", t = "ahbgdc" -> true
s = "axc", t = "ahbgdc" -> false
```

## How to explain it out loud

*"Two pointers, but they don't move in lockstep — one walks through t (always advancing, every character of t gets looked at exactly once), the other walks through s but only advances when it finds a match. Walk i through t; whenever t[i] matches the character s is currently looking for, advance the s-pointer to look for the next character. If the s-pointer ever reaches the end of s, every character was found in order, so s is a subsequence. One pass over t, O(n) time, no extra space."*

## Approach

Early exit: if `s` is longer than `t`, it can't possibly be a subsequence — return `false` immediately.

Track `s_index`, starting at `0`. Walk `i` through every character of `t`. Whenever `t[i]` equals the character `s` is currently waiting to match (`s[s_index]`), advance `s_index`. After the full pass over `t`, `s` is a subsequence of `t` if and only if `s_index` reached `s.length()` — meaning every character of `s` was matched, in order.

Important: the "did we finish matching all of `s`" check must happen **after** the loop completes (or independently of whether the loop body ran at all), not only from inside the loop — otherwise an edge case where `t` itself is empty (so the loop body never executes even once) never gets the chance to confirm that an equally-empty `s` trivially matches.

Time: O(len(t)) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        if (s.length() > t.length()) return false;

        int s_index = 0;
        for (int i = 0; i < t.length(); i++) {
            if (s[s_index] == t[i]) s_index++;
        }

        if (s_index == s.length()) return true;
        return false;
    }
};
```

### Python
```python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        if len(s) > len(t):
            return False
        s_index = 0
        for i in range(len(t)):
            # guard needed here: unlike C++ (where s[s.size()] safely reads '\0'),
            # Python raises IndexError on s[s_index] once s_index == len(s)
            if s_index < len(s) and s[s_index] == t[i]:
                s_index += 1
        return s_index == len(s)
```

### Java
```java
class Solution {
    public boolean isSubsequence(String s, String t) {
        if (s.length() > t.length()) return false;

        int sIndex = 0;
        for (int i = 0; i < t.length(); i++) {
            if (sIndex < s.length() && s.charAt(sIndex) == t.charAt(i)) {
                sIndex++;
            }
        }
        return sIndex == s.length();
    }
}
```

Verified against 7 hand-picked cases (both problem examples, empty `s` with non-empty `t`, both empty, non-empty `s` with empty `t`, `s` equal to `t`, `s` one character short of `t`) plus a 3000-trial randomized stress test against an independent reference implementation — 0 mismatches, C++ additionally clean under AddressSanitizer. Python matches on all 7 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), with the same out-of-bounds guard Python needed, since Java also throws on out-of-range `charAt`.

## Bug log

- First attempt checked `if (s_index == s.length()) return true;` **inside** the `for` loop, right after each character comparison — this correctly handled every case where the loop body ran at least once, including empty `s` with non-empty `t` (the check fires on the very first iteration, since `s_index` starts already equal to `s.length()` when `s` is empty). But when `t` is *also* empty, the loop body never runs at all — nothing to iterate over — so that check never gets evaluated, and execution fell through to a hardcoded `return false;`. Confirmed as a real, narrow bug: `s=""`, `t=""` returned `false` instead of the correct `true` (the empty string is trivially a subsequence of any string, including another empty one) — this exact case accounted for all 32 mismatches in an initial stress test, with every other case already correct. Fixed by moving the completion check to run unconditionally after the loop, rather than only from within it.
