# Integer to Roman (LeetCode Medium)

## Problem

Convert an integer (`1` to `3999`) to a Roman numeral string.

```
3    -> "III"
58   -> "LVIII"     (L=50, V=5, III=3)
1994 -> "MCMXCIV"   (M=1000, CM=900, XC=90, IV=4)
```

## How to explain it out loud

*"Two ways to think about this. One: since the value is capped at 3999, I can handle each decimal digit — thousands, hundreds, tens, units — independently, because Roman numerals for 1-9 in any given place value never interact with other place values. So I precompute the Roman representation for every digit 0-9 in each place (hundreds: '', C, CC, ..., CM; tens: '', X, XX, ..., XC; units similarly), index into those tables using num/1000, (num%1000)/100, (num%100)/10, num%10, and concatenate. O(1) work, four lookups. Two: the more general greedy approach — list every value/symbol pair, including the subtractive ones like 900→CM and 40→XL, ordered largest to smallest. Repeatedly subtract the biggest value that still fits into num, appending its symbol, until num hits zero. That's the version that generalizes to numbering systems without a fixed digit cap."*

## Approach — Digit lookup tables (optimal for this problem's bounded range)

Since `num <= 3999`, break it into decimal digits and handle each place value independently — Roman numeral symbols for one place (thousands, hundreds, tens, units) never combine with symbols from another place. Precompute a small array of strings for each place value, covering digits `0-9` (thousands only needs `0-3`, since `num <= 3999`), including the subtractive forms (e.g., hundreds place: `["", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"]`). Index into each table with `num/1000`, `(num%1000)/100`, `(num%100)/10`, `num%10`, and concatenate the four results.

Time: O(1) (bounded work regardless of input) · Space: O(1)

## Approach — Greedy value/symbol pairs (general-purpose)

Build one list of `(value, symbol)` pairs ordered largest to smallest, including every subtractive combination: `1000/M, 900/CM, 500/D, 400/CD, 100/C, 90/XC, 50/L, 40/XL, 10/X, 9/IX, 5/V, 4/IV, 1/I`. Walk the list in order; for each pair, while `num >= value`, append `symbol` to the answer and subtract `value` from `num`. Because the list is sorted descending and covers every subtractive case explicitly, greedily taking the largest fitting value at each step always produces the correct, unique minimal-length Roman numeral.

Time: O(1) (at most ~13 symbol groups, each appended a bounded number of times) · Space: O(1) extra (excluding output)

## Solution

### C++ (Digit lookup — optimal)
```cpp
class Solution {
public:
    string intToRoman(int num) {
        string m[]={"","M","MM","MMM"};
        string c[]={"","C","CC","CCC","CD","D","DC","DCC","DCCC","CM"};
        string x[]={"","X","XX","XXX","XL","L","LX","LXX","LXXX","XC"};
        string i[]={"","I","II","III","IV","V","VI","VII","VIII","IX"};
        string thousands = m[num/1000];
        string hundreds = c[(num%1000)/100];
        string tens = x[(num%100)/10];
        string units = i[num%10];
        return thousands+hundreds+tens+units;
    }
};
```

### C++ (Greedy — general)
```cpp
class Solution {
public:
    string intToRoman(int num) {
        vector<pair<int,string>> vals = {
            {1000,"M"},{900,"CM"},{500,"D"},{400,"CD"},
            {100,"C"},{90,"XC"},{50,"L"},{40,"XL"},
            {10,"X"},{9,"IX"},{5,"V"},{4,"IV"},{1,"I"}
        };
        string ans = "";
        for (int i = 0; i < vals.size(); i++) {
            while (num >= vals[i].first) {
                ans += vals[i].second;
                num -= vals[i].first;
            }
        }
        return ans;
    }
};
```
Index-based version, equivalent to the range-based-for form — `vals[i]` gives a `pair<int,string>` object, so access its members with `.first`/`.second`, not `->` (that's for pointers/iterators). Loop bound is `vals.size()` (the number of value/symbol pairs), not `num` — `num` is just an `int`, it has no `.length()`.

### Python (both)
```python
class Solution:
    def intToRoman_lookup(self, num: int) -> str:
        m = ["","M","MM","MMM"]
        c = ["","C","CC","CCC","CD","D","DC","DCC","DCCC","CM"]
        x = ["","X","XX","XXX","XL","L","LX","LXX","LXXX","XC"]
        i = ["","I","II","III","IV","V","VI","VII","VIII","IX"]
        return m[num//1000] + c[(num%1000)//100] + x[(num%100)//10] + i[num%10]

    def intToRoman(self, num: int) -> str:
        vals = [
            (1000,"M"),(900,"CM"),(500,"D"),(400,"CD"),
            (100,"C"),(90,"XC"),(50,"L"),(40,"XL"),
            (10,"X"),(9,"IX"),(5,"V"),(4,"IV"),(1,"I")
        ]
        ans = []
        for val, sym in vals:
            while num >= val:
                ans.append(sym)
                num -= val
        return "".join(ans)
```

### Java (Greedy)
```java
class Solution {
    public String intToRoman(int num) {
        int[] values = {1000,900,500,400,100,90,50,40,10,9,5,4,1};
        String[] symbols = {"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
        StringBuilder ans = new StringBuilder();
        for (int idx = 0; idx < values.length; idx++) {
            while (num >= values[idx]) {
                ans.append(symbols[idx]);
                num -= values[idx];
            }
        }
        return ans.toString();
    }
}
```

Both approaches verified against 14 cases (every subtractive form: 4, 9, 40, 90, 400, 900; boundaries 1, 1000, 3999; and mixed values 58, 1994, 444, 2024, 3) — C++ both versions and Python both versions match exactly on all 14, C++ additionally clean under AddressSanitizer. Java (greedy) is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — user's original digit-lookup solution was correct on first attempt (verified against all 14 test cases including every subtractive edge case). Greedy version added afterward as the general-purpose alternative, also correct on first attempt.
