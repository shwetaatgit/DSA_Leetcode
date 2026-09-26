# Ransom Note (LeetCode Easy)

## Problem

Given two strings `ransomNote` and `magazine`, return `true` if `ransomNote` can be constructed by using the letters from `magazine`. Each letter in `magazine` can only be used once.

```
ransomNote = "a", magazine = "b" -> false
ransomNote = "aa", magazine = "aab" -> true
```

## How to explain it out loud

*"Count how many of each character are available in the magazine, using a hashmap. Then walk through the ransom note: for each character, if the magazine never had that character at all, it's immediately impossible. Otherwise, decrement its count — that represents 'using up' one occurrence. If a count ever goes negative, it means the ransom note needed more of that character than the magazine actually had, so it's impossible. If the whole ransom note is processed without either failure, the magazine has enough of everything needed."*

## Approach

Build a frequency map of every character in `magazine`. Then walk through `ransomNote`: for each character, if it's not a key in the map at all, return `false` immediately (magazine never had that letter). Otherwise decrement its count; if the count drops below zero, return `false` (more of that letter was needed than available). If the entire `ransomNote` is processed without triggering either failure, return `true`.

Time: O(m + r), where `m`/`r` are the lengths of `magazine`/`ransomNote` · Space: O(1) effectively (at most 26 lowercase letters, so the map size is bounded by the alphabet)

## Solution

### C++
```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        map<char, int> counts;
        int rLen = ransomNote.length();
        int mLen = magazine.length();

        for (int i = 0; i < mLen; i++) {
            char f = magazine[i];
            if (counts.find(f) != counts.end()) {
                counts[f] += 1;
            }
            else {
                counts.insert(pair<char, int>(f, 1));
            }
        }
        for (int i = 0; i < rLen; i++) {
            char f = ransomNote[i];
            if (counts.find(f) == counts.end()) {
                return false;
            }
            counts[f] -= 1;
            if (counts[f] < 0) {
                return false;
            }
        }
        return true;
    }
};
```

### Python
```python
class Solution:
    def canConstruct(self, ransomNote: str, magazine: str) -> bool:
        counts = {}
        for ch in magazine:
            counts[ch] = counts.get(ch, 0) + 1
        for ch in ransomNote:
            if ch not in counts:
                return False
            counts[ch] -= 1
            if counts[ch] < 0:
                return False
        return True
```

### Java
```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        Map<Character, Integer> counts = new HashMap<>();
        for (char ch : magazine.toCharArray()) {
            counts.merge(ch, 1, Integer::sum);
        }
        for (char ch : ransomNote.toCharArray()) {
            if (!counts.containsKey(ch)) return false;
            counts.put(ch, counts.get(ch) - 1);
            if (counts.get(ch) < 0) return false;
        }
        return true;
    }
}
```

Verified against 7 cases (classic false example, classic true example, a single-letter-short case, empty ransom note, empty magazine, repeated letters needing more than available, exact match) — C++ and Python outputs match exactly on all 7, C++ additionally clean under AddressSanitizer. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the first attempt.
