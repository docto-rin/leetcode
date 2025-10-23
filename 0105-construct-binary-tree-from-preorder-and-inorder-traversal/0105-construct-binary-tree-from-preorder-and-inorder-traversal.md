## Step 1

- 問題文
  - 2つの整数配列`preorder`と`inorder`を受け取る。
  - `preorder`はpreorder（先行順）に、`inorder`はinorder（中間順）に、同一の二分木を走査した結果である。
  - その二分木を構築して返す問題。
  - 制約：
    - 1 <= preorder.length <= 3000
    - inorder.length == preorder.length
    - -3000 <= preorder[i], inorder[i] <= 3000
    - preorder and inorder consist of unique values.
    - Each value of inorder also appears in preorder.
    - preorder is guaranteed to be the preorder traversal of the tree.
    - inorder is guaranteed to be the inorder traversal of the tree.

### 実装1

※Wrong Answer

- アルゴリズムの選択
  - preorderを線形走査して、終点まで常に左に伸ばしていき、あるタイミングで最初（根）まで翻って右に進む方針を立てたが、これは誤り。
    - 根まで翻るとは限らず、途中で右に分岐するパターンもあるが、どのノードまで翻るかの決め方が分からなかった。
    - また、翻りのタイミングを判定する条件も誤っていた。

```python3
from typing import Optional, List


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    # 注意：Wrong Answer
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        tree_size = len(preorder)
        if tree_size == 0:
            return None
        
        for i in range(tree_size):
            if i == 0:
                node = TreeNode(val=preorder[i])
                root = sub_root = node
            elif node is None:
                node = TreeNode(val=preorder[i])
                sub_root.right = node  # <- 常にsub_rootから右に分岐するのが誤り
                sub_root = node
            else:
                node.left = TreeNode(val=preorder[i])
                node = node.left

            if inorder[i] == sub_root.val:  # <- 翻りの判定として不適切
                node = None
        
        return root
```
<details>
  <summary>通らないテストケースの例</summary>

  <img width="680" alt="image" src="https://github.com/user-attachments/assets/c7c749ba-6428-4865-a620-518e38c003e4" />
</details>

### 実装2

- [実装1](#実装1)のアルゴリズムを修正 with GPT-5
  - 誤り1：正しくはvalueが2のノードで（翻って）右に分岐しなければならないが、[実装1](#実装1)では（翻らずに）そのままvalueが3のノードへと左に進んでしまっている。
    - 翻りの条件inorder[i] == sub_root.valが正しくないということ。
    - 正しくは、preorder[ij == inorder[0]である。
      - inorderの処理順が左->自分->右であることから、inorderの先頭に出会ったら、左に行く必要性がなくなる。
  - 誤り2：[実装1](#実装1)では、左への進行を正しいタイミングで止めて翻ったとしても、常に最初（根まで）戻っているのが誤り。
    - 根から左に進行する中で通ってきたノードたちはいずれも右に進行すべきかどうかは未検討である。
    - なので、一気に根まで翻るのではなく、一歩下がる（右上に戻る）のが正しい。
      - そのとき、inorderの先頭を更新する。（inorder用のインデックスをインクリメントしておく）
    - そうしたら次にそのノードがinorder[1]と等しいかみる。等しければまた一歩下がる。
      - このとき、一歩下がる前（左下のノード）において右に進まなくてよかったという情報を付加している。
    - もしinorder[inorder_cursor]と等しくないノードが現れたら、一歩下がる前（左下のノード）から右に分岐する。
      - このとき、一歩下がる前（左下のノード）において右に進まなくてよかったという情報を付加できなかった。
      - しかし、一歩下がる前（左下のノード）は左は処理済みなので、右が次の順序。
        - 右に進むときに、親子関係をつける。（右方向に処理する）
        - 再び、inorder[inorder_cursor]と一致するまで左に進む。（左方向に処理する）
    - また、一歩ずつ翻っていき、それ以上戻れなくなった場合、自身が右に進まなくてよかったという情報はもう得られないので、自動的にその場から右に進む。
- 実装
  - preorderを順に走査してnodeを生成していく。
  - 左に進んでいき、stackに入れていく。
    - stack内は、左は処理済み（子を紐付け済み）だが右は未処理なノード
  - stack[-1].valがinorder[inorder_cursor]と異なれば左に進み、等しければ一歩ずつ翻っていく。
  - 一歩ずつ翻っていくとき、stack[-1].valがinorder[inorder]と異なるノードに出会ったら、その左の子の右の子としてnodeを紐づける。

```python3
from typing import Optional, List


class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        root = TreeNode(preorder[0])
        stack = [root]
        inorder_cursor = 0

        for value in preorder[1:]:
            node = TreeNode(value)
            if stack[-1].val != inorder[inorder_cursor]:
                # go left
                stack[-1].left = node
                stack.append(node)
            else:
                while stack and stack[-1].val == inorder[inorder_cursor]:
                    # go back
                    last = stack.pop()
                    inorder_cursor += 1
                # go right
                last.right = node
                stack.append(node)

        return root
```

### 実装3

- 他のアルゴリズムの発想 with GPT-5
  - preorder の先頭は根である。
  - 根の値を inorder の配列内で探して位置 root_index_in_inorder を得る。
  - inorderは、root_index_in_inorderを境に左右の部分木に分割できることを活用する。
  - 区間をdivide and conquer。
- 実装
  - 簡単に書くために再帰関数を使う。

```python3
from typing import Optional, List


class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        # pseudo
        #
        # preorder: [root, [left subtree], [right subtree]]
        # inorder: [[left subtree], root, [right subtree]]
        # 
        # left subtree
        #   preorder: [left subtree] = [root, [left subtree], [right subtree]]
        #   inorder: [left subtree] = [[left subtree], root, [right subtree]]
        #
        # ...
        #
        # if list.length == 1:
        #   return this node
        # if list.length == 0:
        #   return None
        
        inorder_value_to_index = {}
        for i, value in enumerate(inorder):
            inorder_value_to_index[value] = i

        def build_subtree(pre_first, pre_last, in_first, in_last):
            if pre_first > pre_last:
                return None
            
            root_value = preorder[pre_first]
            root_index_in_inorder = inorder_value_to_index[root_value]
            left_size = root_index_in_inorder - in_first
            right_size = in_last - root_index_in_inorder
            
            left_root = build_subtree(pre_first + 1, pre_first + left_size,
                                      in_first, in_first + left_size - 1)
            right_root = build_subtree(pre_last - right_size + 1, pre_last,
                                       in_last - right_size + 1, in_last)
            
            return TreeNode(val=root_value, left=left_root, right=right_root)

        whole = len(preorder)
        return build_subtree(0, whole - 1, 0, whole - 1)
```

- 展望
  - 引数が過剰。preorderの区間もinorderの区間も長さが同じなので、引数は3つで十分。
  - stack + while loopで書くなど。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.1rv0z8fm6lc3)
  - コード例がC++が多かったので、ぼんやり眺めた。
- https://github.com/nanae772/leetcode-arai60/pull/29/
  - 区間分割をrecursive/iterativeで綺麗に書いてある。

## Step 3

### 実装4

- divide and couquer ([実装3](#実装3)) をiterativeに実装。
- 親子関係の紐付けは、parent, directionをスタックに入れて子の処理でやらせる。
- [実装3](#実装3)の計算順序により忠実に、親の時点で紐づけるには、ダブルポインタ＋二段階スタックループなどをする必要があるはず。
  - https://discordapp.com/channels/1084280443945353267/1421920158506549470/1428453777857581076
  - 実装例：https://github.com/docto-rin/leetcode/pull/23#discussion_r2440407669

```python3
from typing import Optional, List


class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        inorder_value_to_index = {}
        for i, value in enumerate(inorder):
            inorder_value_to_index[value] = i

        dummy = TreeNode()
        subtrees = deque()  # first_index, last_index, parent, is_left
        subtrees.append((0, len(inorder) - 1, dummy, True))
        preorder_index = 0
        while subtrees:
            inorder_first, inorder_last, parent, is_left = subtrees.pop()
            if inorder_first > inorder_last:
                continue
            node = TreeNode(val=preorder[preorder_index])
            if is_left:
                parent.left = node
            else:
                parent.right = node
            preorder_index += 1
            
            inorder_index = inorder_value_to_index[node.val]
            subtrees.append((inorder_index + 1, inorder_last, node, False))
            subtrees.append((inorder_first, inorder_index - 1, node, True))
        
        return dummy.left
```
