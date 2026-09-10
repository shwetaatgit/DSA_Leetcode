# Binary Tree Maximum Path Sum (LeetCode Hard)

## Problem

Given the root of a binary tree, return the maximum path sum of any path. A path is any sequence of nodes connected by parent-child edges; it does **not** need to pass through the root, and doesn't need to end at a leaf. A single node counts as a valid path.

```
    -10
    /  \
   9    20
       /  \
      15   7
-> 42   (path: 15 -> 20 -> 7)
```

## How to explain it out loud

*"The tricky part is that a path can bend once, at a single node — go up one child, through the node, down the other child — but it can't branch further than that, since it has to stay a simple path. So the recursive helper for each node needs to return two conceptually different things. First, the best path sum that continues upward through this node, usable by its parent to extend further — that can only go into at most one child, since going into both would create a branch. Second, separately, at every node I check what the best 'bent' path would be if it bent right here, using both children plus this node's own value — that's a candidate for the global answer, but it's not something the parent could extend, since a path can only bend once. So I track the global best in a variable that gets updated at every node as I check for a possible bend, while the function's actual return value is just the one-directional 'continues upward' sum. One more subtlety: if a child's best contribution is negative, it's better to not extend into that child at all — clamp negative contributions to zero before using them, since a path is always free to just stop."*

## Approach

Recursive helper, called on every node, that returns "the best path sum continuing upward through this node" (usable by the node's parent) while also updating a global running maximum (passed by reference, or held as instance/class state) that tracks "the best path anywhere, allowed to bend at this node."

At each node: recursively compute the best continuing sum from the left child and the right child. Clamp each to `max(0, ...)` — a negative contribution should never be included, since any path is free to simply not extend into a child that would drag the sum down. Update the global max with `leftSum + rightSum + root.val` (the candidate where the path bends at this exact node, using both children). Return `root.val + max(leftSum, rightSum)` upward — the best *single-direction* extension, since the parent can only continue the path through one child without branching.

The global max needs to be tracked separately from the return value, and updated at every node, because the actual best path in the tree might be a bent path fully contained in some subtree that never reaches the overall root — the return value alone (which only represents a straight-line path through the current node) would never capture that.

Time: O(n) — every node visited once · Space: O(h) for the recursion stack (`h` = tree height)

## Solution

### C++
```cpp
class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int maxSum = INT_MIN;
        findMaxSumNode(root, maxSum);
        return maxSum;
    }

    int findMaxSumNode(TreeNode* root, int &maxSum) {
        if (!root) return 0;

        int leftSum = max(0, findMaxSumNode(root->left, maxSum));
        int rightSum = max(0, findMaxSumNode(root->right, maxSum));
        maxSum = max(maxSum, leftSum + rightSum + root->val);

        return (root->val + max(leftSum, rightSum));
    }
};
```

### Python
```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        self.maxSum = float('-inf')
        self._helper(root)
        return self.maxSum

    def _helper(self, root):
        if not root:
            return 0
        leftSum = max(0, self._helper(root.left))
        rightSum = max(0, self._helper(root.right))
        self.maxSum = max(self.maxSum, leftSum + rightSum + root.val)
        return root.val + max(leftSum, rightSum)
```

### Java
```java
class Solution {
    private int maxSum;

    public int maxPathSum(TreeNode root) {
        maxSum = Integer.MIN_VALUE;
        findMaxSumNode(root);
        return maxSum;
    }

    private int findMaxSumNode(TreeNode root) {
        if (root == null) return 0;

        int leftSum = Math.max(0, findMaxSumNode(root.left));
        int rightSum = Math.max(0, findMaxSumNode(root.right));
        maxSum = Math.max(maxSum, leftSum + rightSum + root.val);

        return root.val + Math.max(leftSum, rightSum);
    }
}
```

Verified against 4 hand-picked cases (a simple positive tree, the classic example, a negative-child edge case where extending into a child would hurt, and a single negative-valued node) plus a 300-trial randomized stress test (random tree shapes, values ranging -10 to 10, so plenty of negative subtrees) against an independent brute-force reference — enumerates every pair of nodes, finds the unique path between them via their lowest common ancestor (tracked with parent pointers), and sums it directly, taking the max over all pairs including a node paired with itself — 0 mismatches. C++ clean under AddressSanitizer (leak detection excluded — disposable test-tree nodes aren't freed, unrelated to solution correctness). Python matches on all 4 hand-picked cases. Java is the same logic translated directly (not independently compiled in this environment — no JDK available), using an instance field for `maxSum` instead of C++'s reference parameter, since Java has no pass-by-reference for primitives.

## Bug log

- First attempt passed `maxSum` **by value** (`int maxSum`, not `int &maxSum`) into the recursive helper. Every recursive call received its own private copy; updates made deep in the recursion (`maxSum = max(maxSum, ...)`) only ever modified that call's local copy, discarded the instant the call returned. The `maxSum` variable in `maxPathSum` itself never changed from its initial `INT_MIN`. Confirmed concretely: both `maxPathSum` calls on real trees (including the classic example) returned `-2147483648` regardless of the actual tree. Fixed by changing the parameter to `int &maxSum`, a reference — updates anywhere in the recursion now correctly propagate back to the caller's variable.
- After that fix, a second real bug remained: `leftSum`/`rightSum` were used directly without clamping negative values to zero. Confirmed with a targeted case — `root=5` with a single child `left=-10` returned `-5` instead of the correct `5`, because the code unconditionally added the negative child contribution (`5 + (-10) = -5`) instead of recognizing that the best path here is just the root alone (a path can always choose not to extend into a child that would lower the sum). Fixed by wrapping each recursive call's result in `max(0, ...)` right where `leftSum`/`rightSum` are computed, so a negative-contributing child is treated as contributing nothing rather than actively hurting the sum. Verified afterward against 300 random trees with 0 mismatches.
