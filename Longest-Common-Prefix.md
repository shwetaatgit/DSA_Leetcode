# Longest Common Prefix (LeetCode Easy)

## Problem

Given an array of strings, find the longest string that's a prefix of all of them. No common prefix → return `""`.

```
["flower","flow","flight"] -> "fl"
["dog","racecar","car"] -> ""
```

## How to explain it out loud

*"Vertical scan: walk character position by position. At each position, check every string agrees on that character — if any string disagrees, or any string has already run out of characters, stop and return what I've built so far. Otherwise append the character and move to the next position. O(n·m) worst case, where n is string count and m is the shortest string's length."*

## The bug, explained

First version used `while (i <= strs[0].length())` — the `<=` lets `i` reach exactly `strs[0].length()`, one position past the last valid character. Normally that's harmless: the inner comparison loop against other strings catches the overrun and returns before anything bad happens. But with an array of a **single string**, the inner `for (j = 1; j < strs.size(); j++)` loop never runs at all (`j < 1` is false immediately) — so there's nothing to stop the out-of-bounds append. `strs[0][strs[0].length()]` is well-defined in C++ (guaranteed to be the null terminator, `'\0'`), so it doesn't crash — it just silently appends an invisible null character to the answer. `["abc"]` returned `"abc\0"` (length 4) instead of `"abc"` (length 3).

Fixed by special-casing `strs.size() == 1`: return `strs[0]` immediately, before the loop ever runs. Since the bug only ever surfaced when the inner `for` loop had nothing to iterate over (i.e. fewer than 2 strings, and the empty case is already handled separately), catching that one input shape up front sidesteps the whole problem without touching the loop's `<=` condition. (An equally valid alternative: tighten the loop itself to `while (i < strs[0].length())`, which fixes it structurally instead of by special case — either works.)

## Approach

Walk character position `i` upward. At each `i`, compare every adjacent pair of strings at that index; a mismatch or a string running out of length means the prefix ends here.

Time: O(n·m) · Space: O(1) beyond the output

## Solution

### C++
```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty()) return "";
        else if (strs.size() == 1) return strs[0];

        string pref = "";
        int i = 0;
        while (i <= (int)strs[0].length()) {
            for (int j = 1; j < (int)strs.size(); j++) {
                if (strs[j-1][i] != strs[j][i]) return pref;
                if ((int)strs[j].length() == (int)pref.length()) return pref;
            }
            pref = pref + strs[0][i];
            i++;
        }
        return pref;
    }
};
```

### Python
```python
class Solution:
    def longestCommonPrefix(self, strs: List[str]) -> str:
        if not strs:
            return ""
        pref = ""
        i = 0
        while i < len(strs[0]):
            for j in range(1, len(strs)):
                if strs[j-1][i] != strs[j][i]:
                    return pref
                if len(strs[j]) == len(pref):
                    return pref
            pref = pref + strs[0][i]
            i += 1
        return pref
```

### Java
```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0) return "";
        String pref = "";
        int i = 0;
        while (i < strs[0].length()) {
            for (int j = 1; j < strs.length; j++) {
                if (strs[j-1].charAt(i) != strs[j].charAt(i)) return pref;
                if (strs[j].length() == pref.length()) return pref;
            }
            pref = pref + strs[0].charAt(i);
            i++;
        }
        return pref;
    }
}
```

Verified against 7 cases (single string, classic partial match, no common prefix, prefix-of-shorter-string, single character, empty strings, single empty string) — C++ additionally clean under AddressSanitizer.

## Bug log

- `while (i <= strs[0].length())` allowed `i` one position past the last valid index. Harmless with 2+ strings (the inner loop's comparisons or length check always caught it first), but with a single-string array the inner loop never executes, so nothing stopped an out-of-bounds — well-defined but wrong — append of the null terminator. Caught with `["abc"]` returning a 4-character string instead of 3. Fixed by special-casing `strs.size() == 1`.
