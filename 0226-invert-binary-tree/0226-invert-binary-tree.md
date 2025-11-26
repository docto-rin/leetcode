# 226. Invert Binary Tree

- URL: https://leetcode.com/problems/invert-binary-tree/
- Difficulty: Easy
- Tags: Tree, Depth-First Search, Breadth-First Search, Binary Tree
- Notebook: https://share.solve.it.com/d/9bd3c55307a8d0799a9d6b2fdeb50133

## 問題文

<p>Given the <code>root</code> of a binary tree, invert the tree, and return <em>its root</em>.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2021/03/14/invert1-tree.jpg" style="width: 500px; height: 165px;" />
<pre>
<strong>Input:</strong> root = [4,2,7,1,3,6,9]
<strong>Output:</strong> [4,7,2,9,6,3,1]
</pre>

<p><strong class="example">Example 2:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2021/03/14/invert2-tree.jpg" style="width: 500px; height: 120px;" />
<pre>
<strong>Input:</strong> root = [2,1,3]
<strong>Output:</strong> [2,3,1]
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre>
<strong>Input:</strong> root = []
<strong>Output:</strong> []
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li>The number of nodes in the tree is in the range <code>[0, 100]</code>.</li>
	<li><code>-100 &lt;= Node.val &lt;= 100</code></li>
</ul>


## Step 1

### 実装1

- アルゴリズムの選択
  - 全てのノードの.leftと.rightをswapできればいい。順序はなんでもいい。
  - preorderが最も自然なのでpreorderでやる。
- 実装
  - recursiveでもiterativeでもいいが、preorderはいわば末尾再帰の形なのでiterativeでも容易にかける。
    - iterativeでやる。
  - Noneチェックをstackにappend前にするか、pop後にするか。
    - どっちでもいいがコード中に分散しないpop後にやる方法で書く。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)
    - 実装よりここが悩んだ。
    - Noneもappendするのでstackに最大n個入りうるのでO(n)。
    - Noneをappend前に弾くとしたら、O(height)。height <= n/2より結局O(n)。

```python
from typing import Optional


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        stack = [root]
        while stack:
            node = stack.pop()
            if node is None:
                continue
            node.left, node.right = node.right, node.left
            stack.append(node.left)
            stack.append(node.right)
        return root
        
```

- 3分かかった。
- 振り返り
  - 書いている間、特に何も悩まなかった。

## Step 2

- LLMレビューの感想
  - 特になし。
- コメント集：空
- PR
  - https://github.com/ryosuketc/leetcode_grind75/pull/6
  - https://github.com/Ryotaro25/leetcode_first60/pull/71
  - https://github.com/huyfififi/coding-challenges/pull/6
  - https://github.com/rihib/leetcode/pull/26/
  - https://github.com/wf9a5m75/leetcode3/pull/13
  - https://github.com/kzhra/Grind41/pull/6
  - https://github.com/irohafternoon/LeetCode/pull/8

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの学習記録とそれについているコメントをあなたがチェックし、PRごとの内容を簡潔にまとめて欲しいです。


**A:**
各PRの内容を確認しますね。まず全てのPRを読み込みます。

各PRの内容をまとめます：

---

**1. ryosuketc/leetcode_grind75 PR#6** (C++)
- **実装**: C++でiterativeとrecursive両方
- **コメント**:
  - `std::move`とGCについての議論 → C++にはGCがない。スタックの自動解放とGCは別物
  - `nullptr` vs `NULL` → `nullptr`一択（`NULL`は問題を起こすことがある）
  - 日本語個人ブログは信頼性に注意、cpprefjp等の公式リファレンスで裏取りを推奨

---

**2. Ryotaro25/leetcode_first60 PR#71** (C++)
- **実装**: 再帰版とBFS版
- **コメント**:
  - `std::swap`を使ってswap→invertの順でも良い
  - root == nullの早期チェックは好みの問題（統一感の議論）
  - `using namespace std;`を避け、`std::queue`のように明示する方が実務的

---

**3. huyfififi/coding-challenges PR#6** (Python)
- **実装**: DFS(stack)とBFS(queue)
- **コメント**:
  - stackに`None`を入れるか入れないかは好みの範囲
  - ただしBFSでvisitedフラグを出てきた後に立てると急激に遅くなるケースがある
  - `a, b = b, a`はPythonのイディオムとして積極的に使ってよい

---

**4. rihib/leetcode PR#26** (Go)
- **実装**: 再帰版
- **コメント**: odaさんからレビューがあったが詳細は見えず
- ラベル: Grind75, Blind75, NeetCode150, Easy, Recursion, Stack

---

**5. wf9a5m75/leetcode3 PR#13** (Python)
- **実装**: 再帰版とiterativeの複数パターン
- **コメント**:
  - 不要な条件分岐(`if right_tree:`等)は消せる
  - stackに`None`を入れて出てきたらチェック、でもOK
  - ただし`None`を入れないことでループ回数が減る（僅かだが）ので好み
  - 「どちらでもいい」との結論

---

**6. kzhra/Grind41 PR#6** (C++)
- **実装**: BFS(queue)を中心に複数パターン
- **コメント**:
  - 不要なelse節を削除可能
  - `if (root->left)`のような条件なくても動く
  - queueに全部入れてpop後にnullptrチェックする手もある
  - nullptrなら早めに抜けてネストを浅くする提案

---

**7. irohafternoon/LeetCode PR#8** (C++)
- **注意**: このPRは「Valid Parentheses」の問題で、Invert Binary Treeではありません
- **コメント**（参考として）:
  - `using namespace std;`は名前空間を汚すので好まれない
  - `map`に`const`修飾を付けると`[]`ではなく`at`でアクセスが必要
  - `count`より`contains`(C++20)が明示的
  - `emplace` vs `push`の違い（in-place構築 vs コピー/移動）

---

何か特定のPRについて詳しく知りたい点はありますか？

- > ただしBFSでvisitedフラグを出てきた後に立てると急激に遅くなるケースがある
  - これは痛い目を見たことあるので、必ず入れる前にチェックする癖がついている。
  - グラフ系だと入れる前にチェックしてもコードの重複が起きないので心理的に書きやすい。
    ```py
    while frontiers:
        node = frontiers.popleft()
        for neighbor in node.neighbors:
            if neighbor not in visited:
                visited.add(neighbor)
                frontiers.append(neighbor)
    ```

### 実装2

- 一応、append前にNoneチェックの方法。
- append_if_exist()みたいなヘルパー作る気も進まないのでそのまま書く。

```python
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if root is None:
            return None
        
        stack = [root]
        while stack:
            node = stack.pop()
            node.left, node.right = node.right, node.left
            if node.left is not None:
                stack.append(node.left)
            if node.right is not None:
                stack.append(node.right)
        return root
```

## Step 3

[実装1](#実装1)

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装3

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```