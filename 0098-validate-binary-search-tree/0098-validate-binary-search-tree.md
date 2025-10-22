## Step 1

- 問題文
  - 二分木の根`root`を受け取り、有効な二分探索木（BST）か判断せよ。
  - 有効なBSTとは、
    - The left subtree of a node contains only nodes with keys strictly less than the node's key.
    - The right subtree of a node contains only nodes with keys strictly greater than the node's key.
    - Both the left and right subtrees must also be binary search trees.
  - 制約：
    - The number of nodes in the tree is in the range [1, 10^4].
    - -2^31 <= Node.val <= 2^31 - 1

### 実装1

- アルゴリズムの選択
  - 以下の3つが思いついた。
    - inorder（中間順）に走査してprev.val < curr.valを確かめていく方法。
    - bottom-up（帰りがけ）にBSTの有効性を積み上げていく方法。
    - top-down（行き掛け）にBSTの有効性を確かめていく方法。
  - inorderの方法は走査順序さえ間違えなければシンプルで、計算量も小さいので良さそうだと思った。
    - しかし、再帰を使わないと自力で実装できないと感じた。
  - BSTの検証を再帰的に行う2個目と3個目だが、引き継ぐ情報が複雑で、整理するのに時間がかかりそうだと感じた。
    - おそらく、subtreeのrootノード以外に、最大値、最小値などの情報を管理する必要がありそう。
  - 実装のしやすさ、可読性の高さからinorderを選択。計算量や再帰深度の比較はできていない。
  - また、いずれの方法にしても（自分の実装力では）iterativeに書けないなと思った。
- 実装
  - inorderにノードを取り出すgeneratorを定義し、これを使って走査する。
  - 始点処理だが、最初にgeneratorを1回next()で進めておくのが自然と感じた。
  - 入力制約からroot is not Noneは保証されている。
    - root is Noneのとき、True/Falseどちらを返すのも自然と思わなかったので、エラー送出にした。
- 計算量
  - Time: O(n)
  - Space: O(depth) = O(n)

```python3
from typing import Optional


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            raise ValueError("root node must not be None")

        def generate_inorder(node):
            if node.left is not None:
                yield from generate_inorder(node.left)

            yield node

            if node.right is not None:
                yield from generate_inorder(node.right)
        
        node_generator = iter(generate_inorder(root))
        previous_val = next(node_generator).val
        for node in node_generator:
            if node.val <= previous_val:
                return False
            previous_val = node.val
        return True
```

ここまで16分。

- fyi
  - `yield from`とすると、iterableから（または、委譲先のジェネレータがyieldするたびに）値を逐次収集し、すべてのyieldが終わったら呼び出し元に制御を戻す。
  - `yield`とすると、1回収集した後、先に進む。
  - https://docs.python.org/ja/3.14/reference/expressions.html#yield-expressions

### 実装2

- bottom-up（帰りがけ）にBSTの有効性を積み上げていく方法。
- 計算量
  - Time: O(n)
  - Space: O(depth) = O(n)

```python3
from typing import Optional


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            raise ValueError("root node must not be None")

        def is_valid_bst(root):
            minimum = root.val
            maximum = root.val
            
            if root.left is not None:
                left_valid, left_min, left_max = is_valid_bst(root.left)
                if not left_valid or left_max >= root.val:
                    return False, 0, 0
                minimum = left_min
            
            if root.right is not None:
                right_valid, right_min, right_max = is_valid_bst(root.right)
                if not right_valid or right_min <= root.val:
                    return False, 0, 0
                maximum = right_max
            
            return True, minimum, maximum
        
        return is_valid_bst(root)[0]
```

## Step 2

- GPT-5によるレビュー
  - [実装1](#実装1)
    - > 深い木で再帰が落ちる可能性があります。反復版を提示します。
      - 反復でかけるならほとんどの場合反復で書いたほうがいいのは同意。
    - > yield node ではなく yield node.val にすれば属性アクセスを減らせます。
      - 確かに。
    - > root is None で例外はやや攻撃的です。True を返すか、少なくともドキュメント化しましょう。
      - 攻撃的なのはそうかもしれない。APIドキュメントに書いてなかったらユーザーはびっくりするかも。
      - Falseを返すのは入力が二分木かつ二分探索木でない場合、という保守的な観点からは、if root is None: return Trueは納得できる。
  - [実装2](#実装2)
    - > 関数名の引数名が root なのに内部で部分木を受け取っており紛らわしいです。
      - subtreeの根という意図だが、確かに紛らわしいかも。nodeの方がいいか。
    - > docstringを足しましょう
      - GPT-5が書いたdocstringに、"Post-order aggregates (is_valid, min_val, max_val) for each subtree."と書いてあったが、post-orderという見方が自分はStep 1時点でできていなかった。
      - 最初に思いついた方法は、inorder（中間順）, bottom-up -> postorder（後行順）, top-down -> preorder（先行順）とまとめ直せる。
  - Q. 任意のrecursive解法は、iterativeでも書ける、と考えて良さそうですか？
    - > A. 再帰は「スタック操作の構文糖」であり、構文糖を展開すれば必ず iterative にできる。ただし、**人間にとっての可読性・メンテ性**の観点では再帰のほうがしばしば自然。常にiterativeに直すべきとは限りません。
      - これ、当たり前なはずですが、自分はしばしばiterativeで書き直すことに苦労してます。
  - Q. recursiveをiterativeに直す汎用的な方法はありますか？
    - > A. 「再帰→反復」変換の汎用テンプレ（色付きスタック）
      ```python3
      WHITE, GRAY = 0, 1  # WHITE: これから展開, GRAY: 再訪(処理点)
      stack = [(WHITE, root)]
      while stack:
          color, node = stack.pop()
          if node is None:
              continue
          if color == WHITE:
              # preorder: 自分→子 の順にしたいなら、(GRAY, node) を最初に積む
              stack.append((WHITE, node.right))
              stack.append((GRAY, node))            # inorderにしたければここで GRAY を挟む
              stack.append((WHITE, node.left))
              # postorder: 子→自分 の順にしたいなら (GRAY, node) を最後に積む
          else:
              # ここが「訪問（処理）」の場所。pre/in/post は積む順序で決まる
              pass
      ```
  - Q. 色（フラグ）なしで汎用的に表現できますか？
    - > * **可能**ですが、完全に統一した 1 つのループ骨格で pre/in/post を切り替えるには、
      >   * 上のような**push 順序の工夫**（pre/in/post で順序だけ変える）、
      >   * もしくは **2スタック**/`last_visited` といった**構造的な工夫**が必要です。
      > * 「色（状態フラグ）を使わない＝状態を持たない」ではありません。**状態はどこかで必要**で、       
      >   * push 順序
      >   * `last_visited`
      >   * 2本目のスタック
      > 
      > のいずれかに“押し込む”だけです（= フラグの等価表現）。

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.r5pdnx6retf4)
  - https://discord.com/channels/1084280443945353267/1226508154833993788/1279677251193012254
    - > Generator + 再帰、強力ですね。pros cons の cons としては、再帰の深さの限界があること、走りかけの Generator を木の深さ分だけ作るのでそこそこ重いことがありそうですね。
    - > inner にすることは、たとえば、その外側のローカル変数を使うとかで、せざるをえないことがありますが、そうでなければ、class method として並べちゃったほうが私は読みやすいと思いますね。入れ子にすると外側の関数が呼ばれるたびに内側が作られることになるので。
      - 同意します。
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1354481736192491601
    - > ここにキャッシュがついていると、上の isValidBST を2回呼んだ時、つまり、木を変更してもう一回呼んだときに、TreeNode の id でキャッシュされると思われるので、木の先のほうが変更されていても、同じ値が返るという意味でうまくいかないのではないでしょうか。
      > なので、isValidBST をヘルパー関数に変更して、インナーファンクションとして isValidBST と subtreeMinMax を書いたほうがいいのではないでしょうかね。
      - これ、なるほどなぁと思いました。@cacheのキャッシュはあくまで異なる`root`を跨いで生存してほしくないということ。
      - また、再帰ののたびにsubterrMinMax関数が定義されるのを避けるために、再帰関数内ではなく、再帰関数と同スコープのinner functionにするというのも深いなぁと感じた。
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1320305275478999091
    - > node.left と left が違う型のものなのがわりと混乱する感じがします。
      - 同意します。以前も二分木で左右pivotを書くとき、この理由からleft, rightの命名は避けた記憶があリます。
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1338273196557733971
    - > 上から範囲の制約を下げていくか、下から値の範囲を上げていくか、インオーダーで出力して昇順になっているか、などでしょうか。
      - とりあえず選択肢は網羅できている。あとは、頭に描いたアルゴリズムを綺麗に実装する力が足りていないと感じる。
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1338272059142312038
    - infについて
    - > 私はこれは許容かなと思います。ただ、型が違うのが少し違和感ですね。本当に気になるならば None を渡して場合分けというのも一つです。
      - infを使った方が可読性が高くて好み。型が違うのは確かに違和感。
- https://github.com/YukiMichishita/LeetCode/pull/8
  - preorder(recursive/iterative), inorder(recursive/iterative) など様々な方法を書かれていて参考になる。

### 実装3

- top-down（preorder）をrecursiveで実装。
- 計算量
  - Time: O(n)
  - Space: O(depth) = O(n)

```python3
from typing import Optional


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst(node, low, high):
            if node is None:
                return True
            if node.val <= low or node.val >= high:
                return False
            return (is_valid_bst(node.left, low, node.val) 
                and is_valid_bst(node.right, node.val, high))

        return is_valid_bst(root, float("-inf"), float("inf"))
```

### 実装4

- top-down（preorder）をiterativeで実装。
- [実装3](#実装3)をiterativeに直しやすいと感じるのは末尾再帰の形に近いからだろうか。

```python3
from typing import Optional
from collections import deque


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        nodes_with_bounds = deque([(root, -float("inf"), float("inf"))])
        while nodes_with_bounds:
            node, low, high = nodes_with_bounds.pop()
            
            if node is None:
                continue
            
            if not (low < node.val < high):
                return False
            
            nodes_with_bounds.append((node.left, low, node.val))
            nodes_with_bounds.append((node.right, node.val, high))
        
        return True
```

### 実装5

- inorderをiterativeに実装。
- 全く筆が進まなかった。
  - 再帰の関数呼び出しスタックを頭で描けていないせい。
  - GPT-5によれば、
    > 再帰版からこの iterative 版（明示スタックによる inorder）への書き換えは、手順を一つひとつ理解して展開しないと到達できません。
    > 多くの初学者や中級者は「再帰はスタック操作の構文糖」という理屈を知っていても、それを実際に展開してバグなく動かすのは難しいです。...
    - 自分は現状再帰関数を魔法のように使ってしまっている...
- nodeにNoneが入ることがないように書いてみました。 
- 計算量
  - Time: O(n)
  - Space: O(depth) = O(n)

```python3
from typing import Optional


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            return True

        stack = [root]
        node = root
        previous = None
        while True:
            while node.left is not None:
                stack.append(node.left)
                node = node.left
            
            while stack:
                node = stack.pop()
                if (previous is not None
                    and node.val <= previous):
                    return False
                previous = node.val

                if node.right is not None:
                    stack.append(node.right)
                    node = node.right
                    break
            else:
                return True
```

### 実装6

- [実装5](#実装5)でwhile-elseを避ける場合は下記が良いと思う。
- while-elseによるreturnが自然だとは思う。"else"がもっとわかりやすい単語なら[実装5](#実装5)の方が終了タイミングがより明確で好み。

```python3
from typing import Optional


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            return True
        
        stack = [root]
        node = root
        previous_value = None
        while stack:
            while node.left is not None:
                stack.append(node.left)
                node = node.left
            
            while stack:
                node = stack.pop()
                if (previous_value is not None
                    and node.val <= previous_value):
                    return False
                previous_value = node.val
                
                if node.right is not None:
                    stack.append(node.right)
                    node = node.right
                    break
        
        return True
```

### 実装7

- bottom-up（帰りがけ）をiterativeで書く方法。

```python3
from typing import Union, Optional
from dataclasses import dataclass


@dataclass
class Subtree:
    low: Union[int, float] = float("inf")
    high: Union[int, float] = -float("inf")


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            return True
        
        preorder = [root]
        postorder = []
        
        while preorder:
            node = preorder.pop()
            postorder.append(node)
            if node.left is not None:
                preorder.append(node.left)
            if node.right is not None:
                preorder.append(node.right)
        
        subtrees = {}  # {TreeNode: Subtree}
        while postorder:
            node = postorder.pop()
            left_subtree = subtrees.get(node.left, Subtree())
            right_subtree = subtrees.get(node.right, Subtree())
            if not (left_subtree.high < node.val < right_subtree.low):
                return False
            subtrees[node] = Subtree(
                low=min(node.val, left_subtree.low),
                high=max(node.val, right_subtree.high)
                )
        return True
```

## Step 3

[実装6](#実装6)に同じ。

| algorithm | recursive | iterative |
| --------- | --------- | --------- |
| preorder  | [実装3](#実装3) | [実装4](#実装4) |
| inorder   | [実装1](#実装1) | [実装5](#実装5)、[実装6](#実装6) |
| postorder | [実装2](#実装2) | [実装7](#実装7) |
