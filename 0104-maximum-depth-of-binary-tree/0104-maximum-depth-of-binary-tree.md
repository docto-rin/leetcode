## Step 1

- 問題文
  - 2分木の根`root`を与えられ、その最大の深さを返す問題。
  - The number of nodes in the tree is in the range [0, 104].

### 実装1

- アルゴリズムの選択
  - BFS風にdepth-wiseで子が尽きないか見ていく方針でよさそう。
- 実装の方針
  - 二重ループで書く。
  - 同じdepth内での探索順序は順不同なので、list型（stack, LIFO）で十分だろう。
  - 木構造は同じnodeを重複してキューに入れる心配がないので、visitedのような変数は不要。
  - 1回目にif root is None: return 0を書き忘れてしまった。入力に対するチェックを怠ってしまった。

```python3
from typing import Optional


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(node, stack):
            if node is not None:
                stack.append(node)
        
        depth = 0
        frontiers = []
        append_if_exists(root, frontiers)
        while frontiers:
            depth += 1
            next_frontiers = []
            for node in frontiers:
                append_if_exists(node.left, next_frontiers)
                append_if_exists(node.right, next_frontiers)
            frontiers = next_frontiers
        
        return depth
```

ここまで5分。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.27jfjzhov3la)
  - https://discord.com/channels/1084280443945353267/1226508154833993788/1246313673446653963
    - > あー、ちょっと全ノードに部下を立たせてみましょう。
    - コードを書いた方は帰りがけでmax_depthを完成させる方法が発想できず苦戦したということだと思いますが、逆に行きがけでmax_depthを作ってしまうコードが面白いと感じました。
    - レビュワーの論点としては、nonlocal depthは不要ということですかね。それはたしかに、行きがけ、帰りがけどちらでmax_depthを完成させるにしても不要に見えます。

## Step 3

### 実装2

やっぱり、dequeを使った方が省メモリに実装できることに気づき、書き直しました。

```python3
from collections import deque


class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def append_if_exists(node, nodes):
            if node is not None:
                nodes.append(node)
        
        depth = 0
        frontiers = deque()
        append_if_exists(root, frontiers)
        while frontiers:
            depth += 1
            depth_size = len(frontiers)
            for _ in range(depth_size):
                node = frontiers.popleft()
                append_if_exists(node.left, frontiers)
                append_if_exists(node.right, frontiers)
        
        return depth
```
