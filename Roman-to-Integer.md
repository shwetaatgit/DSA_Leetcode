# Roman to Integer (LeetCode Easy)

## Problem

Convert a Roman numeral string to an integer. Symbols: `I=1, V=5, X=10, L=50, C=100, D=500, M=1000`. Normally left-to-right addition, but six subtractive pairs exist where a smaller symbol before a larger one means subtract: `IV=4, IX=9, XL=40, XC=90, CD=400, CM=900`.

```
"LVIII" -> 58    (L=50, V=5, III=3)
"MCMXCIV" -> 1994
```

## How to explain it out loud

*"Scan from the back. Start with the value of the last symbol. Moving right to left, compare each symbol's value to the one immediately after it — the one I've already folded in. If the current symbol's value is greater than or equal to the next one, add it; that's the normal case, including repeated symbols like the three I's in III. If it's strictly less, subtract it — that's the subtractive pair, like the I in IV. One pass, one map, O(n) time."*

The `>=` instead of `>` matters: repeated identical symbols (III, XX, MMM) must always add, and treating "equal" as "not greater" would wrongly send them down the subtract path.

## Approach

Map each symbol to its value. Walk the string right to left, keeping a running total that starts as the value of the last character. For each earlier character, compare its value to the one right after it (already processed): `>=` means add, `<` means it's a subtractive prefix, subtract.

Time: O(n) · Space: O(1) (map has a fixed 7 entries)

## Solution

### C++
```cpp
class Solution {
public:
    int romanToInt(string s) {
        unordered_map<char,int> m = {{'I',1}, {'V',5}, {'X',10}, {'L',50}, {'C',100}, {'D',500}, {'M',1000}};
        int result = m[s[s.length()-1]];
        for (int i = s.length()-2; i >= 0; i--) {
            if (m[s[i]] >= m[s[i+1]]) result += m[s[i]];
            else result -= m[s[i]];
        }
        return result;
    }
};
```

### Python
```python
class Solution:
    def romanToInt(self, s: str) -> int:
        m = {'I':1,'V':5,'X':10,'L':50,'C':100,'D':500,'M':1000}
        result = m[s[-1]]
        for i in range(len(s)-2, -1, -1):
            if m[s[i]] >= m[s[i+1]]:
                result += m[s[i]]
            else:
                result -= m[s[i]]
        return result
```

### Java
```java
class Solution {
    public int romanToInt(String s) {
        Map<Character,Integer> m = new HashMap<>();
        m.put('I',1); m.put('V',5); m.put('X',10); m.put('L',50);
        m.put('C',100); m.put('D',500); m.put('M',1000);
        int result = m.get(s.charAt(s.length()-1));
        for (int i = s.length()-2; i >= 0; i--) {
            if (m.get(s.charAt(i)) >= m.get(s.charAt(i+1))) result += m.get(s.charAt(i));
            else result -= m.get(s.charAt(i));
        }
        return result;
    }
}
```

Verified against 6 cases (repeats, single subtractive pair, two-digit, mixed, full-year style, max valid Roman numeral 3999) — C++ additionally clean under AddressSanitizer.

## Bug log

- `sum` used throughout the loop and in `return`, but the variable was actually declared as `result` — undeclared-variable compile error.
- `s.lenth()` typo for `s.length()`.
- Comparison `m[s[i]] > m[s[i+1]]` (strict) sent equal-valued adjacent symbols down the subtract branch. Repeated symbols (`III`, `XX`) are never subtractive — needed `>=` so "equal" falls into "add." Caught with `"III"` returning `-1` instead of `3`.
