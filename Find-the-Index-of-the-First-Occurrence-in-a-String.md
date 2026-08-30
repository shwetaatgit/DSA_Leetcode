# Find the Index of the First Occurrence in a String (LeetCode Easy)

## Problem

Return the index of `needle`'s first occurrence in `haystack`, or `-1` if absent. (This is `strStr()`.)

```
haystack = "hello", needle = "ll" -> 2
haystack = "aaaaa", needle = "bba" -> -1
```

## How to explain it out loud

*"Brute-force sliding comparison: try every starting index in haystack where needle could still fit, and check if the substring there matches. Guard up front for needle being longer than haystack, since that's not just 'no match,' it breaks the loop bound arithmetic if left unchecked. O(n·m) worst case."*

## Approach

Slide a window of `needle`'s length across `haystack`, comparing at each start index. Stop and return the index on first match; return `-1` if the loop finishes without one.

Time: O(n·m) worst case (n = haystack length, m = needle length) · Space: O(1) beyond the substring comparison

## Solution

### C++
```cpp
class Solution {
public:
    int strStr(string haystack, string needle) {
        if (haystack.length() < needle.length()) return -1;
        for (int i = 0; i <= (int)(haystack.length()-needle.length()); i++) {
            if (haystack.substr(i, needle.length()) == needle) return i;
        }
        return -1;
    }
};
```

### Python
```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        if len(haystack) < len(needle):
            return -1
        for i in range(len(haystack) - len(needle) + 1):
            if haystack[i:i+len(needle)] == needle:
                return i
        return -1
```

### Java
```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (haystack.length() < needle.length()) return -1;
        for (int i = 0; i <= haystack.length()-needle.length(); i++) {
            if (haystack.substring(i, i+needle.length()).equals(needle)) return i;
        }
        return -1;
    }
}
```

Verified against 7 cases (mid-string match, no match, needle longer than haystack, both empty, empty needle, later match, full match) — C++ additionally clean under AddressSanitizer.

## Bug log

- Same underflow class as Remove Element earlier: `haystack.length() - needle.length()` are both unsigned (`size_t`). When `needle` is longer, the subtraction wraps to a huge number instead of going negative, the loop runs far past valid bounds, and `substr(i, ...)` eventually gets called with `i` beyond the string — throws `std::out_of_range` and crashes, rather than just returning wrong output. Fixed with an explicit guard: `if (haystack.length() < needle.length()) return -1;` before the loop ever runs.
