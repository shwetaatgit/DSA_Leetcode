# Valid Palindrome (LeetCode Easy)

## Problem

Given a string `s`, return `true` if it's a palindrome after converting all uppercase letters to lowercase and removing all non-alphanumeric characters.

```
s = "A man, a plan, a canal: Panama" -> true
s = "race a car" -> false
s = " " -> true   (empty after filtering)
```

## How to explain it out loud

*"Two ways to do this. Simplest: build a cleaned copy of the string — lowercase, alphanumeric characters only — then run standard two pointers from both ends inward, comparing characters. That's correct but uses O(n) extra space for the copy. The tighter version skips the copy entirely: two pointers directly on the original string, and inside the loop, first advance the left pointer past any non-alphanumeric characters, then the right pointer the same way from its side, then compare. The key detail is you have to keep re-checking after each skip — a run of multiple punctuation characters in a row needs the skip to happen repeatedly, not just once, before you're safe to compare."*

## Approach — Clean copy + two pointers

Build a new string containing only the lowercased alphanumeric characters of `s` (skip everything else). Then run standard two pointers (`l=0`, `r=length-1`) inward on the cleaned string, comparing characters and returning `false` on any mismatch.

Time: O(n) · Space: O(n) for the cleaned copy

## Approach — In-place two pointers (optimal, O(1) extra space)

Same idea, but operate directly on the original string — no copy. `l=0`, `r=s.length()-1`. Inside the loop: if `s[l]` isn't alphanumeric, advance `l` and go back to re-check the loop condition (`continue`) — don't fall through to a comparison yet. Same for `s[r]` on the right. Only once *both* `s[l]` and `s[r]` are alphanumeric do you compare them (case-insensitively); on a match, advance both pointers inward.

The `continue` is essential, not optional: a run of multiple consecutive non-alphanumeric characters (e.g. `" ,"` or `".."`) needs the skip to repeat until it actually lands on a real character — a single `if` (skip once, then always fall through to compare) will end up comparing an unvalidated character before it's actually ready, producing wrong answers even though it looks like it's doing the same thing.

Time: O(n) · Space: O(1) extra

## Solution

### C++ (Clean copy)
```cpp
class Solution {
public:
    bool isPalindrome(string s) {
        // remove all non-alphanumeric characters, lowercase the rest
        string str = "";
        for (int i = 0; i < s.length(); i++) {
            if (isalnum(s[i])) str += tolower(s[i]);
        }

        int l = 0, r = str.length() - 1;
        while (l < r) {
            if (str[l] != str[r]) return false;
            l++;
            r--;
        }
        return true;
    }
};
```

### C++ (In-place two pointers — optimal)
```cpp
class Solution {
public:
    bool isPalindrome(string s) {
        int l = 0, r = s.length() - 1;
        while (l < r) {
            if (!isalnum(s[l])) {
                l++; continue;
            }
            else if (!isalnum(s[r])) {
                r--; continue;
            }

            if (tolower(s[l]) != tolower(s[r])) return false;
            else {
                l++;
                r--;
            }
        }
        return true;
    }
};
```

### Python (both)
```python
class Solution:
    def isPalindrome_clean(self, s: str) -> bool:
        cleaned = "".join(c.lower() for c in s if c.isalnum())
        l, r = 0, len(cleaned) - 1
        while l < r:
            if cleaned[l] != cleaned[r]:
                return False
            l += 1
            r -= 1
        return True

    def isPalindrome(self, s: str) -> bool:
        l, r = 0, len(s) - 1
        while l < r:
            if not s[l].isalnum():
                l += 1
                continue
            if not s[r].isalnum():
                r -= 1
                continue
            if s[l].lower() != s[r].lower():
                return False
            l += 1
            r -= 1
        return True
```

### Java (In-place two pointers)
```java
class Solution {
    public boolean isPalindrome(String s) {
        int l = 0, r = s.length() - 1;
        while (l < r) {
            if (!Character.isLetterOrDigit(s.charAt(l))) {
                l++; continue;
            }
            if (!Character.isLetterOrDigit(s.charAt(r))) {
                r--; continue;
            }
            if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) {
                return false;
            }
            l++;
            r--;
        }
        return true;
    }
}
```

Verified against 8 hand-picked cases (classic example, non-palindrome with spaces, whitespace-only, empty, punctuation-flanking-single-char, non-palindrome with punctuation, punctuation on both sides, doubled-punctuation) plus a 5000-trial randomized stress test (strings mixing letters, digits, and punctuation) comparing the in-place version against the clean-copy version as ground truth — 0 mismatches. C++ both versions clean under AddressSanitizer. Python both versions match exactly on all 8 hand-picked cases. Java (in-place) is the same logic translated directly (not independently compiled in this environment — no JDK available), using `Character.isLetterOrDigit`/`toLowerCase` in place of `isalnum`/`tolower`.

## Bug log

- First attempt at the in-place version used `if (!isalnum(s[l])) l++; else if (!isalnum(s[r])) r--;` with **no `continue`**, falling straight through into the comparison every iteration regardless of whether a skip had just happened. Two compounding problems: only one side could be skipped per iteration (the `else if` meant if the left side needed skipping, the right side's non-alphanumeric status was never checked that same iteration), and a skip didn't prevent the immediately-following comparison from running against a still-unvalidated character. Confirmed as a real, high-frequency bug: failed the **classic problem example itself** (`"A man, a plan, a canal: Panama"` returned `false` instead of `true`), plus 143 mismatches out of 2000 random trials against the clean-copy reference. Concrete minimal counterexample: `.a.` (should be `true`, filters down to the single character `"a"`) — `s[0]` isn't alphanumeric so `l` advances to `1`, but then the code immediately compares `s[1]='a'` against `s[2]='.'` (never having skipped or validated the right side), sees a mismatch, and wrongly returns `false`.
- Fixed by adding `continue` after each skip, sending control back to re-check the loop condition and re-test both sides from scratch before ever reaching the comparison. This correctly handles runs of multiple consecutive non-alphanumeric characters on either or both sides. Verified afterward with 0 mismatches across 5000 random trials.
