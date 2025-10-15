## Step 1

- 問題文
  - 2つの2分木の根`root1`、`root2`を受け取り、マージした木を返す問題。
  - The number of nodes in both trees is in the range [0, 2000].
  - -10^4 <= Node.val <= 10^4
  - 以下の説明のため、木の名前をそれぞれtree1, tree2, merged treeとする

### 実装1

- アルゴリズムの選択
  - BFSでもDFSでも手数は変わらない。BFS風に書くことにする。
- 実装
  - root1 is None and root2 is Noneのケースは入力に対する例外処理になるので、念頭におく。
  - キューに入れるのは、マージしたいtree1, tree2のノードと、merged treeの対象ノードの親、その親と子の関係（left or right）の4つと考えた。
    - 例：frontiers.append((root1, root2, dummy, "left"))
  - 最初に考えたのは、マージしたいtree1, tree2のノードと、merged treeの対象ノードの親が持つ子のポインタの3つと考えたが、失敗した。
    - 例：frontiers.append((root1, root2, dummy.left))
    - こうしたとき、dummy.leftがNoneであると、キューに入れた時点で親（dummy）との関係性が途切れてしまう。
    - キューには基本オブジェクト（の参照）を入れるようにし、属性（ダブルポインタ）を入れるときはその意味を注意した方がよさそう。
  - merged tree用のnodeは、tree1とtree2から奪ってvalを書き換える方向性なら新規作成ノードをなくせそう。
    - だが、速さのために破壊的にするのは嬉しくないと考えたので逐次新規作成することにする。
  - merged treeのrootは、番兵の子としておくか、ループ内の条件分岐でポインタを保存しておく方法があるが、ループ内の簡潔のため番兵を使う。

```python3
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        def calc_merged(node1, node2):
            if node1 is None and node2 is None:
                return None
            
            merged = TreeNode(val=0)
            if node1 is not None:
                merged.val += node1.val
            if node2 is not None:
                merged.val += node2.val
            return merged

        def append_children_safely(node1, node2, merged, frontiers):
            node1_left = None
            node1_right = None
            node2_left = None
            node2_right = None
            if node1 is not None:
                node1_left = node1.left
                node1_right = node1.right
            if node2 is not None:
                node2_left = node2.left
                node2_right = node2.right
            frontiers.append((node1_left, node2_left, merged, "left"))
            frontiers.append((node1_right, node2_right, merged, "right"))

        dummy = TreeNode()
        frontiers = deque()
        frontiers.append((root1, root2, dummy, "left"))
        while frontiers:
            node1, node2, merged_parent, direction = frontiers.popleft()
            merged_child = calc_merged(node1, node2)
            if merged_child is None:
                continue
            
            if direction == "left":
                merged_parent.left = merged_child
            elif direction == "right":
                merged_parent.right = merged_child
            append_children_safely(node1, node2, merged_child, frontiers)
        
        return dummy.left 
```

ここまで30分。

- アルゴリズムは正しいと思われるが、実装が明らかに美しくない。
  - 推測1：キューに詰める情報が適切でない？（過剰？）
  - 推測2：関数間に責務の重複がある？（append_children_safely関数は特に気持ち悪さがある）
- Runtime: 18 ms (Beats 6.24%) とのこと。
  - 遅いのはPythonかつノードを全て新規作成しているから当たり前だろう。
  - [実装1](#実装1)が遅い事実自体よりも、多くの人が破壊的な方法を選択しているらしいことに驚いた。
    - 破壊的な方法 & deepcopyの方が速いとかはあり得るのだろうか？後で試そう。

### 実装2

[実装1](#実装1)と同様の方針だが、キューに詰める内容、while内で行う処理を見直した。

whileの外で先にmerged_rootを作れるようになった（＝while内に条件分岐を増やさずとも番兵を回避できるようになった）ので、番兵をやめてみた。

```python3
from collections import deque


class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        def calc_merged(node1, node2):
            if node1 is None and node2 is None:
                return None
            
            merged = TreeNode(val=0)
            if node1 is not None:
                merged.val += node1.val
            if node2 is not None:
                merged.val += node2.val
            return merged

        merged_root = calc_merged(root1, root2)
        if merged_root is None:
            return merged_root
        
        frontiers = deque() # (Optional[TreeNode], Optional[TreeNode], TreeNode)
        frontiers.append((root1, root2, merged_root))
        while frontiers:
            node1, node2, merged = frontiers.popleft()
            
            node1_left = None
            node1_right = None
            node2_left = None
            node2_right = None
            if node1 is not None:
                node1_left = node1.left
                node1_right = node1.right
            if node2 is not None:
                node2_left = node2.left
                node2_right = node2.right
            
            merged.left = calc_merged(node1_left, node2_left)
            merged.right = calc_merged(node1_right, node2_right)
            if merged.left is not None:
                frontiers.append((node1_left, node2_left, merged.left))
            if merged.right is not None:
                frontiers.append((node1_right, node2_right, merged.right))
        
        return merged_root
```

- これは[実装1](#実装1)より明確に可読性が改善したと感じる。
- 実装1, 2だと、破壊的に（新規インスタンス生成を制限して）やりたくなったときにロジックの書き直しが生じる。
  - 一方で、最初から破壊的に書いておけば、破壊したくない時はdeepcopyするだけで切り替え可能になる。
  - なので入力を破壊するか否か迷ったら、最初は破壊的に書くのが案外良いのかもしれないと思った。
- 一方がNoneになったら、その部分木は一括で処理できることに気づいた。
  - これは破壊的なやり方ベースで考えないと思いつけない。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.cxy3cik6kyqx)
  - https://discord.com/channels/1084280443945353267/1295357747545505833/1375523076468506654
    - > setattr、getattr、attrgetter あたりを使うべきでしょうね。
      - https://docs.python.org/ja/3/library/functions.html#setattr
      - https://docs.python.org/ja/3/library/functions.html#getattr
      - https://docs.python.org/ja/3/library/operator.html#operator.attrgetter
    - GPT-5に表でまとめてもらった。
      | 関数           | 目的          | 実装2での利用価値       |
      | ------------ | -----------    | --------------- |
      | `setattr`    | 動的属性設定     | 実装1では有用、実装2では不要          |
      | `getattr`    | 動的属性取得     | 不要（TreeNodeの属性は固定）         |
      | `attrgetter` | 高階関数用getter | 不要（map/sort/filterとの併用が普通）|
    - setattrは[実装1](#実装1)で有用。
      - 例：setattr(merged_parent, direction, merged_child)
    - getattrは属性が存在しない場合にデフォルト値を返せるのが便利だと思った。辞書の.get()アクセスに近い。
      - 例1：merged.val += getattr(node1, "val", 0)
      - 例2：node1_left = getattr(node1, "left", None)
    - attrgetterはnodes.sort(key=attrgetter("val"))みたいに使うやつ。itemgetterとかの仲間。
      | 関数                                   | 意味            | 対応              |
      | ------------------------------------ | ------------- | --------------- |
      | `operator.itemgetter(key)`           | インデックスやキーアクセス | `x[key]`        |
      | `operator.attrgetter(name)`          | 属性アクセス        | `x.name`        |
      | `operator.methodcaller(name, *args)` | メソッド呼び出し      | `x.name(*args)` |
  - https://discord.com/channels/1084280443945353267/1262688866326941718/1297934906189549599
    - ダブルポインタ
      | 概念       | Pythonでの直感      | Goでの型        |
      | -------- | --------------- | ------------ |
      | 値そのもの    | `TreeNode(val)` | `TreeNode`   |
      | 参照（ポインタ） | 変数に入れたオブジェクト    | `*TreeNode`  |
      | 参照への参照（ダブルポインタ） | 「変数自体を指すもの」     | `**TreeNode` |
    - pythonではダブルポインタ**TreeNodeは定義できず、やり取りも不可能。
    - Go の、
      ```go
      stack = append(stack, Nodes{left(node1), left(node2), &(*merged).Left})
      ```
      は、pythonっぽい疑似コードで書くと、
      ```python3
      stack.append((node1.left, node2.left, reference_to(merged.left)))
      ```
      となるが、pythonにはreference_toのような関数・概念は存在しない。

## Step 3

### 実装3

破壊的な方法。tree1をベースに作る。

```python3
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if root1 is None:
            return root2
        if root2 is None:
            return root1
        
        frontiers = []  # (TreeNode, TreeNode)
        frontiers.append((root1, root2))
        while frontiers:
            node1, node2 = frontiers.pop()
            node1.val += node2.val

            if node1.left is None:
                node1.left = node2.left
            elif node2.left is None:
                pass
            else:
                frontiers.append((node1.left, node2.left))
            
            if node1.right is None:
                node1.right = node2.right
            elif node2.right is None:
                pass
            else:
                frontiers.append((node1.right, node2.right))
        
        return root1
```

- 考え方の一つとして、キューに入れるオブジェクトにNoneが入る可能性を排除する（型安全性）が良い方法に繋がった気がする。
- if-elif-elseのところ、elif: passを書くかは賛否分かれそう。
  - [実装3](#実装3)では、すべての場合が網羅されている。
- 非破壊でやるならroot1 = copy.deepcopy(root1)を追加すればいいだろう。
