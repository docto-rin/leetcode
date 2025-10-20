## Step 1

- 問題文
  - 二分木の根`root`と整数`targetSum`が与えられる。
  - 根から葉までの経路で、valueを足し上げると`targetSum`に等しくなるような経路があれば`True`を返す。
  - なお葉とは、子のないノードである。
  - 制約：
    - The number of nodes in the tree is in the range [0, 5000].
    - -1000 <= Node.val <= 1000
    - -1000 <= targetSum <= 1000

### 実装1

- アルゴリズムの選択
  - BFS/DFSであらゆる経路について素直に葉まで足し合わせ、葉でtargetSumになればreturn Trueすれば良さそう。
  - node.valは正も負も取りうることから、足し合わせていく和は単調に変動しないので、targetSum以上（以下）になった時点で打ち切りする、などの枝刈りはできないと判断。
- 実装
  - whileループで探索する。
  - キューに入れる前にnodeがNoneでないかを弾くと見通しが良いので、そうする。
    - 最初にroot is Noneを弾いておく必要がある。今回は忘れずにできた。
- 計算量
  - Time: O(V + E) = O(V)
  - Space: O(width) = O(V)

```python3
from typing import Optional
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        if root is None:
            return False
        
        frontiers = deque()
        frontiers.append((root, 0))
        while frontiers:
            node, total = frontiers.popleft()
            total += node.val
            if node.left is None and node.right is None:
                if total == targetSum:
                    return True
                continue
            if node.left is not None:
                frontiers.append((node.left, total))
            if node.right is not None:
                frontiers.append((node.right, total))
        return False
```

ここまで5分。

## Step 2

- GPT-5によるレビュー
  - > 現在は取り出し後に total += node.val。読みやすさを優先するなら、キューには「次に検査する時点での合計」を積むか、
  - 積む時点で加算する場合、足し算の位置がコード中で分散してしまうのが好みでない。
  - > あるいは subtractive 形式で「残り」を積むと、葉の判定が1行で済みます。
  - これは思いつかなかった。しかし、差に注目するのは不必要にロジックを複雑化させているように感じてしまう。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ed3x3pkyeqkp)
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1258455843226255361
    - > これ、引き算先にしちゃって、
      ```python3
      if not node.left and not node.right:
          return rest == 0
      ```
      > のほうが素直ではないでしょうか
    - 最初から同じ感覚で書けたのでよかった。
    - 葉を先に例外的に処理し、その下で通常の子を持つノードを処理する。
  - https://discord.com/channels/1084280443945353267/1228700203327164487/1229111777946505216
    - > いわゆるスタックを使って書き直せますか?スタックは ArrayDeque でしたっけ。Stack は、もう使わないんでしたっけ。
    - pythonなら.popleft()を.pop()にするだけなので簡単。
    - javaでArrayDequeが推奨されていることは自分も以前のレビューで言及したことがある。
      - https://discordapp.com/channels/1084280443945353267/1402285228722229258/1426823682239631420

## Step 3

[実装1](#実装1)
