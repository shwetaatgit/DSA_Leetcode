# Invert Binary Tree (LeetCode Easy)

## Problem

Given the root of a binary tree, invert it — swap every node's left and right children, recursively across the entire tree.

```
    4                4
   / \              / \
  2   7    -->     7   2
 / \ / \          / \ / \
1  3 6  9        9  6 3  1
```

## How to explain it out loud

*"Classic recursion. At every node, swap its left and right children. Then recursively do the same thing to whatever is now in the left subtree, and whatever is now in the right subtree — the recursion handles every level automatically. Base case: a null node has nothing to invert, so just return it as-is. Doesn't matter whether you swap before or after recursing into the children, as long as you're consistent, since each node's own swap is independent of what happens deeper in the tree."*

## Approach

If the current node is `null`, there's nothing to invert — return it directly (base case). Otherwise, swap the node's `left` and `right` pointers, then recursively call `invertTree` on the (now-swapped) left child and the (now-swapped) right child. Return the node itself, now the root of the inverted subtree.

Time: O(n) — every node visited exactly once · Space: O(h) for the recursion stack, where `h` is the tree's height (O(log n) for a balanced tree, O(n) worst case for a completely skewed one)

## Solution

### C++
```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (root == NULL) return root;

        TreeNode* temp = root->left;
        root->left = root->right;
        root->right = temp;

        invertTree(root->left);
        invertTree(root->right);
        return root;
    }
};
```

### Python
```python
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if root is None:
            return root
        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
        return root
```

### Java
```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return root;

        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;

        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
}
```

Verified against 4 cases (the classic example, a small 3-node tree, an empty tree, a single-node tree) by building each tree from a level-order input, inverting, and comparing the resulting level-order traversal against the expected inverted shape — C++ and Python outputs match exactly on all 4, C++ additionally clean under AddressSanitizer (leak detection excluded, since the disposable test harness intentionally doesn't free tree nodes — unrelated to the solution's correctness). Java is the same logic translated directly (not independently compiled in this environment — no JDK available), no language-specific behavior involved.

## Bug log

- None — correct on the first attempt.
