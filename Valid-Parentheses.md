# Valid Parentheses (LeetCode Easy)

## Problem

Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the input string is valid — every open bracket must be closed by the same type of bracket, in the correct order.

```
s = "()[]{}"  -> true
s = "(]"      -> false
s = "([)]"    -> false
s = "{[]}"    -> true
```

## How to explain it out loud

*"A stack is the natural fit because of the 'most recently opened must be closed first' rule — that's exactly LIFO behavior. Walk through the string: every time I see an opening bracket, push it. Every time I see a closing bracket, it must match whatever's currently on top of the stack — if the stack is empty, there's nothing to match, so it's invalid; if the top doesn't match the closing bracket's type, it's invalid; otherwise pop it off, since that pair is now resolved. At the end, the string is valid only if the stack is completely empty — any leftover unclosed opening brackets make it invalid too."*

## Approach

Use a stack of characters. Walk through `s` once:
- If the character is an opening bracket (`(`, `{`, `[`), push it.
- If it's a closing bracket, first check the stack isn't empty (an empty stack means there's nothing to match against — automatically invalid), then check the top of the stack is the matching opening bracket. If either check fails, return `false`. Otherwise pop the matched opening bracket off.

After processing the whole string, the result is valid if and only if the stack is empty — any brackets still sitting on the stack were opened but never closed.

Time: O(n) · Space: O(n) worst case (all opening brackets)

## Solution

### C++
```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> st;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == ')') {
                if (st.size() == 0 || st.top() != '(') return false;
                else st.pop();
            }
            else if (s[i] == '}') {
                if (st.size() == 0 || st.top() != '{') return false;
                else st.pop();
            }
            else if (s[i] == ']') {
                if (st.size() == 0 || st.top() != '[') return false;
                else st.pop();
            }
            else {
                st.push(s[i]);
            }
        }

        return !(st.size() > 0);
    }
};
```

### Python
```python
class Solution:
    def isValid(self, s: str) -> bool:
        st = []
        pairs = {')': '(', '}': '{', ']': '['}
        for ch in s:
            if ch in pairs:
                if len(st) == 0 or st[-1] != pairs[ch]:
                    return False
                st.pop()
            else:
                st.append(ch)
        return len(st) == 0
```

### Java
```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> st = new ArrayDeque<>();
        Map<Character, Character> pairs = Map.of(')', '(', '}', '{', ']', '[');
        for (char ch : s.toCharArray()) {
            if (pairs.containsKey(ch)) {
                if (st.isEmpty() || st.peek() != pairs.get(ch)) return false;
                st.pop();
            } else {
                st.push(ch);
            }
        }
        return st.isEmpty();
    }
}
```

Verified against 9 hand-picked cases (all-matched pairs of every bracket type, mismatched types, wrong nesting order, correctly nested, a lone closing bracket, a lone opening bracket, empty string, all-unclosed opens, repeated matched pairs) plus a 3000-trial randomized stress test against a reference implementation — 0 mismatches. C++ clean under AddressSanitizer and hardened libstdc++ assertions (`-D_GLIBCXX_ASSERTIONS`). Python matches on all 9 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), using `Deque<Character>` as a stack since Java has no built-in `stack<char>` equivalent with O(1) peek/pop at the front.

## Bug log

- First attempt called `st.top()` in each closing-bracket branch **without first checking whether the stack was empty**. `.top()` on an empty `std::stack` is undefined behavior. Confirmed as a real crash: with hardened assertions enabled (`-D_GLIBCXX_ASSERTIONS`), `isValid(")")` aborted with `Assertion '!this->empty()' failed` — any input with a closing bracket that has no matching open before it (starts with a closer, or simply has more closes than opens at some point) triggered this. Fixed by adding `st.size() == 0 ||` as the first condition in each closing-bracket check, short-circuiting before `.top()` is ever called on an empty stack.
- Second attempt (fixing the above) introduced a **typo/compile error**: in the `)` branch, wrote `st.pop() != '('` instead of `st.top() != '('`. In C++, `std::stack::pop()` returns `void` — it only removes the top element, it doesn't return it (unlike Python's `list.pop()` or Java's `Deque.pop()`, which do return the removed value). Comparing a `void` result to `'('` is a compile error (`invalid operands of types 'void' and 'char'`). Fixed by using `.top()` to peek (no removal) for the comparison, keeping `.pop()` as the separate call that actually removes the element once the match is confirmed — matching the pattern already correctly used in the `}` and `]` branches.
