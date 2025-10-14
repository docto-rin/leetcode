## Step 1

- 問題文
  - 2分木の根`root`を与えられ、葉（子を持たないノード）への最短経路上のノード数を返す問題。
  - The number of nodes in the tree is in the range [0, 10^5].

### 実装1

- アルゴリズムの選択
  - BFS風のiterativeとrecursionがある。
  - とりあえずどっちでもできるのでiterativeが好ましい。
- 実装
  - if root is Noneは冒頭で処理しておく。
  - 関数化はオーバーヘッドの割に合わないので行わない。

```python3
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        
        depth = 0
        frontiers = deque()
        frontiers.append(root)
        while frontiers:
            depth += 1
            for _ in range(len(frontiers)):
                node = frontiers.popleft()
                if node.left is None and node.right is None:
                    return depth
                if node.left is not None:
                    frontiers.append(node.left)
                if node.right is not None:
                    frontiers.append(node.right)
```

ここまで5分。迷いなく書けた。

### 実装2

練習のために再帰も書いてみる。帰りがけでの完成をイメージしている。

```python3
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
            
        if root.left is None and root.right is None:
            return 1
        if root.left is None:
            return 1 + self.minDepth(root.right)
        if root.right is None:
            return 1 + self.minDepth(root.left)
        
        return 1 + min(self.minDepth(root.left), self.minDepth(root.right))
```

空行を少しこだわった。こちらも迷いなくかけた。

- そういえば、if-elif-elseを使いたい場面が最近減ってきたが、どういう時に使うのが良いだろうか。
  - 上から下に1列で書いていくプログラミングの記法に合っていない気がして、忌避感が高まっている。
  - 感覚的にはelseの分だけ列数が増え、視覚的に並行して下に下がっていくイメージだが、Python記法的にそれはできないのでインデントブロックで分割している。
  - しかし、elifなどを読んでいる時、頭の中にはifやelseも並行して読んでいる。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.prowafrzksyh)
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1331618526070571079
    - > この depth の更新は while の一番下のほうが素直じゃないでしょうか。(つまり、nodes_depth の更新とともに数を増やします。)
    - たしかに、ループ内での更新位置は統一した方が可読性が高い気がする。
    - 今回の自分のコードは他に更新する変数がないので、今のままでも良いと考えました。
  - https://discord.com/channels/1084280443945353267/1199984201521430588/1336114499584790560
    - > TreeNodeのテストの書き方
    - なるほど、ビジュアルが面白い。勉強になります。
  - https://discord.com/channels/1084280443945353267/1199984201521430588/1336975513159204937
    - > 実際は、テストフレームワークをなんらか使うはずで、そうなると大体の場合は、テストケース一つにつき関数を一つ作ることになるでしょう。

## Step 3

[実装1](#実装1)
