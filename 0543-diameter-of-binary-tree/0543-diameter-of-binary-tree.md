# 543. Diameter of Binary Tree

- URL: https://leetcode.com/problems/diameter-of-binary-tree/
- Difficulty: Easy
- Tags: Tree, Depth-First Search, Binary Tree
- Notebook: https://share.solve.it.com/d/97edf75b8b065147544543f0b244941f

## Step 1

### 実装1

- アルゴリズムの選択
  - 左右の部分木から、diameterとdepthを受け取れば、全体のdiameterとdepthを計算できる。
  - よって、bottom-up（帰りがけに完成）する再帰でできる。
  - これはiterativeで書くとダブルポインタとフラグ管理が必要になり複雑になるタイプなので、一旦書きやすいrecursiveで書く。
  - top-downやinorderの計算順序だと思いつかない。
- 実装
  - 返り値にdepthも必要なので、再帰関数はinner functionとして書く。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
from typing import Optional


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        def get_diameter_and_depth(node):
            if node is None:
                return 0, 0
            
            left_diameter, left_depth = get_diameter_and_depth(node.left)
            right_diameter, right_depth = get_diameter_and_depth(node.right)

            diameter = max(
                left_diameter, right_diameter, left_depth + right_depth
                )
            depth = max(left_depth, right_depth) + 1
            return diameter, depth
        
        return get_diameter_and_depth(root)[0]
```

- 6分で書けた。
- 振り返り
  - 再帰関数に頼ってしまったものの、割とスムーズに引き継ぎの方針を立てることができた。
  - 次にiterativeで書きたい。

### 実装2

- 実装の検討
  - iterativeで書く
  - 部分木からの引き継ぎの置き場を空リスト（または任意のmutableなオブジェクト）とする。
  - 引き継ぎ結果がない場合は引き継ぎの依頼を出し、左右ともに終わっていたら両者からの引き継ぎをもとに自分が仕事をする。
  - ただし、自分がNoneノードの場合、引き継ぎ依頼を出す必要がなく、即座に仕事をする。

```python
from typing import Optional


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        result = []
        stack = [(root, [], [], result)]
        while stack:
            node, left_output, right_output, output = stack[-1]

            if node is None:
                output[:] = 0, 0
                stack.pop()
                continue
            
            if not left_output:
                stack.append((node.left, [], [], left_output))
            if not right_output:
                stack.append((node.right, [], [], right_output))
            if not (left_output and right_output):
                continue
            
            left_diameter, left_depth = left_output
            right_diameter, right_depth = right_output
            diameter = max(
                left_diameter, right_diameter, left_depth + right_depth
            )
            depth = max(left_depth, right_depth) + 1
            output[:] = diameter, depth
            stack.pop()

        return result[0]

```

- 10分で書けた。
- 振り返り
  - 迷いなく書けるようになっており成長を感じた。
  - これまでの経験的には他の計算順序でもできそうだが、Step 2に回す。

## Step 2

- LLMレビューの感想
  - 確かに、depthは普通rootを0 or 1として下に行くほど増えていく概念だった。heightの方が読み手の驚きが少なそう。
  - diameterはmaxにしか興味がなく、全てのノードで自分を通るdiamterを計算するというのを利用してnonlocalでmax更新するのはわかりやすいかも。
  - 調べたところ確かにoutput[:] = 0, 0は標準的でなく、output[:] = [0, 0]が標準のようだ。
    - もしくはoutput.extend([0, 0])やoutput += [0, 0]など。
    - よく考えるとリストの長さが変わっている（拡張している）のでスライス代入よりextendや+=の方が自然に思えてきた。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.okuct9g1tj6)
  - https://discord.com/channels/1084280443945353267/1192728121644945439/1244684428064591974
    - > 複数回呼んだ場合に結果が意図しない、のは、絶対に駄目というわけではないのです。...ただ、これくらい単純なものだと、しない選択肢もあるだろうに、何を代償にわざわざそうしたんだろう、という感覚です。
    - ステートフルで書く場合に生じる問題。初期化を丁寧にやるか、ステートレスで書くべき。
      - 元コードの書き手の心理はおそらく、「diameterをnonlocal的にmax更新したいが、クラス属性でやったら書きやすそう（？）」と推測.
      - CQS原則から、getterは状態をむやみに変えるべきでないと感じる。
    - またステートレスで十分そうなのにステートフルで書く場合、読み手はそこに意図を求める。
- PR
  - https://github.com/Kitaken0107/GrindEasy/pull/17
  - https://github.com/huyfififi/coding-challenges/pull/21
  - https://github.com/Jikuhara/LeetCode/pull/5
  - https://github.com/rihib/leetcode/pull/39
  - https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/36

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
各PRの内容をまとめました：

---

**1. Kitaken0107/GrindEasy#17** (Python)
- **実装**: `self.diameter`をインスタンス変数として保持し、再帰で更新
- **レビューコメント**:
  - `,`の後にスペースを入れる（PEP8）
  - **参照透過性の問題**: `diameterOfBinaryTree()`を複数回呼ぶと結果がおかしくなる。シンプルな処理でわざわざステートフルにする意図が不明
  - 関数名`depth`は`get_depth()`や`traverse()`がより適切（副作用があるため）
  - 変数名は省略せずフルスペルで

---

**2. huyfififi/coding-challenges#21** (Python)
- **実装**: recursive版とiterative版の両方を実装
- **レビューコメント**: なし（セルフマージ）

---

**3. Jikuhara/LeetCode#5** (C言語)
- **実装**: 再帰でdepthを計算しながらポインタ経由でdiameterを更新
- **レビューコメント**:
  - 変数宣言はスコープ先頭でなく、使用箇所で（C99以降）
  - 変数名は原則フルスペル（スコープが短ければ1文字可）
  - `return (0);`の括弧は不要（古いCの名残）
  - LeetCodeの計測時間は誤差が大きいので当てにしない
  - 「計算量」と「計算時間」の用語の使い分け
  - ファイル末尾は改行で終える

---

**4. rihib/leetcode#39** (Go)
- **実装**: depthを返しながらdiameterをポインタで更新する再帰
- **レビューコメント**:
  - nodchip: 「よいと思います」（approve）
  - oda: 「ペアを返す再帰にすると線形時間になる」パターンの参照リンクを共有

---

**5. NobukiFukui/Grind75-ProgrammingTraining#36** (Python)
- **実装**: Kitakenさんの実装を参考にtraverse関数でself.diameterを更新
- **レビューコメント**:
  - 関数内の先頭に空行は不要（視認性の観点）
  - `traverse`という関数名でdepthを返すのは違和感 → `calculate_diameter_and_depth`のようにペアで返す案
  - 再帰関数でreturnがない分岐があるとNoneになる問題の指摘
  - pdbやpudbでデバッグすることを推奨
  - 「ペアを返すと計算量O(n)になる」という理解を確認

---

**共通して出てきたポイント**:
1. **ペアで返す**: (diameter, depth)をペアで返すと線形時間で済む
2. **参照透過性**: ステートフル（self.diameter）にすると複数回呼び出しで問題が起きる
3. **関数名**: 副作用がある場合は`traverse`、返り値の意味が明確なら`calculate_xxx`など

### 実装3

- リファクタ内容
  - [実装2](#実装2)だが、Noneチェックをstackに入れる前にする方法もある。
    - 実行時間の定数倍最適化ができる。（ループが回る回数がおよそ1/2になる。）
  - output[:] = 0, 0はoutput += [0, 0]に変更。

```python
from typing import Optional


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        
        result = []
        stack = [(root, [], [], result)]
        while stack:
            node, left_out, right_out, output = stack[-1]
            
            if not left_out and node.left is not None:
                stack.append((node.left, [], [], left_out))
                continue
            if not right_out and node.right is not None:
                stack.append((node.right, [], [], right_out))
                continue
            
            if node.left is None:
                left_out += [0, 0]
            if node.right is None:
                right_out += [0, 0]
            
            left_diameter, left_height = left_out
            right_diameter, right_height = right_out
            diameter = max(
                left_diameter, right_diameter, left_height + right_height
            )
            height = max(left_height, right_height) + 1
            output += [diameter, height]
            stack.pop()
        
        return result[0]
```

- 振り返り
  - left_out += [0, 0]はどこに書くのがいいんだろう。
    - if not left_out: のところにさらに条件分岐を追加するのが自然だが、ネストが深くなり嫌った。
    - また、if node.left is None: left_out += [0, 0]してからif not left_out:を見る方法もあるが、ややパズルっぽい気がした。
    - しかしながら、今の書き方だとcontinue系より下に到達した時に、未だleft_outなどが空な場合が残っているという気持ち悪さがある。

### 実装4

- アルゴリズムの検討
  - max_heightをmax更新をする方法。
    - 各ノードを最大の高さとして通るような場合分けを行っていることに相当し、少し無駄が減っている。
  - 子の仕事を依頼する操作をヘルパー関数として切り出し、left_out += [0, 0]なども内包させる。
    - [実装3](#実装3)で感じていたleft_out += [0. 0]のタイミングの不自然さとネストのトレードオフを両方解決する妙案。

```python
from typing import Optional


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        max_diameter = 0
        stack = []
        
        def push_job(node, height_pointer):
            if node is None:
                height_pointer += [0]
                return
            stack.append((node, [], [], height_pointer))
        
        push_job(root, [])
        while stack:
            node, left_height, right_height, height = stack[-1]
            
            if not left_height:
                push_job(node.left, left_height)
                continue
            if not right_height:
                push_job(node.right, right_height)
                continue
            
            max_diameter = max(max_diameter, left_height[0] + right_height[0])
            height += [max(left_height[0], right_height[0]) + 1]
            stack.pop()
        
        return max_diameter
```

## Step 3

[実装4](#実装4)

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