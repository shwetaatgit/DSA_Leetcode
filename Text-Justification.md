# Text Justification (LeetCode Hard)

## Problem

Given an array of strings `words` and a width `maxWidth`, format the text so each line is exactly `maxWidth` characters, fully justified (both left and right aligned). Pack as many words per line as possible greedily. For a normal line, distribute extra spaces as evenly as possible between words — if it doesn't divide evenly, the leftmost gaps get the extra space. The last line, and any line with only one word, is left-justified with a single space between words and padded with trailing spaces to reach `maxWidth` (no extra inter-word spacing).

```
words = ["This", "is", "an", "example", "of", "text", "justification."]
maxWidth = 16

[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

## How to explain it out loud

*"This is mostly careful bookkeeping, not a clever trick. First, greedily figure out which words fit on each line: keep adding words while the sum of their lengths plus one mandatory space per gap still fits within maxWidth. Once a line's word range is decided, building the actual string has two cases. If it's the last line overall, or the line only has one word, left-justify it — join words with a single space and pad the end with spaces to reach maxWidth, no extra distribution. Otherwise, compute how many total extra spaces need distributing across the gaps: divide evenly (quotient), and whatever's left over (remainder) goes one-per-gap starting from the leftmost gaps. So gap number k gets quotient+1 spaces if k is within the remainder count, otherwise just quotient spaces."*

## Approach

**Line-grouping pass:** walk through `words` with an index `i`. For each line, start with `totalChar = len(words[i])` and try extending a pointer `j` forward. Word `j` can join the current line only if adding it still fits: `totalChar + len(words[j]) + (j - i) <= maxWidth`, where `(j - i)` is the number of gaps the line would need once word `j` is included (one word already occupies index `i`, so including up to index `j` means `j - i + 1` words and therefore `j - i` gaps). While that holds, absorb `words[j]` into `totalChar` and advance `j`. When the inner loop stops, the line covers `words[i..j-1]`; whether this is the last line is `j == n`.

**Line-building:** given a word range `[start, end]` (inclusive) and `totalChar` (sum of just those words' lengths, no spaces), let `gaps = end - start`.
- **Special case — last line, or a line with only one word (`gaps == 0`):** join the words with a single space each, then pad the end with spaces until the string reaches `maxWidth`.
- **Normal case:** the total padding needed is `maxWidth - totalChar`, spread across `gaps` gaps. `quotient = padding / gaps` is the baseline spaces every gap gets; `extra = padding % gaps` is how many gaps (counting from the left) get one additional space. For the `k`-th gap (1-indexed, `k = i - start` when appending `words[i]`), use `quotient + 1` spaces if `k <= extra`, else `quotient` spaces.

Time: O(total characters in `words`) — each word is examined a constant number of times · Space: O(total characters) for the output

## Solution

### C++
```cpp
class Solution {
public:
    string makeString(vector<string>& words, int start, int end, int mW, int totalChar, bool isLast) {
        int gaps = end - start;
        if (isLast || gaps == 0) {
            // left-justify: single space between words, pad right with spaces to reach mW
            string str = words[start];
            for (int i = start + 1; i <= end; i++) {
                str += ' ';
                str += words[i];
            }
            while ((int)str.length() < mW) str += ' ';
            return str;
        }

        string str = words[start];
        int quotient = (mW - totalChar) / gaps;
        int extra = (mW - totalChar) % gaps;
        for (int i = start + 1; i <= end; i++) {
            int gapIndex = i - start;  // 1-based gap number
            int space = (gapIndex <= extra) ? quotient + 1 : quotient;
            str += string(space, ' ');
            str += words[i];
        }
        return str;
    }

    vector<string> fullJustify(vector<string>& words, int maxWidth) {
        vector<string> result;
        int i = 0;
        int n = words.size();

        while (i < n) {
            int totalChar = words[i].length();
            int j = i + 1;
            while (j < n && totalChar + (int)words[j].length() + (j - i) <= maxWidth) {
                totalChar += words[j].length();
                j++;
            }
            bool isLast = (j == n);
            result.push_back(makeString(words, i, j - 1, maxWidth, totalChar, isLast));
            i = j;   // advance to the next line's starting word
        }
        return result;
    }
};
```

### C++ (alternate — `end` passed exclusive, i.e. `end = j` not `j-1`)
```cpp
class Solution {
public:
    vector<string> fullJustify(vector<string>& words, int maxWidth) {
        vector<string> result;

        int i=0;
        int n = words.size();
        bool isLast = false;

        while(i<n){
            int totalChar = words[i].length();
            int j = i+1;
            while(j<n && totalChar+ words[j].length()+ j-i <= maxWidth){
                totalChar += words[j].length();
                j++;
            }

            if(j==n) isLast = true;
            result.push_back(makeString(words, i, j, maxWidth, totalChar, isLast));
            i=j;
        }
        return result;
    }

    string makeString(vector<string>& words, int start, int end, int mW, int totalChar, bool isLast){
        string str = words[start];

        if(isLast || end-start ==1) { //either last word or only one word
            for (int i = start+1; i < end; i++) {
                str += ' ';
                str += words[i];
            }
            str += string((mW-str.length()), ' ');
            return str;
        }

        int gaps = end-start-1;
        int quotient = (mW-totalChar)/gaps;
        int extra = (mW-totalChar)%gaps;

        for(int i = start+1; i<end; i++){
            int gap_count = i-start;
            int space = (gap_count<=extra) ? quotient+1 : quotient;

            str += string(space, ' ');
            str += words[i];
        }
        return str;
    }
};
```
Same algorithm, different indexing convention: `end` here is passed as `j` (one *past* the last word on the line) rather than `j-1` (inclusive). That flips both reading loops to `i < end` instead of `i <= end`, and `end-start` directly gives the word count on the line (so the single-word check becomes `end-start == 1`, and `gaps = end-start-1`). Verified against the same 4 cases with identical output to the version above.

### Python
```python
class Solution:
    def makeString(self, words, start, end, mW, totalChar, isLast):
        gaps = end - start
        if isLast or gaps == 0:
            s = words[start]
            for i in range(start+1, end+1):
                s += ' ' + words[i]
            s += ' ' * (mW - len(s))
            return s

        s = words[start]
        quotient = (mW - totalChar) // gaps
        extra = (mW - totalChar) % gaps
        for i in range(start+1, end+1):
            gapIndex = i - start
            space = quotient + 1 if gapIndex <= extra else quotient
            s += ' ' * space + words[i]
        return s

    def fullJustify(self, words: List[str], maxWidth: int) -> List[str]:
        result = []
        i = 0
        n = len(words)
        while i < n:
            totalChar = len(words[i])
            j = i + 1
            while j < n and totalChar + len(words[j]) + (j - i) <= maxWidth:
                totalChar += len(words[j])
                j += 1
            isLast = (j == n)
            result.append(self.makeString(words, i, j-1, maxWidth, totalChar, isLast))
            i = j
        return result
```

### Java
```java
class Solution {
    private String makeString(List<String> words, int start, int end, int mW, int totalChar, boolean isLast) {
        int gaps = end - start;
        StringBuilder sb = new StringBuilder(words.get(start));
        if (isLast || gaps == 0) {
            for (int i = start + 1; i <= end; i++) {
                sb.append(' ').append(words.get(i));
            }
            while (sb.length() < mW) sb.append(' ');
            return sb.toString();
        }

        int quotient = (mW - totalChar) / gaps;
        int extra = (mW - totalChar) % gaps;
        for (int i = start + 1; i <= end; i++) {
            int gapIndex = i - start;
            int space = (gapIndex <= extra) ? quotient + 1 : quotient;
            for (int k = 0; k < space; k++) sb.append(' ');
            sb.append(words.get(i));
        }
        return sb.toString();
    }

    public List<String> fullJustify(String[] words, int maxWidth) {
        List<String> result = new ArrayList<>();
        int n = words.length;
        int i = 0;

        while (i < n) {
            int totalChar = words[i].length();
            int j = i + 1;
            while (j < n && totalChar + words[j].length() + (j - i) <= maxWidth) {
                totalChar += words[j].length();
                j++;
            }
            boolean isLast = (j == n);
            result.add(makeString(Arrays.asList(words), i, j - 1, maxWidth, totalChar, isLast));
            i = j;
        }
        return result;
    }
}
```

Verified against 4 cases (the classic example, a second standard multi-line example with an even-division gap case, a wider example spanning 6 lines including its own last-line, and a single-word line) — every output line's length checked to equal `maxWidth` exactly, and content cross-checked by hand against LeetCode's known expected output for the classic example. C++ clean under AddressSanitizer. Python matches C++ exactly on all 3 directly-compared tests. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- The version worked through interactively had several issues, all fixed together: the outer `while(i<n)` loop in `fullJustify` never advanced `i` (infinite loop) and had no `return result;`; the line-fitting condition `totalChar + (j-i) < maxWidth` never actually accounted for the candidate word's own length before deciding to include it, which could let a line's total content overflow past `maxWidth`; `makeString` referenced a variable `j` that didn't exist in its own scope (should have been the `end` parameter); the special case for the last line / single-word line was left empty and fell through into the general (gap-distribution) code below it, which divides by `gaps` — a division by zero for any single-word line; and the function had a duplicate `string str` declaration plus an incomplete, unfinished `str +=` statement. Rewritten with: `i = j` at the end of the outer loop; the fitting check corrected to `totalChar + len(words[j]) + (j-i) <= maxWidth`; `end` used consistently in place of `j`; the special case fully implemented with an early `return`, so the division-by-`gaps` code is never reached when `gaps == 0`.
