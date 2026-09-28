# Average of Levels in Binary Tree (LeetCode Easy)

## Problem

Given the root of a binary tree, return the average value of the nodes on each level, top to bottom.

```
      3
     / \
    9  20
       / \
     15   7

-> [3.0, 14.5, 11.0]
```

## How to explain it out loud

*"This is level-order BFS, but with a trick to know exactly where one level ends and the next begins: before draining any nodes for the current level, I snapshot the queue's current size — that's exactly how many nodes belong to this level, since every node currently in the queue was pushed by the *previous* level's processing, before any of the current level's children got pushed in. I sum those `size` nodes' values while also enqueueing their children, then divide by `size` for this level's average, and move on — the queue now holds exactly the next level's nodes and nothing else."*

## Approach — BFS, level-size snapshot

Standard BFS with a queue, seeded with `root`. Each iteration of the outer loop processes exactly one level: capture `size = q.size()` *before* touching the queue — this count is fixed for the level, even though the queue's size keeps changing as children get pushed inside the inner loop. Run the inner loop exactly `size` times: pop a node, add its value to a running `sum`, and push its non-null children (which become next level's nodes, appended after all of this level's remaining nodes — so they don't get processed prematurely in the same inner loop). After the inner loop, `sum / size` is this level's average.

Time: O(n) — every node is pushed and popped exactly once · Space: O(w) for the queue, where `w` is the widest level of the tree (worst case O(n) for a completely full/wide tree), plus O(n) for the output vector.

## Solution

### C++
```cpp
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        vector<double> result;
        if(root==NULL) return result;

        queue<TreeNode*> q;
        q.push(root);

        while(!q.empty()){
            int size = q.size();
            double sum = 0;
            for(int i=0; i<size; i++){
                TreeNode* node = q.front();
                q.pop();
                sum+=node->val;
                if(node->left) q.push(node->left);
                if(node->right) q.push(node->right);
            }
            result.push_back(sum/size);
        }
        return result;
    }
};
```

### Python
```python
from collections import deque

class Solution:
    def averageOfLevels(self, root: Optional[TreeNode]) -> list[float]:
        result = []
        if root is None:
            return result

        q = deque([root])
        while q:
            size = len(q)
            total = 0
            for _ in range(size):
                node = q.popleft()
                total += node.val
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            result.append(total / size)

        return result
```

### Java
```java
class Solution {
    public List<Double> averageOfLevels(TreeNode root) {
        List<Double> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);

        while (!q.isEmpty()) {
            int size = q.size();
            double sum = 0;
            for (int i = 0; i < size; i++) {
                TreeNode node = q.poll();
                sum += node.val;
                if (node.left != null) q.offer(node.left);
                if (node.right != null) q.offer(node.right);
            }
            result.add(sum / size);
        }
        return result;
    }
}
```

Verified against 6 hand-picked cases: the classic LeetCode example both with and without explicit null markers in the level-order build, a single-node tree, an empty tree, a tree with all-negative values, and a tree with three large-magnitude values (`2×10^9` each) at LeetCode's stated `int` value extremes, confirming `double` accumulation stays exact at that scale — all correct, clean under AddressSanitizer + UndefinedBehaviorSanitizer. Python and Java are the same level-size-snapshot BFS translated directly (Java not independently compiled in this environment — no JDK available).

## Bug log

- Correct on the first attempt — no bugs found.
