# Valid Anagram (LeetCode Easy)

## Problem

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` (same letters, same counts, any order), else `false`.

**Example**
```
s = "anagram", t = "nagaram"  -> true
s = "rat",     t = "car"      -> false
```

## Approaches

**1. Brute force** — for each character in `t`, scan `s` for an unused matching character and mark it used. O(n²), and needs a length check up front since mismatched lengths can never be anagrams.

**2. Frequency map (optimal)** — count characters going up through `s`, count down going through `t` in the same map. If any count drops below zero, they can't be anagrams; if you make it through clean, they are.

Time: O(n) · Space: O(k), k = number of distinct characters (bounded by the alphabet, effectively O(1) here)

## Solution

### Python
```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False

        freq = {}
        for c in s:
            freq[c] = freq.get(c, 0) + 1
        for c in t:
            freq[c] = freq.get(c, 0) - 1
            if freq[c] < 0:
                return False
        return True
```

*Idiomatic one-liner, for reference:*
```python
from collections import Counter

class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        return Counter(s) == Counter(t)
```

### C++
```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.length() != t.length()) return false;

        unordered_map<char, int> freq;
        for (char c : s) freq[c]++;
        for (char c : t) {
            freq[c]--;
            if (freq[c] < 0) return false;
        }
        return true;
    }
};
```

### Java
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;

        Map<Character, Integer> freq = new HashMap<>();
        for (char c : s.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }
        for (char c : t.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) - 1);
            if (freq.get(c) < 0) return false;
        }
        return true;
    }
}
```

All three verified against: `("anagram","nagaram") -> true`, `("rat","car") -> false`, `("aacc","ccac") -> false`, `("ab","a") -> false`.
