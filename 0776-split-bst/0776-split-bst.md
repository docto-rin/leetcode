## Step 1

- 問題文（LINTCODE版）
  - 二分探索木（BST）の根`root`とこのBSTを2つの部分木に分割するためのターゲット値`v`が与えられる。
  - 2分割される部分木のうち、一方は`v`以下の値を持つノード、他方は`v`より大きい値を持つノードから構成される。
  - 元の木`root`は必ずしも`v`に等しい値を持つノードを持つ必要はない。
  - さらに、元の木`root`の大部分の構造は保持されるべきである。
  - 簡単に言えば、元の木において親ノードPを持つ任意の子ノードCについて、分割後も両者が同じ部分木内にある場合、ノードCは依然としてPの子ノードであるべきである。
  - この分割された2つの部分木のうち、ノード数の多い方を返せ。
  - もしノード数が同じである場合は、根の値が大きい方を返せ。
  - 制約：
    - The size of the BST will not exceed 50
    - The BST is always valid and each node's value is different

### 実装1

- アルゴリズムの選択
  - 説明のため、`v`以下の値からなる部分木と`v`より大きい値からなる部分木を、左部分木、右部分木と呼ぶことにする。
  - 分割地点自体は、BSTを活かしてTreeNode版の二分探索で容易に特定できる。
  - 難関は、つなげかえが必要になる場合と、返す部分木をどちらにするかの判断方法。
  - divide and conquer、つまりtop-down的な解決を目指す。
- 実装
  - 再帰関数か、bottom-up化してiterativeだが、前者でさえも実装難易度が高いので前者で済ませる。
  - ヘルパー再帰関数を定義する。根ノードを入力とし、左部分木と右部分木を返り値とする。
    - このとき、再帰関数内で帰りがけの作業としてつなげかえを行う。
  - ヘルパー再帰関数の外側で2つの部分木のノード数のカウンタ変数を定義し、nonlocal宣言して更新する。
    - これ以外には、更新値を返り値にするか、クラス属性にするなどがある。
    - 返り値を増やすのは可読性が下がるのでやりたくない。クラス属性は、今回は他にメソッドを定義しないので見送る。
- 計算量
  - Time: O(n)
  - Space: O(depth) = O(n)

```python3
from lintcode import (
    TreeNode,
)

"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    """
    @param root: the given tree
    @param v: the target value
    @return: the root TreeNode after splitting
    """
    def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
        """Split a BST by value v and return the subtree with more nodes.
        Note that this method is destructive to the original tree pointers.
        """
        left_size = 0
        right_size = 0
        
        def split_bst_aux(node):
            nonlocal left_size
            nonlocal right_size
            
            if node is None:
                return None, None
            
            if node.val <= v:
                left_size += 1
                left_root, right_root = split_bst_aux(node.right)
                split_bst_aux(node.left)
                node.right = left_root
                return node, right_root

            right_size += 1
            left_root, right_root = split_bst_aux(node.left)
            split_bst_aux(node.right)
            node.left = right_root
            return left_root, node
        
        left_root, right_root = split_bst_aux(root)
        
        if left_size > right_size:
            return left_root
        if left_size < right_size:
            return right_root
        if left_root.val > right_root.val:
            return left_root
        return right_root
```

- 30分くらいかかった。
  - 書き始める前、繋ぎかえの一般化ができず苦しんだ。
- 思いつき、書き始めてからはスムーズに書けた。
  - 1回誤答した。単一の部分木内と確定した側も再帰関数を呼ばないと、部分木のサイズが正しく測れない。（再帰関数の返り値を捨てている行のことです。）

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.3czeid3ovy2a)
- https://github.com/TORUS0818/leetcode/pull/49/
  - 分割（ポインタのつなげかえ）の走査とカウントの走査を分けている。この方が可読性が高い気もする。
  - > 必ず large_root.val のほうが大きくなると思います。
  - 見落としていた。
- https://github.com/Mike0121/LeetCode/pull/16
  - > 左右の対称性をソースコードでも示すため、こちらのように if else の形で書いたほうが良いと思います。
  - これは書いてみたら確かに読みやすかった。常にif-continueにしたらいいわけじゃないことを学んだ。
  - > left, rightより、smaller, largerの方が情報が載ると思いました。
  - その通りだと思った。left, rightはポインタのleft, rightとも混ざるのが良くない。
- その他、[実装1](#実装1)を振り返って：
  - `root`がNoneのときや、部分木の根がNoneのとき、if left_root.val > right_root.valでAttributeErrorになるのが良くない。
  - ロジックが複雑で気が配れていなかった。

## Step 3

### 実装2

```python3
from lintcode import (
    TreeNode,
)

class Solution:
    def split_b_s_t(self, root: Optional[TreeNode], v: int) -> Optional[TreeNode]:
        """split bst by value v. return the subtree with more nodes.
        please note that this method is destrutive.
        """
        def split_bst_aux(node):
            if node is None:
                return None, None
            
            if node.val <= v:
                root_lower, root_higher = split_bst_aux(node.right)
                node.right = root_lower
                return node, root_higher
            else:
                root_lower, root_higher = split_bst_aux(node.left)
                node.left = root_higher
                return root_lower, node
        
        def count_nodes(node):
            if node is None:
                return 0
            return 1 + count_nodes(node.left) + count_nodes(node.right)
        
        root_lower, root_higher = split_bst_aux(root)
        size_lower = count_tree_nodes(root_lower)
        size_higher = count_tree_nodes(root_higher)
        
        if size_lower > size_higher:
            return root_lower
        return root_higher
```
