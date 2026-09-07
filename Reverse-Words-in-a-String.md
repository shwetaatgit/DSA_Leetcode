# Reverse Words in a String (LeetCode Medium)

## Problem

Given a string `s`, reverse the order of the words. Words are separated by whitespace; input may have leading/trailing spaces or multiple spaces between words. Output must have a single space between words and no leading/trailing spaces.

```
s = "the sky is blue"   -> "blue is sky the"
s = "  hello world  "   -> "world hello"
s = "a good   example"  -> "example good a"
```

## How to explain it out loud

*"Two phases. First, extract the actual words into a vector, ignoring extra spaces entirely — scan through the string, build up a word character by character while I'm inside one, and the moment I hit a space (or reach the end of the string) with a non-empty word buffered, push it and reset the buffer. That naturally skips leading spaces, trailing spaces, and runs of multiple spaces, since I only ever push when there's an actual word to push. Second phase: walk the words vector back to front, joining with single spaces. The one subtlety is flushing the last word — if the string doesn't end in a space, the 'word just ended' check and the 'is this a space' check can't be the same conditional branch, because on the final character you need to flush regardless of whether it's a space or not."*

## Approach

**Extraction pass:** walk the string once. For each character: if it's not a space, append it to a running `word` buffer. Independently — not as an `else` — check whether the current character is a space *or* it's the last character of the string, and whether `word` is non-empty; if so, push `word` onto a `words` vector and clear the buffer. Checking both conditions independently (not `if / else if`) is what correctly flushes a word that runs all the way to the end of the string.

**Assembly pass:** walk `words` from the back to the front, appending each word to the result, with a single space between words (no trailing space after the last one).

Time: O(n) · Space: O(n) for the words vector and result

## Approach — In-place, O(1) extra space (follow-up / optimal)

LeetCode's follow-up asks for O(1) extra space, not counting the output. Three steps, all done in-place on the mutable string (works directly in C++; Java/Python strings are immutable, so this trick needs a `char[]`/`StringBuilder` there to get the same space benefit):

1. **Compact:** squeeze out leading, trailing, and repeated spaces using a write-pointer — same family of technique as Remove Duplicates from Sorted Array. Walk a `read` pointer forward; whenever it lands on the start of a word, copy that whole word to the `write` position (prefixing with a single space if it's not the first word), then resize the string down to `write` length.
2. **Reverse the whole compacted string.** This puts the words in the right final order, but scrambles each word's internal letters (e.g. `"the sky"` → `"yks eht"`).
3. **Reverse each individual word back to normal**, in place, by scanning for word boundaries (spaces) and reversing just that slice. `"yks eht"` → `"sky the"`.

Time: O(n) (each character touched a constant number of times) · Space: O(1) extra, in-place on the string

## Solution

### C++
```cpp
class Solution {
public:
    string reverseWords(string s) {
        vector<string> words;
        string word = "";
        for (int i = 0; i < s.length(); i++) {
            if (s[i] != ' ') {
                word += s[i];
            } else {
                if (word.length() != 0) {
                    words.push_back(word);
                    word = "";
                }
            }
        }
        // flush whatever's left in the buffer — handles the case where
        // the string doesn't end in a space, so the loop above never
        // gets a trailing space to trigger the push on the last word.
        if (word.length() != 0) {
            words.push_back(word);
        }

        string result = "";
        for (int i = words.size()-1; i >= 0; i--) {
            result += words[i];
            if (i != 0) {
                result += ' ';
            }
        }
        return result;
    }
};
```
Same fix as before (flush the last word unconditionally), restructured to be more explicit: `if / else` handles "am I building a word or hit a space," and a separate post-loop `if` flushes anything still buffered once the loop ends — rather than folding the end-of-string check into the same conditional that checks for spaces.

### C++ (In-place, O(1) extra space — optimal)
```cpp
class Solution {
public:
    string reverseWords(string s) {
        int n = s.length();

        // 1. compact: remove leading/trailing/multiple spaces, in place
        int write = 0, read = 0;
        while (read < n) {
            if (s[read] != ' ') {
                if (write != 0) s[write++] = ' ';  // separator, skip before the first word
                while (read < n && s[read] != ' ') {
                    s[write++] = s[read++];
                }
            } else {
                read++;
            }
        }
        s.resize(write);

        // 2. reverse the whole compacted string
        reverse(s.begin(), s.end());

        // 3. reverse each word back to normal, in place
        int i = 0, m = s.length();
        while (i < m) {
            int j = i;
            while (j < m && s[j] != ' ') j++;
            reverse(s.begin()+i, s.begin()+j);
            i = j + 1;
        }
        return s;
    }
};
```

### Python
```python
class Solution:
    def reverseWords(self, s: str) -> str:
        words = []
        word = ""
        for i in range(len(s)):
            if s[i] != ' ':
                word += s[i]
            if (s[i] == ' ' or i == len(s)-1) and len(word) != 0:
                words.append(word)
                word = ""
        return " ".join(reversed(words))
```

### Java
```java
class Solution {
    public String reverseWords(String s) {
        List<String> words = new ArrayList<>();
        StringBuilder word = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (ch != ' ') {
                word.append(ch);
            }
            if ((ch == ' ' || i == s.length()-1) && word.length() != 0) {
                words.add(word.toString());
                word.setLength(0);
            }
        }
        StringBuilder result = new StringBuilder();
        for (int i = words.size()-1; i >= 0; i--) {
            result.append(words.get(i));
            if (i != 0) result.append(' ');
        }
        return result.toString();
    }
}
```

Verified against 7 cases (classic, leading/trailing double spaces, multiple spaces between words, single word with no spaces, single character, single word wrapped in spaces, all-whitespace input) — C++ (both vector-based and in-place versions) and Python outputs match exactly on all 7. The in-place version additionally cross-checked against the vector version as ground truth on 9 cases (including nested multi-space cases like `"  Bob    Loves  Alice   "`) — all matched. Both C++ versions clean under AddressSanitizer. Java is the same logic as the vector-based version translated directly (not independently compiled in this environment — no JDK available); note Java/Python strings are immutable, so the in-place O(1)-space trick would need a `char[]`/`StringBuilder` there to get the same space benefit — it's a genuinely C++-specific advantage here since C++ `string` is mutable in place.

## Bug log

> ⚠️ **Real bug, caught by testing — dropped the final word whenever the string didn't end in a trailing space.**
>
> First version chained the two checks as `if (s[i] != ' ') {...} else if ((s[i]==' ' || i==len-1) && word.length()!=0) {...}`. Because they were `if / else if`, only one could fire per iteration. On the *last* character of a string like `"the sky is blue"`, `s[i]` is `'e'` — not a space — so the first branch always won, appending to `word`. The second branch (which contains the `i == s.length()-1` end-of-string check) never even got evaluated on that iteration, because it was gated behind the first branch being false. Result: the loop ended with `"blue"` sitting in the `word` buffer, never pushed into `words` — silently dropped from the output. Confirmed with concrete tests: `"the sky is blue"` returned `"is sky the"` (missing `blue`), and `"single"` / `"a"` returned empty strings entirely, since their one-and-only word never got flushed.
>
> **Fix:** make the two checks independent `if` statements (not `if / else if`), so appending to `word` and checking "should I flush now" both get evaluated on every character, including the last one — regardless of whether that last character happens to be a space.
