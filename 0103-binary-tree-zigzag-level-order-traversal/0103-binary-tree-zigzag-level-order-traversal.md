## Step 1

- 問題文
  - 二分木の根`root`を与えられ、ノードの値を"ジグザグ"レベル順に走査して返す問題。(i.e., from left to right, then right to left for the next level and alternate between).
  - 制約:
    - The number of nodes in the tree is in the range [0, 2000].
    - -100 <= Node.val <= 100

### 実装1

- アルゴリズムの選択
  - BFS/DFS。素直にBFSで。
  - [前回の問題](https://leetcode.com/problems/binary-tree-level-order-traversal/)とほとんど同じように書ける。
  - 二重ループを使う。
  - levelごとのvaluesは、level内では常にleft to rightで一旦貯めていき、複数レベルのリストに格納する直前に反転するかどうか決めたらいい。
- 実装
  - level変数を外側のループが回るごとのインクリメントする。
  - 1-indexed（rootのlevelが1）の方がわかりやすいと思い、そうする。
  - if level % 2 == 0: 反転、とする。
- 計算量
  - Time: O(n)
  - Space: O(n) 

```python3
from typing import Optional, List


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        values_ordered_by_level = []
        level_nodes = [root]
        level = 1  # root level is 1
        while level_nodes:
            level_values = []
            next_nodes = []
            
            for node in level_nodes:
                level_values.append(node.val)
                if node.left is not None:
                    next_nodes.append(node.left)
                if node.right is not None:
                    next_nodes.append(node.right)
            
            if level % 2 == 0:
                # zigzag level
                level_values.reverse()
            
            values_ordered_by_level.append(level_values)
            level_nodes = next_nodes
            level += 1
        
        return values_ordered_by_level
```

ここまで8分。

### 実装2

- DFSでも書いてみる。
- こちらもzigzagにするのは後処理的にやるのがシンプルに思えた。

```python3
from typing import Optional, List


class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        values_ordered_by_level = []
        nodes_with_level = [(root, 1)]
        while nodes_with_level:
            node, level = nodes_with_level.pop()
            
            while level > len(values_ordered_by_level):
                values_ordered_by_level.append([])
            values_ordered_by_level[level - 1].append(node.val)

            # LIFO: popped from left to right
            if node.right is not None:
                nodes_with_level.append((node.right, level + 1))
            if node.left is not None:
                nodes_with_level.append((node.left, level + 1))
        
        for level in range(1, len(values_ordered_by_level) + 1):
            if level % 2 == 0:
                values_ordered_by_level[level - 1].reverse()
        
        return values_ordered_by_level
```

## Step 2

- レビュー by GPT-5
  - > 反転自体は問題ありませんが、反転を避けたい場合は「収集中に `appendleft` を使う」手があります。具体的には偶数レベルでは `appendleft(node.val)`、奇数レベルでは `append(node.val)` として `deque` に詰め、最後に `list(...)` 化します。これは「コピー＋反転」を避けられます。
    - これは良さそう。O(n)の定数倍の最適化になる。
    - ところで、sequence.reverse()について調べていたところ、以下のように書いてあり興味深かったです。
      - > Reverse the items of sequence in place. This method maintains economy of space when reversing a large sequence. **To remind users that it operates by side-effect, it returns None.**
      - https://docs.python.org/ja/3.13/library/stdtypes.html#sequence.reverse
- https://github.com/nanae772/leetcode-arai60/pull/27
  - > levelの偶奇で判断するのは要件から離れていると感じるので、levelが上がるにつれてis_left_to_rightのbool値を反転させていく方が好みです。
    - 確かに。実装するときも、剰余が0 or 1で迷ったので読み手からしてもわかりくい（＝パズルになっている）だろう。
    - 情報量としてはフラグで十分なので、levelの偶奇で判断する実装するということは、levelによる分岐の拡張性を残したいという意志である。
- https://github.com/garunitule/coding_practice/pull/27
  - > 当時はどちらかというとreverseするための計算量O(N)を節約できるからdequeよさそう、と思っていたのですが、今改めて考えると、おっしゃる通り結局listに変換する際にコピーをするのでその分結局O(N)発生してしまっていて、あまりdequeにすることの旨みはなかったかもしれません。。
    - よく考えてみると、reverseの有無によらずlist()でO(N)の計算が入るので、逆に遅くなるまでありそう。
    - 例えばリストで頑張るなら、最初にそのレベルの全ノードのプレイスホルダーを作っておいて、zigzagする場合は後ろからインデックスアクセスで詰めていけば若干速いかもしれない。
      - 可読性が下がるが、まぁ許容できる範疇と考えた。（不要なパズルは生じないと思う。）

## Step 3

### 実装3

[実装1](#実装1)を改善する。

```python3
from typing import Optional, List


class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        level_order_values = []
        level_nodes = [root]
        is_left_to_right = True
        while level_nodes:
            level_values = [0] * len(level_nodes)
            next_nodes = []

            for i, node in enumerate(level_nodes):
                if is_left_to_right:
                    index = i
                else:
                    index = len(level_nodes) - 1 - i
                level_values[index] = node.val
                if node.left is not None:
                    next_nodes.append(node.left)
                if node.right is not None:
                    next_nodes.append(node.right)
            
            level_order_values.append(level_values)
            level_nodes = next_nodes
            is_left_to_right = not is_left_to_right
        
        return level_order_values
```
