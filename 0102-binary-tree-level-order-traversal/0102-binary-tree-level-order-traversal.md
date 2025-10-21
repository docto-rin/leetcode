## Step 1

- 問題文
  - 二分木の根`root`が与えられ、レベル順に走査したときのノードたちの値を返す。
  - 例：
    - Input: root = [3,9,20,null,null,15,7]
    - Output: [[3],[9,20],[15,7]]
  - 制約：
    - The number of nodes in the tree is in the range [0, 2000].
    - -1000 <= Node.val <= 1000

### 実装1

- アルゴリズムの選択
  - BFSしか思いつかない。
- 実装
  - なぜか一瞬levelごとに吐くgeneratorを書こうと思ったが、必要がないのでやめた。
  - levelごとのループと、level内でのnodeループの2重ループで書く。
  - キューにはNoneでないことを確かめていれる方が経験で見通しよく書けるので、そうする。
- 計算量
  - Time: O(V + E) = O(V)
  - Space: O(width) = O(V)

```python3
from typing import Optional, List


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        level_by_level = []
        frontiers = [root]
        while frontiers:
            level_by_level.append([])
            next_frontiers = []
            for node in frontiers:
                level_by_level[-1].append(node.val)
                if node.left is not None:
                    next_frontiers.append(node.left)
                if node.right is not None:
                    next_frontiers.append(node.right)
            frontiers = next_frontiers
        return level_by_level
```

ここまで6分。

### 実装2

- 実装
  - 実装1で、Noneチェックをキューから取り出すときにやる実装も書いてみる。
  - メリットはNoneチェックを1箇所で書けることや、root is Noneを特別扱いしなくて良い。
  - デメリットとしては、whileループが常に(深さ + 1)回ることや、それに伴う後処理が直感的でない（一種のパズルになっている）こと。
  - 個人的には実装1のが素直で好み、可読性も高いと感じる。

```python3
from typing import Optional, List


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        level_by_level = []
        frontiers = [root]
        while True:
            level_by_level.append([])
            next_frontiers = []
            for node in frontiers:
                if node is None:
                    continue
                level_by_level[-1].append(node.val)
                next_frontiers.append(node.left)
                next_frontiers.append(node.right)
            frontiers = next_frontiers
            if not level_by_level[-1]:
                break
        
        level_by_level.pop()
        return level_by_level
```

## Step 2

- レビュー by GPT-5
  - > frontiers は意味自体は伝わりますが、木の BFS 文脈では current_level と next_level の方が直感的です。
    - 変数名の引き出しがなかった、確かに"level"というキーワードを軸に命名すると一貫性がある気がする。
  - > level_by_level[-1] への都度アクセスよりも、ローカル変数 level_vals に集めてから append する方が読みやすいです。
    - なるほど。
    - インデントレベルによって、どの変数にアクセスするかを制限すると可読性が上がる。
  - > ループキューには deque を使うのが定番ですが、今回のように「レベル単位で for で舐める」パターンならリストでも計算量上の問題はありません。どちらでも良いですが、一般化しやすさで deque を推します。
    - その気持ちはわかる。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.hmkwp3k9djib)
  - 大体のコメントで、そうですよねー、という感覚を持った。
- https://github.com/nanae772/leetcode-arai60/pull/26
  - こちらでも、`nodes_by_level.append([node.val for node in nodes])`でnodes_by_levelへのアクセスがまとまっていて読みやすいと感じた。
  - itertools.chainを初めて見ました。
    - https://docs.python.org/ja/3.13/library/itertools.html
    - https://docs.python.org/ja/3.13/library/itertools.html#itertools.chain
    - 複数のiterableをまるで直列接続したようなiteratorを作る
  - 別にDFSでも頑張ればできるということを学びました。

### 実装3

DFS

```python3
from typing import Optional, List


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        values_ordered_by_level = []
        nodes_with_level = [(root, 0)]
        while nodes_with_level:
            node, level = nodes_with_level.pop()
            
            while level > len(values_ordered_by_level) - 1:
                values_ordered_by_level.append([])
            values_ordered_by_level[level].append(node.val)

            # LIFO: popped from left to right
            if node.right is not None:
                nodes_with_level.append((node.right, level + 1))
            if node.left is not None:
                nodes_with_level.append((node.left, level + 1))
        
        return values_ordered_by_level
```

## Step 3

### 実装4

[実装1](#実装1)の可読性をあげて書く。

```python3
from typing import Optional, List


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if root is None:
            return []
        
        values_ordered_by_level = []
        current_level = [root]
        while current_level:
            level_values = []
            next_level = []
            
            for node in current_level:
                level_values.append(node.val)
                if node.left is not None:
                    next_level.append(node.left)
                if node.right is not None:
                    next_level.append(node.right)
            
            values_ordered_by_level.append(level_values)
            current_level = next_level
        
        return values_ordered_by_level
```
