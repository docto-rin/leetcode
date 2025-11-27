# 235. Lowest Common Ancestor of a Binary Search Tree

- URL: https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/
- Difficulty: Medium
- Tags: Tree, Depth-First Search, Binary Search Tree, Binary Tree
- Notebook: https://share.solve.it.com/d/35bcbca740468ca2f9098c4cb207b97b

## 問題文

<p>Given a binary search tree (BST), find the lowest common ancestor (LCA) node of two given nodes in the BST.</p>

<p>According to the <a href="https://en.wikipedia.org/wiki/Lowest_common_ancestor" target="_blank">definition of LCA on Wikipedia</a>: &ldquo;The lowest common ancestor is defined between two nodes <code>p</code> and <code>q</code> as the lowest node in <code>T</code> that has both <code>p</code> and <code>q</code> as descendants (where we allow <strong>a node to be a descendant of itself</strong>).&rdquo;</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2018/12/14/binarysearchtree_improved.png" style="width: 200px; height: 190px;" />
<pre>
<strong>Input:</strong> root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
<strong>Output:</strong> 6
<strong>Explanation:</strong> The LCA of nodes 2 and 8 is 6.
</pre>

<p><strong class="example">Example 2:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2018/12/14/binarysearchtree_improved.png" style="width: 200px; height: 190px;" />
<pre>
<strong>Input:</strong> root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
<strong>Output:</strong> 2
<strong>Explanation:</strong> The LCA of nodes 2 and 4 is 2, since a node can be a descendant of itself according to the LCA definition.
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre>
<strong>Input:</strong> root = [2,1], p = 2, q = 1
<strong>Output:</strong> 2
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li>The number of nodes in the tree is in the range <code>[2, 10<sup>5</sup>]</code>.</li>
	<li><code>-10<sup>9</sup> &lt;= Node.val &lt;= 10<sup>9</sup></code></li>
	<li>All <code>Node.val</code> are <strong>unique</strong>.</li>
	<li><code>p != q</code></li>
	<li><code>p</code> and <code>q</code> will exist in the BST.</li>
</ul>


## Step 1

### 実装1

- アルゴリズムの選択
  - 最初単なるBinary Treeだと思っていたが、Binary Search Treeとのことだった。
  - であれば、p.val, q.valを頼りにrootから適切な経路で降りていく。
  - pとqがleftとrightに分岐する点がLCAとわかる。
  - BSTの性質から、node.val, p.val, q.valの大小関係により進むべき方向がわかる。
- 実装
  - single loopでiterativeに書けばいいだろう。
- 計算量
  - Time: O(n)
    - 平衡二分木ならlog nだが、直線みたいなBSTだとn回になりうる。
  - Extra Space: O(1)

```python
# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        if any(node is None for node in [root, p, q]):
            raise ValueError("input nodes must not be None")
        
        node = root
        lower, upper = sorted([p.val, q.val])
        while node is not None:
            if node.val > upper:
                node = node.left
                continue
            
            if node.val < lower:
                node = node.right
                continue
            
            return node   

        raise ValueError("node p and q must exist in the BST")
```

### 実装2

- 問題設定の検討
  - 単にBinary Treeの場合を考える。
- アルゴリズムの検討
  - DFSやBFSなどを使ってp, qへの経路を特定したのち、同様の方法でLCAを決定可能。
  - また、元の問題はp, qの2つをターゲットにしているが、3つ以上でも書けるので一般化して書いてみる。
- 実装
  - recursive backtrackingかiterative DFS。一長一短だと思うが、後者でやる。
- 計算量
  - Time: ~~O(n)~~ -> O(n^2)
    - path + [node.left]などでリストの再構築が起きるので。
    - recursive backtrackingにすると形式的にはO(n)と言えるかもしれないが、引数の複製などのオーバーヘッドにより実質的には大差ないという印象。
  - Extra Space: O(n)

```python
import itertools
from collections.abc import Iterable


class Solution:
    def get_paths_to(
        self, root: TreeNode, targets: set[TreeNode]
    ) -> list[list[TreeNode]]:
        if root is None:
            raise ValueError("root node must not be None")

        result_paths = []
        stack = [[root]]
        while stack and len(result_paths) < len(targets):
            path = stack.pop()
            node = path[-1]

            if node in targets:
                result_paths.append(path)

            if node.left is not None:
                stack.append(path + [node.left])
            if node.right is not None:
                stack.append(path + [node.right])

        if len(result_paths) < len(targets):
            raise ValueError("not all targets found in binary tree")
        return result_paths

    def get_lca(
        self, root: TreeNode, targets: Iterable[TreeNode]
    ) -> TreeNode:
        paths = self.get_paths_to(root, set(targets))
        min_length = len(min(paths, key=len))

        for i in range(1, min_length):
            if all(p1[i] is p2[i] for p1, p2 in itertools.pairwise(paths)):
                continue
            return paths[0][i - 1]

        return paths[0][min_length - 1]

    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        return self.get_lca(root, {p, q})

```

- 振り返り
  - [実装1](#実装1)、[実装2](#実装2)ともにValueErrorのメッセージに値を埋め込んだらもっとデバッグしやすいかも。
  - [実装2](#実装2)はモジュール化を割とよくできたと思っているが、さらに良い分け方があるだろうか。
  - 問題設定の改変としては、BSTのまま"All Node.val are unique."をなくすパターンもありそうが、現時点で解決策をパッとイメージできないのでStep 2以降で検討する。


## Step 2

- LLMレビューの感想
  - [実装1](#実装1)は割と良さそう。
    - if-elif-elseよりはif-continue-if-continueの方が好み。
  - [実装2](#実装2)は、改善の余地がある。
    - 特にget_lca。いろいろ書き方を考えたい。
      - pairwiseで比較していくのは素朴・安直だったかも。推移律より常に0番目を比較対象にしても良い。
      - if not allのような否定形は避けられるなら避けたい。
        - not all（少なくとも一つ不成立）自体が認知負荷が高いと感じる。
        - ifの条件はなるべく厳しく書きたい。allは厳しく、肯定形で読みやすい。
      - returnが2つに分かれているが、本質的に同じなのでうまくまとめたい。
        - 例えば、
          ```py
          lca_depth = min_length - 1
          for i in range(1, min_length):
              if not all(p1[i] is p2[i] for p1, p2 in itertools.pairwise(paths)):
                lca_depth = i - 1
                break
          return paths[0][lca_depth]
          ```
    - get_paths_toは、まぁiterativeでいいのではないかという気持ち。
    - ValueErrorには値を埋め込みたいが、凝りすぎて条件分岐を増やすのも保守性が低いかなぁと感じてしまう。
- [コメント集] -> 空
- PR
  - https://github.com/ryosuketc/leetcode_grind75/pull/10
  - https://github.com/huyfififi/coding-challenges/pull/10
  - https://github.com/rihib/leetcode/pull/29
  - https://github.com/thonda28/leetcode/pull/12
  - https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/22

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
各PRを確認しますね。

各PRの内容をまとめました：

---

**1. ryosuketc/leetcode_grind75 #10** (C++)
- **ソリューション**: BSTの性質を利用したiterative解法。stackを使用。
- **主なレビューコメント**:
  - `std::stack<TreeNode*>` は高々1要素しか入らないので、単に `TreeNode*` でよい（nodchip）
  - C++での例外の重さについて議論。Exceptionは約200μsかかる。Google社内で例外を使わない理由は「重さ」よりも「大域脱出でコードが追えなくなる」ことが大きい（oda）
  - `std::unreachable`（C++23）について言及

---

**2. huyfififi/coding-challenges #10** (Python)
- **ソリューション**: 再帰版とiterative版の両方を実装。BSTの性質を活用。
- **主なレビューコメント**:
  - 再帰版で `return` が抜けていた指摘（oda）
  - 「All Node.val are unique」という制約がないと、全部同じ値だったら全部調べないとわからない、という補足（oda）

---

**3. rihib/leetcode #29** (Go)
- **ソリューション**: BSTの性質を利用したiterative解法。
- **主なレビューコメント**:
  - **NaNの問題**: float型だと`NaN`との比較が常にfalseになり無限ループになる可能性。「必ずどこかでtrueが返ることを期待する書き方は危険」という教訓（PR説明文）
  - Go のインデントについて：GitHub上では tab が8スペースで表示される件（thonda28）

---

**4. thonda28/leetcode #12** (Python)
- **ソリューション**: BSTの性質を利用したiterative解法。if-elif-else構造を採用。
- **主なレビューコメント**:
  - `p.val` と `q.val` の大小比較をループ前にやると、if文の判定条件がシンプルになる（liquo-rice）
  - 「同じ比較を何度もしている」点が気になる（liquo-rice）

---

**5. NobukiFukui/Grind75-ProgrammingTraining #22** (Python)
- **ソリューション**: 再帰版とiterative版の両方。
- **主なレビューコメント**:
  - `root` を直接変更していくのは違和感がある。`node` に代入してから操作すべき（nodchip）
  - if-elif-else の一貫性について：`if-if-else` ではなく `if-elif-else` か `if-if-return` のどちらかにすべき（liquo-rice）

---

**共通して見られるポイント**:
1. `root` は変更せず `node` に代入して操作する
2. p, qの大小を事前に比較しておくと条件がシンプルになる
3. if-elif-else の書き方の一貫性
4. 再帰で return を忘れないこと

### 実装3

- 問題設定の検討
  - 重複した値を許すBSTを考える。
- アルゴリズムの検討
  - node.valがlowerやupperと一致した時、leftもrightも考える探す必要がある。
    - この時点で、[実装1](#実装1)のような単一の経路を追っていく方針は使えない。
  - [実装2](#実装2)に枝刈りを追加すると考えた方がいい。
    - node.left, node.rightをstackに入れる条件が追加される。
- 実装
  - せっかくなので今度はbacktrackingで書く。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
class Solution:
    def get_paths_to(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> list[list[TreeNode]]:
        path = []
        found_paths = []
        targets = {p, q}
        lower, upper = sorted([p.val, q.val])
        
        def search(node):
            if node is None or len(found_paths) == 2:
                return
            
            path.append(node)

            if node in targets:
                found_paths.append(path.copy())
            
            if node.val >= lower:
                search(node.left)
            if node.val <= upper:
                search(node.right)

            path.pop()
        
        search(root)
        return found_paths

    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        found_paths = self.get_paths_to(root, p, q)
        if len(found_paths) < 2:
            raise ValueError("both p and q must exist in BST")

        min_length = len(min(found_paths, key=len))
        lca_depth = min_length - 1
        for i in range(1, min_length):
            if found_paths[0][i] is not found_paths[1][i]:
                lca_depth = i - 1
                break
        return found_paths[0][lca_depth]

```

```python
s = Solution()
s.lowestCommonAncestor(root1, p1, q1).val
```

```
→ 3
```

## Step 3

### 実装4

- 何もみないで書く。

```python
from collections.abc import Iterable


class Solution:
    def get_paths_to(
        self, root: TreeNode, target_nodes: Iterable[TreeNode]
    ) -> list[list[TreeNode]]:
        stack = [[root]]
        found_paths = []
        targets = set(target_nodes)
        lower = min(targets, key=lambda node: node.val).val
        upper = max(targets, key=lambda node: node.val).val
        while stack and len(found_paths) < len(targets):
            path = stack.pop()
            node = path[-1]
            
            if node is None:
                continue
            
            if node in targets:
                found_paths.append(path)
            
            if node.val >= lower:
                stack.append(path + [node.left])
            if node.val <= upper:
                stack.append(path + [node.right])
                
        return found_paths

    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        found_paths = self.get_paths_to(root, {p, q})
        if len(found_paths) < 2:
            raise ValueError("both p and q must exist in BST")
        
        min_length = len(min(found_paths, key=len))
        lca_depth = min_length - 1
        for i in range(1, min_length):
            if found_paths[0][i] is not found_paths[1][i]:
                lca_depth = i - 1
                break
        return found_paths[0][lca_depth]

```

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装5

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```