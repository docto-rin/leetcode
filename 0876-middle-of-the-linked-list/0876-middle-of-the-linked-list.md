# 876. Middle of the Linked List

- URL: https://leetcode.com/problems/middle-of-the-linked-list/
- Difficulty: Easy
- Tags: Linked List, Two Pointers
- Notebook: https://share.solve.it.com/d/bbd3858b3d2b5f284f160661b6a4c20d

## Step 1

### 実装1

- アルゴリズムの選択
  - linked listの長さがわからないと始まらないので線形走査1周は必須。
  - 1周すると、全体の長さとmiddle nodeが何番目かが分かる。
  - middle nodeを見つける方法は2つある。
    - 1周目で全てのnode（の参照）をlistで保持しておき、インデックスアクセスして得る。
    - 再びheadから2周目を行い、middle nodeまでたどり着いたら立ち止まる。
  - 空間計算量と時間計算量のトレードオフだが、総合したら基本的には後者が好ましそう。
    - 時間計算量の節約は定数倍だが、空間計算量の節約はO(n) -> O(1)とオーダーが変わるため。
- 実装
  - single loopを2個、逐次的に行う。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python
from typing import Optional


# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        num_nodes = 0
        node = head
        while node is not None:
            num_nodes += 1
            node = node.next
        
        node = head
        for _ in range(num_nodes // 2):
            node = node.next
        return node
```

- 振り返り
  - 特になし。

## Step 2

**Q:** Step 1のコードをレビューして、改善点を教えてください。


**A:**
コードを拝見しました。ロジックは正しく、計算量の分析も的確です！

一点だけ質問させてください：

現在の実装ではリストを**2回走査**していますね（1回目で長さを数え、2回目で中間ノードまで移動）。これを**1回の走査**で実現する方法は思いつきますか？

ヒント：2つのポインタを使い、それぞれ異なる速度で進めるとしたら…？

### 実装2

- アルゴリズムの改善
  - tortoise and hare（カメとウサギ、Floyd's algorithm）を使うことで[実装1](#実装1)と比べ時間計算量をおよそ2/3倍にできる。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python
class Solution:
    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        fast = slow = head
        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next
        
        return slow

```

- コメント集：空
- PR
  - https://github.com/huyfififi/coding-challenges/pull/22
  - https://github.com/rihib/leetcode/pull/40
  - https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/37

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。
resolvedなどで中身が十分に確認できない場合、tool callingに時間がかかっても大丈夫なので分かるまで丁寧に調査してください。


**A:**
それぞれのPRを確認しますね。少々お待ちください。

PRページの概要は取得できましたが、具体的なコードを確認するためにファイルの変更を見る必要がありそうです。特にrihib/leetcodeのPRはレビューコメントがresolvedになっているので詳細を確認します。

各PRの情報が揃いました。以下にまとめます。

---

**PR 1: huyfififi/coding-challenges#22** (Python)
- **解法**: 2つのアプローチを実装
  1. **Fast & Slow (Tortoise and Hare)**: LeetCode 141で使ったFloydのアルゴリズムから着想。slowは1つずつ、fastは2つずつ進み、fastが末尾に着いたらslowが中間。
  2. **長さを測る方法**: 1周目でリストの長さを測り、2周目で`length // 2`だけ進む。
- **レビューコメント**: なし（セルフマージ）
- **メモ**: 変数名`move_count_to_middle`の代替案を考えたが良い案が思いつかなかったと記載。

---

**PR 2: rihib/leetcode#40** (Go)
- **解法**: 同様に2つのアプローチ
  1. **Step 1-2**: Fast & Slow法。最初は`head == nil`の早期リターンを入れていたが、不要と気づき削除。
  2. **Step 3**: 長さを数える方法（他の方の解法を見て実装）
- **主なレビューコメント**:
  - **@oda**: step1の早期リターンについて言及（resolvedで詳細不明）
  - **@oda**: step3について「この方法を先に思いつかなかったのが逆に良くない」という本人のコメントに対して何かコメント（resolved）
  - **@liquo-rice**: 「良さそうです」

---

**PR 3: NobukiFukui/Grind75-ProgrammingTraining#37** (Python) ← 最も議論が活発
- **解法**: 
  1. **1st**: 長さを数えて中間まで進む方法（関数を分離）
  2. **2nd**: Fast & Slow法（LeetCodeのSolutionsを参照）
  3. **3rd**: レビューを反映した改善版
- **主なレビューコメント**:
  - **@Mike0121**: `int(max_node_count / 2 + 1)` → `max_node_count // 2 + 1` で型変換を避ける
  - **@Mike0121**: `node_count = 1`が1-indexedなら、コメントがあると良い
  - **@TORUS0818**: `node_count`を外から与えるのではなく関数内で定義すべき
  - **@TORUS0818**: 関数名はPEP8に従い`snake_case`にすべき（`checkNodeNumber` → `check_node_number`）
  - **@TORUS0818**: `end_node`という変数名は実態と合っていない、`num_proceed`が良い
  - **@TORUS0818**: while条件の`node_count`が機能していない、ループ内で減らしていく実装を提案
  - **@oda**: 長さを数える方法が標準的な解答で良い
  - **@Kitaken0107**: 関数に分けない方がスッキリ書ける場合も（ただし変数スコープの観点では関数化に利点あり、と@odaがフォロー）

---

**総括**: 3つのPR全てで「Fast & Slow法」と「長さを測る方法」の2つが議論されています。命名規則、1-indexed/0-indexed、関数分離の是非などが共通の論点でした。

## Step 3

### 実装3

- 何も見ずに書き直し
- ナイーブな方を書く。

```python
class Solution:
    def get_length(self, head: Optional[ListNode]) -> int:
        num_nodes = 0
        node = head
        while node is not None:
            num_nodes += 1
            node = node.next
        return num_nodes

    def middleNode(self, head: Optional[ListNode]) -> Optional[ListNode]:
        num_nodes = self.get_length(head)
        node = head
        for _ in range(num_nodes // 2):
            node = node.next
        return node
```

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装4

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```