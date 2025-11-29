# 235. Lowest Common Ancestor of a Binary Search Tree

- URL: https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/
- Difficulty: Medium
- Tags: Tree, Depth-First Search, Binary Search Tree, Binary Tree
- Notebook: https://share.solve.it.com/d/35bcbca740468ca2f9098c4cb207b97b

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

自分のPR：https://github.com/docto-rin/leetcode/pull/66

**Q:** 単なる Binary Tree 向けの別解について、リファクタして丁寧に書いてください。なお、三項演算子は使わないようにし、ノードがNoneどうかについては、 if ... is not Noneを使用してください 


**A:**
```python
class Solution:
    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        """
        Binary Tree (BSTでなくてもOK) における p と q の LCA を求める。
        
        考え方:
        - 再帰的に左右の部分木を探索し、p または q を見つけたらそのノードを返す
        - 左右両方から結果が返ってきたら、現在のノードが LCA
        - 片方だけから返ってきたら、その結果をそのまま上に返す
        """
        # ベースケース: 現在のノードが p または q なら、それ自体を返す
        if root is p or root is q:
            return root
        
        # 左右の部分木を探索
        left_result = None
        right_result = None
        
        if root.left is not None:
            left_result = self.lowestCommonAncestor(root.left, p, q)
        
        if root.right is not None:
            right_result = self.lowestCommonAncestor(root.right, p, q)
        
        # 左右両方で見つかった場合、現在のノードが LCA
        if left_result is not None and right_result is not None:
            return root
        
        # 片方だけで見つかった場合、その結果を返す
        if left_result is not None:
            return left_result
        
        return right_result
```

**動作の流れ（例）**

```
        3
       / \
      5   1
     / \
    6   2
        ↑p  ↑q (p=6, q=2 とする)
```

1. `root=3`: 左部分木を探索 → `left_result=5`、右部分木 → `right_result=None`
2. `root=5`: 左部分木 → `left_result=6(=p)`、右部分木 → `right_result=2(=q)`
3. 両方見つかったので `root=5` が LCA として返る
4. `root=3` に戻ると `left_result=5`, `right_result=None` なので `5` を返す

**計算量**

- Time: O(n) — 最悪で全ノードを訪問
- Space: O(h) — 再帰スタックの深さ（h は木の高さ）

**あなたの実装2との比較**

| 観点 | 実装2（経路保存） | この実装（再帰） |
|------|------------------|-----------------|
| 空間計算量 | O(n) — 経路をリストで保存 | O(h) — 再帰スタックのみ |
| コード量 | 多い（get_paths_to + get_lca） | 少ない |
| 拡張性 | 3つ以上のノードに対応しやすい | 2ノード専用 |
| 可読性 | 経路が明示的で理解しやすい | 再帰の動きを追う必要あり |

**Q:** レビュワーが提示した元のコードの再帰関数のアルゴリズムの意味は「p, qのうち、木内にあるものについてのLCAを返す」ということですね。

ユースケース次第では許容されることもありそうですが、この関数のユースケースはどんなものが考えられますか？


**A:**
なるほど、面白い視点ですね！「p, q のうち存在するものに対する LCA を返す」という動作が**意図的に有用**なケースを考えてみます。

**ユースケース1: フォールバック付き検索**

ファイルシステムやDOM操作で「2つの要素の共通祖先を取得するが、片方が削除されていても動作を続けたい」場合：

```python
# 例: 2つのファイルの共通ディレクトリを取得
# 片方が削除されていたら、もう片方のディレクトリを返す
common_dir = find_common_ancestor(root, file_a, file_b)
```

**ユースケース2: Optional なターゲット**

「メインターゲット p は必須、サブターゲット q はあれば考慮」というケース：

```python
# 例: ユーザーの現在位置 p と、お気に入り地点 q の共通祖先（地域）を取得
# お気に入りが未設定（木に存在しない）なら、現在位置をそのまま返す
region = get_common_region(map_tree, current_location, favorite_location)
```

**ユースケース3: マージ操作**

Git のようなバージョン管理で、ブランチの共通祖先を探すとき：

```python
# 片方のブランチが既に削除されていても、もう片方の履歴は取得できる
merge_base = find_merge_base(commit_tree, branch_a_head, branch_b_head)
```

**ユースケース4: Graceful degradation**

堅牢性を重視するシステムで「できる範囲で最善の結果を返す」方針：

```python
# 両方あれば LCA、片方なら存在する方、両方なければ None
# 呼び出し側が None チェックして対応する
result = find_lca_or_available(tree, node_a, node_b)
if result is None:
    handle_not_found()
```

---

**ただし注意点**

これらのユースケースでは、**呼び出し側が「片方しか見つからない可能性がある」ことを理解している**前提です。

関数名やドキュメントで明示しないと危険：

```python
# 良い命名例
def find_lca_or_existing_node(root, p, q) -> TreeNode | None:
    """
    Returns:
    - LCA if both p and q exist
    - p if only p exists
    - q if only q exists  
    - None if neither exists
    """
```

LeetCode の問題では「p, q は必ず存在する」という前提なので、この動作の違いは表面化しません。しかし実務では、**関数の契約（何を保証するか）を明確にする**ことが重要ですね。

### 実装5

- レビューを受けて書き直し（BSTではなく単なるBinary Treeの場合）
  - https://github.com/docto-rin/leetcode/pull/66#discussion_r2571495973
    - bottom-up recursiveで書くことで、path全体ではなくLCAだけを追えば十分になる。
    - 再帰関数のアルゴリズムの意味は「p, qのうち、木内にあるものについてのLCAを返す」になる。
    - p, qがroot木内にない場合で意図しない挙動になるのを防ぐために、loggerなどにWARNINGを出すと良さそう。
      - これはprint文で再現する。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
from typing import Optional


class Solution:
    def lowestCommonAncestor(
        self, root: TreeNode, p: TreeNode, q: TreeNode
    ) -> TreeNode:
        self.p = p
        self.q = q
        self.found_p = False
        self.found_q = False

        if p is None:
            print("WARNING: p is None, will be ignored")
            self.found_p = True
        if q is None:
            print("WARNING: q is None, will be ignored")
            self.found_q = True
        
        result = self._get_lca_or_existing_node(root)
        
        if not self.found_p:
            print(f"WARNING: node p (val={p.val}) not found in the tree")
        if not self.found_q:
            print(f"WARNING: node q (val={q.val}) not found in the tree")
        
        return result
    
    def _get_lca_or_existing_node(self, root: TreeNode) -> Optional[TreeNode]:
        """
        Returns:
        - LCA if both p and q exist
        - p if only p exists
        - q if only q exists  
        - None if neither exists
        """
        if root is None:
            return None
        
        left_result = self._get_lca_or_existing_node(root.left)
        right_result = self._get_lca_or_existing_node(root.right)

        is_p = root is self.p
        is_q = root is self.q
        
        if is_p:
            self.found_p = True
        if is_q:
            self.found_q = True

        if is_p or is_q:
            return root
        
        if left_result is not None and right_result is not None:
            return root
        
        if left_result is not None:
            return left_result
        return right_result

```

```python
# テスト用の木を作成
tree_image = r"""
     5
    / \
   3   7
  / \
 2   4
"""
print(tree_image)

root = TreeNode(
    val=5,
    left=TreeNode(
        val=3,
        left=TreeNode(val=2),
        right=TreeNode(val=4)
    ),
    right=TreeNode(val=7)
)

p_exists = root.left               # val=3
q_exists = root.left.left          # val=2
p_not_in_tree = TreeNode(val=999)  # 木に存在しないノード
q_not_in_tree = TreeNode(val=888)  # 木に存在しないノード
```

```python
s = Solution()

test_cases = [
    ("ケース1: p, q 両方存在", p_exists, q_exists),
    ("ケース2: p のみ存在、q は木に存在しない", p_exists, p_not_in_tree),
    ("ケース3: p, q 両方とも木に存在しない", p_not_in_tree, q_not_in_tree),
    ("ケース4: p が None", None, q_exists),
    ("ケース5: q が None", p_exists, None),
    ("ケース6: p, q 両方 None", None, None),
]

for name, p, q in test_cases:
    print(f"\n=== {name} ===")
    result = s.lowestCommonAncestor(root, p, q)
    print(f"Result: {result.val if result else None}")

```