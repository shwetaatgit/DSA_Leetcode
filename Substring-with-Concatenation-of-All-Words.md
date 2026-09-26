# Substring with Concatenation of All Words (LeetCode Hard)

## Problem

Given a string `s` and an array `words` where every word has the **same length**, return the starting indices of all substrings in `s` that are a concatenation of *each word in `words` exactly once*, in **any order**.

```
s = "barfoothefoobarman", words = ["foo","bar"]        -> [0, 9]
s = "wordgoodgoodgoodbestword", words = ["word","good","best","word"] -> []
s = "barfoofoobarthefoobarman", words = ["bar","foo","the"] -> [6, 9, 12]
```

Constraints of note: `words.length` up to 5000 — this rules out any approach that generates permutations of `words` (that's `n!`, astronomically infeasible even at n=15, let alone 5000).

## How to explain it out loud

*"Every word has the same length, so any valid match is just `numWords` fixed-size chunks laid end to end, in some order. That turns this into Minimum Window Substring, but the 'unit' sliding through the window is a whole chunk instead of a single character. I build a `need` frequency map from `words`, then slide a window across `s` in `wordLen`-sized jumps, tracking a `window` map of chunks currently inside it and a count of how many total chunks are 'accounted for'. If a chunk isn't in `need` at all, the window it's in is poisoned — I hard-reset past it rather than trying to shrink one chunk at a time. If a chunk's count in the window exceeds what's needed, I shrink from the left in chunk-sized jumps until it doesn't. Once the count hits `numWords`, I've got an exact match at the window's left edge. The one twist beyond Minimum Window Substring: since I'm walking in `wordLen`-sized jumps, I have to repeat this whole scan `wordLen` times, once starting at each offset `0` through `wordLen-1` — otherwise I'd only ever see chunk boundaries aligned to one particular position and miss matches aligned differently."*

## Approach — Chunked sliding window, need/window frequency maps

Let `wordLen = words[0].length()`, `numWords = words.size()`, `totalLen = wordLen * numWords`. Build `need`: a frequency map counting occurrences of each string in `words` (words can repeat).

For each `offset` from `0` to `wordLen - 1`:
- Reset `window` (empty map), `count = 0`, `l = offset`.
- Walk `r` from `offset` to `n - wordLen` in steps of `wordLen`, reading `w = s.substr(r, wordLen)` each time.
  - If `w` is **not** in `need`: this chunk can never be part of any match. The entire window built up so far is invalid (it now contains a chunk that appears nowhere in `words`), so hard reset: clear `window`, `count = 0`, `l = r + wordLen` — don't try to shrink incrementally, just abandon everything up to and including this bad chunk.
  - If `w` **is** in `need`: add it to `window`, increment `count`. If `window[w]` now exceeds `need[w]` (too many copies of this particular word), shrink from the left in `wordLen` jumps — removing `s.substr(l, wordLen)` from `window`, decrementing `count`, advancing `l` by `wordLen` — until `window[w] <= need[w]` again.
  - If `count == numWords`, every chunk currently in the window is accounted for and none are surplus — this is an exact match. Record `l`. Then slide the window forward by removing the leftmost chunk (decrement `window`/`count`, advance `l` by `wordLen`) so the next iteration can look for the next match.

Why the offset loop is necessary: chunk boundaries only line up with `r`'s step size starting from wherever `r` began. Starting at `offset = 0` only ever inspects chunks at positions `0, wordLen, 2*wordLen, ...` — a valid match starting at position `1` (say) would never be examined, because its internal word boundaries don't align with that stepping. Running the scan once per starting offset (`0` through `wordLen - 1`) guarantees every possible alignment is covered, and each individual offset's scan is still a clean single pass over its slice of `s`.

Why a hard reset on a miss (not a shrink loop) is correct: with characters (Min Window Substring), a character not in `need` can just sit outside the window forever without corrupting anything — you shrink around it. Here, a chunk not in `need` sits *inside* whatever window currently spans it once `r` passes it, and there's no way to shrink `l` past it while keeping the window's span contiguous and chunk-aligned other than jumping `l` to just past that bad chunk — which is exactly a hard reset of the whole window.

Time: O(n × wordLen) — `wordLen` separate passes, each O(n / wordLen) chunks, each involving an O(wordLen) substring extraction and O(1) average map operations, so `wordLen` passes × `(n/wordLen)` chunks × `O(wordLen)` work per chunk = O(n × wordLen) total. Space: O(numWords × wordLen) for the `need`/`window` maps (bounded by the total length of `words`).

## Solution

### C++
```cpp
class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        vector<int> result;
        int wordLen = words[0].length();
        int numWords = words.size();
        int totalLen = wordLen * numWords;
        int n = s.length();
        if (n < totalLen) return result;

        unordered_map<string,int> need;
        for (auto& w : words) need[w]++;

        for (int offset = 0; offset < wordLen; offset++) {
            unordered_map<string,int> window;
            int count = 0;
            int l = offset;
            for (int r = offset; r + wordLen <= n; r += wordLen) {
                string w = s.substr(r, wordLen);
                if (need.find(w) != need.end()) {
                    window[w]++;
                    count++;
                    while (window[w] > need[w]) {
                        string leftWord = s.substr(l, wordLen);
                        window[leftWord]--;
                        count--;
                        l += wordLen;
                    }
                    if (count == numWords) {
                        result.push_back(l);
                        string leftWord = s.substr(l, wordLen);
                        window[leftWord]--;
                        count--;
                        l += wordLen;
                    }
                } else {
                    window.clear();
                    count = 0;
                    l = r + wordLen;
                }
            }
        }
        return result;
    }
};
```

### Python
```python
from collections import defaultdict

class Solution:
    def findSubstring(self, s: str, words: list[str]) -> list[int]:
        word_len = len(words[0])
        num_words = len(words)
        total_len = word_len * num_words
        n = len(s)
        if n < total_len:
            return []

        need = defaultdict(int)
        for w in words:
            need[w] += 1

        result = []
        for offset in range(word_len):
            window = defaultdict(int)
            count = 0
            l = offset
            r = offset
            while r + word_len <= n:
                w = s[r:r+word_len]
                if w in need:
                    window[w] += 1
                    count += 1
                    while window[w] > need[w]:
                        left_word = s[l:l+word_len]
                        window[left_word] -= 1
                        count -= 1
                        l += word_len
                    if count == num_words:
                        result.append(l)
                        left_word = s[l:l+word_len]
                        window[left_word] -= 1
                        count -= 1
                        l += word_len
                else:
                    window.clear()
                    count = 0
                    l = r + word_len
                r += word_len
        return result
```

### Java
```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        List<Integer> result = new ArrayList<>();
        int wordLen = words[0].length();
        int numWords = words.length;
        int totalLen = wordLen * numWords;
        int n = s.length();
        if (n < totalLen) return result;

        Map<String, Integer> need = new HashMap<>();
        for (String w : words) need.merge(w, 1, Integer::sum);

        for (int offset = 0; offset < wordLen; offset++) {
            Map<String, Integer> window = new HashMap<>();
            int count = 0;
            int l = offset;
            for (int r = offset; r + wordLen <= n; r += wordLen) {
                String w = s.substring(r, r + wordLen);
                if (need.containsKey(w)) {
                    window.merge(w, 1, Integer::sum);
                    count++;
                    while (window.get(w) > need.get(w)) {
                        String leftWord = s.substring(l, l + wordLen);
                        window.merge(leftWord, -1, Integer::sum);
                        count--;
                        l += wordLen;
                    }
                    if (count == numWords) {
                        result.add(l);
                        String leftWord = s.substring(l, l + wordLen);
                        window.merge(leftWord, -1, Integer::sum);
                        count--;
                        l += wordLen;
                    }
                } else {
                    window.clear();
                    count = 0;
                    l = r + wordLen;
                }
            }
        }
        return result;
    }
}
```

Verified: C++ against 6 hand-picked cases (both classic examples, a case with a repeated word across the whole string, a single-character single-word case, an all-overlapping-matches case, and a case needing a repeated word at a non-trivial offset) plus a 300-trial randomized stress test against a brute-force O(n × numWords) reference — 0 mismatches, clean under AddressSanitizer. Python independently re-verified against 3 of the hand-picked cases plus its own 300-trial randomized stress test — 0 mismatches. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- User's own first attempt (never completed) had a missing closing parenthesis (`if(s.substr(lc,lc+3) == words[i]{`), used undefined variables `lc`/`n`, and hardcoded the chunk length as `3` instead of `words[0].length()` — caught before compiling, not a runtime bug. More fundamentally, the approach (single pointer `lc`, decrementing frequency on a repeat, `l = lc`) had no mechanism to detect *which* word triggered an over-count or handle a chunk not present in `words` at all, so it wouldn't have converged to correct results even with the syntax fixed.
- Correct approach's first attempt: none — went straight from the need/window design (reusing the Minimum Window Substring pattern) to a version that passed the full test suite on the first run, including the offset loop and the hard-reset-on-miss detail, both identified as necessary *before* writing code rather than discovered by debugging a failure.
