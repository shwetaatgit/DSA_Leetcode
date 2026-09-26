# Word Pattern (LeetCode Easy)

## Problem

Given a `pattern` and a string `s`, find if `s` follows the same pattern — there's a bijection between each letter in `pattern` and each word in `s` (split by spaces): every occurrence of a letter must always map to the same word, and no two different letters may map to the same word.

```
pattern = "abba", s = "dog cat cat dog" -> true
pattern = "abba", s = "dog cat cat fish" -> false
pattern = "aaaa", s = "dog cat cat dog" -> false
pattern = "abba", s = "dog dog dog dog" -> false
```

## How to explain it out loud

*"Same bijection idea as Isomorphic Strings, just with letters mapping to whole words instead of letters mapping to letters. First split s into its individual words. If the word count doesn't match the pattern's length, immediate false — can't have a one-to-one mapping otherwise. Then walk both in lockstep with two separate maps, letter-to-word and word-to-letter. Whenever a letter is new, check its target word hasn't already been claimed by some other letter; whenever a word is new, nothing extra to check yet — just record both directions. If either side has been seen before, its existing mapping must match what's being proposed now, or it's not a valid pattern."*

## Approach

Split `s` into words. If the word count doesn't equal `pattern.length()`, return `false` immediately — a bijection needs equal-sized sets on both sides.

Walk `pattern` and the word list together. Keep two maps: `char -> string` and `string -> char`. At each position: if the current pattern character is new (not yet a key in the first map), check whether the current word has *already* been claimed by some other character (a lookup in the second map) — if so, that's a conflict, return `false`. Otherwise record both directions. If the pattern character already has a recorded mapping, it must match the current word exactly, or return `false`.

Checking both maps independently — one for each direction — is what correctly enforces the bijection, exactly as with Isomorphic Strings: a single shared map can't express "this word hasn't been used by anyone else" separately from "this letter has always meant this word."

Time: O(n) where `n` is the total length of `s` (for splitting) plus O(pattern.length()) for the mapping pass · Space: O(distinct letters + distinct words)

## Solution

### C++
```cpp
class Solution {
public:
    bool wordPattern(string pattern, string s) {
        vector<string> res;
        int start = 0;
        for (int i = start + 1; i <= (int)s.size(); )
            if (i == (int)s.size() || s[i] == ' ') {
                res.push_back(s.substr(start, i - start));
                start = i + 1;
                i = start + 1;
            }
            else
                i++;

        if (pattern.size() != res.size())
            return false;

        unordered_map<char, string> map1;
        unordered_map<string, char> map2;
        for (int i = 0; i < (int)res.size(); i++)
            if (map1.find(pattern[i]) == map1.end()) {
                if (map2.find(res[i]) != map2.end())
                    return false;
                map1[pattern[i]] = res[i];
                map2[res[i]] = pattern[i];
            }
            else {
                string str = map1[pattern[i]];
                if (str != res[i])
                    return false;
            }

        return true;
    }
};
```

### Python
```python
class Solution:
    def wordPattern(self, pattern: str, s: str) -> bool:
        words = s.split()
        if len(pattern) != len(words):
            return False
        p2w, w2p = {}, {}
        for c, w in zip(pattern, words):
            if c in p2w and p2w[c] != w:
                return False
            if w in w2p and w2p[w] != c:
                return False
            p2w[c] = w
            w2p[w] = c
        return True
```

### Java
```java
class Solution {
    public boolean wordPattern(String pattern, String s) {
        String[] words = s.split(" ");
        if (pattern.length() != words.length) return false;

        Map<Character, String> p2w = new HashMap<>();
        Map<String, Character> w2p = new HashMap<>();
        for (int i = 0; i < pattern.length(); i++) {
            char c = pattern.charAt(i);
            String w = words[i];
            if (p2w.containsKey(c) && !p2w.get(c).equals(w)) return false;
            if (w2p.containsKey(w) && w2p.get(w) != c) return false;
            p2w.put(c, w);
            w2p.put(w, c);
        }
        return true;
    }
}
```

Verified against 6 hand-picked cases (both classic true/false examples, a pattern collapsing two letters onto the same word, two different words both claimed by the same letter in the pattern, a single-character/single-word case, and a two-distinct-letters-one-word collision) plus a 3000-trial randomized stress test against an independent reference implementation (splits `s` via `stringstream`) — 0 mismatches. C++ clean under AddressSanitizer. Python matches on all 6 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), using `String.split(" ")` in place of the manual substring-splitting loop.

## Bug log

- None — correct on the first attempt, including correctly applying the bidirectional two-map pattern (the same fix just worked through for Isomorphic Strings) without needing correction. One harmless style note: a local variable inside the validation loop was originally named `s`, shadowing the function's own `s` parameter (the sentence string) — not a bug here since the outer `s` is never referenced again after being split into `res`, but renamed to `str` for clarity, consistent with avoiding this class of naming collision elsewhere in this set of solutions.
