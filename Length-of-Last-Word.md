# Length of Last Word (LeetCode Easy)

## Problem

Given a string `s` of words and spaces, return the length of the last word (a word is a maximal run of non-space characters).

```
"   fly me   to   the moon  " -> 4    ("moon")
```

## How to explain it out loud

*"Scan from the back. First phase: skip trailing spaces until I hit an actual letter. Second phase: count letters until I hit the next space, or run out of string. Two boolean-driven phases in one backward pass, O(n) time, O(1) space."*

## Approach

Track a `wordStarted` flag. While it's false and the current character is a space, keep skipping (trailing whitespace). Once a non-space character is hit, flip the flag and start counting. If a space appears after the flag is set, that's the boundary before the word — stop.

Time: O(n) · Space: O(1)

## Solution

### C++
```cpp
class Solution {
public:
    int lengthOfLastWord(string s) {
        int count = 0;
        bool wordStarted = false;
        for (int i = s.length()-1; i >= 0; i--) {
            if (!wordStarted && s[i]==' ') continue;
            else if (wordStarted && s[i]==' ') break;
            else {
                count++;
                wordStarted = true;
            }
        }
        return count;
    }
};
```

### Python
```python
class Solution:
    def lengthOfLastWord(self, s: str) -> int:
        count = 0
        word_started = False
        for i in range(len(s)-1, -1, -1):
            if not word_started and s[i] == ' ':
                continue
            elif word_started and s[i] == ' ':
                break
            else:
                count += 1
                word_started = True
        return count
```

### Java
```java
class Solution {
    public int lengthOfLastWord(String s) {
        int count = 0;
        boolean wordStarted = false;
        for (int i = s.length()-1; i >= 0; i--) {
            if (!wordStarted && s.charAt(i)==' ') continue;
            else if (wordStarted && s.charAt(i)==' ') break;
            else {
                count++;
                wordStarted = true;
            }
        }
        return count;
    }
}
```

Verified against 5 cases (no trailing space, multiple internal/trailing spaces, longer word, single character, single trailing space) — C++ additionally clean under AddressSanitizer.

## Bug log

None — correct on first attempt.
