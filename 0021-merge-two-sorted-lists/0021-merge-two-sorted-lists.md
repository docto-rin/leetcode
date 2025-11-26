# 21. Merge Two Sorted Lists

- URL: https://leetcode.com/problems/merge-two-sorted-lists/
- Difficulty: Easy
- Tags: Linked List, Recursion
- Notebook: https://share.solve.it.com/d/0ed7224bc57732ba0ae8101d88e30633

## 問題文

<p>You are given the heads of two sorted linked lists <code>list1</code> and <code>list2</code>.</p>

<p>Merge the two lists into one <strong>sorted</strong> list. The list should be made by splicing together the nodes of the first two lists.</p>

<p>Return <em>the head of the merged linked list</em>.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>
<img alt="" src="https://assets.leetcode.com/uploads/2020/10/03/merge_ex1.jpg" style="width: 662px; height: 302px;" />
<pre>
<strong>Input:</strong> list1 = [1,2,4], list2 = [1,3,4]
<strong>Output:</strong> [1,1,2,3,4,4]
</pre>

<p><strong class="example">Example 2:</strong></p>

<pre>
<strong>Input:</strong> list1 = [], list2 = []
<strong>Output:</strong> []
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre>
<strong>Input:</strong> list1 = [], list2 = [0]
<strong>Output:</strong> [0]
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li>The number of nodes in both lists is in the range <code>[0, 50]</code>.</li>
	<li><code>-100 &lt;= Node.val &lt;= 100</code></li>
	<li>Both <code>list1</code> and <code>list2</code> are sorted in <strong>non-decreasing</strong> order.</li>
</ul>


## Step 1

### 実装1

- アルゴリズムの選択
  - 素直に線形走査する。
  - 片方が末尾までたどり着いたら、あとは一気にもう片方の残りの先頭をくっ付ければ終わる。
    - 時間計算量の節約になりうる。
- 実装
  - iterativeかrecursiveだが、前者で容易に書けるので前者で書く。
  - 番兵を使う方がシンプルに書けそうだが、どちらでも良いだろう。
    - コーディング負荷を高めるために（練習目的で）番兵は使わずに書く。
  - 問題文より、入力を破壊して（in-placeに）mergeする。
- 計算量
  - list1.length = n1, list2.length = n2とおいて、
  - Time: O(min(n1, n2))
  - Extra Space: O(1)

```python
from typing import Optional


# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        if list1 is None and list2 is None:
            return None
        if list1 is None:
            return list2
        if list2 is None:
            return list1
        
        node1 = list1
        node2 = list2
        head = None
        while node1 is not None and node2 is not None:
            if node1.val <= node2.val:
                to_add = node1
                node1 = node1.next
            else:
                to_add = node2
                node2 = node2.next
            
            if head is None:
                head = tail = to_add
            else:
                tail.next = to_add
                tail = tail.next
        
        if node1 is None:
            tail.next = node2
        else:
            tail.next = node1
        return head
```

- 振り返り
  - 9分弱かかった。Arai 60を終えてから少し日数が経ったので感覚が抜けてきている。
  - 最初のtest runでエラーになってしまった。
    - list1, list2のいずれかが最初からNoneのとき、whileがskipされtailが未定義のまま.nextを参照してしまった。
    - それらのケースはearly returnし、whileが最低1回はまわるように書くと収まりのいいコードになった。
  - while後の後処理によりtailが実際の末尾でなくなってしまうが、すぐにreturn headするのでそこまで違和感ない。

## Step 2

**Q:** 実装1のコードをレビューして、改善点を教えてください。


**A:**
実装1のレビューです。いくつか改善点があります：

**冒頭のNoneチェック**

3つのif文は1つにまとめられます。`list1`がNoneなら`list2`を返す（`list2`もNoneならNoneが返る）、そうでなければ`list1`を返す：

```python
if list1 is None or list2 is None: return list1 or list2
```

**末尾の残り連結**

`node1 or node2`で残っている方を取得できます：

```python
tail.next = node1 or node2
```

**全体の簡略化案**

```python
def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
    if list1 is None or list2 is None: return list1 or list2
    node1, node2, head = list1, list2, None
    while node1 and node2:
        if node1.val <= node2.val: to_add, node1 = node1, node1.next
        else: to_add, node2 = node2, node2.next
        if head is None: head = tail = to_add
        else: tail.next, tail = to_add, to_add
    tail.next = node1 or node2
    return head
```

主なポイント：
- `or`演算子でNoneチェックを簡潔に
- タプル代入で複数変数を1行で更新
- `is not None`は`and`/`while`の条件では省略可能

番兵なしでもこのくらいコンパクトになります。どう思いますか？

- LLMレビューの感想
  - Noneチェックに関しては、両方Noneは不要だった。
  - 後処理でtail.next = node1 or node2も確かにコンパクトで良い気がする。

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.a32zb9r43ryo)
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1217743406927970304
    - https://github.com/colorbox/leetcode/pull/5
    - > これ、ある方を足しませんか。
      - 確かに。early returnの方に引っ張られて同じ書き方をしてしまったが、最初に自然言語で考えた方法がそのままコードに反映できていなかった。
      - Noneならそうでないもう一方を足す、ではなくNoneでない方を見つけて足す。
    - > こっちのほうが自然では。if list1.val > list2.val: swap(list1, list2)
      - こう書けば同じロジックを2回書かなくて済む。こちらの方が洗練されている。
- https://github.com/ryosuketc/leetcode_grind75/pull/3
- https://github.com/huyfififi/coding-challenges/pull/3
- https://github.com/azriel1rf/leetcode-prep/pull/6
- https://github.com/kzhra/Grind41/pull/3

**Q:** 上記に他の人が解いたものを見てまとめています。

下4つのPRとそれについているコメントをあなたがチェックし、内容を簡潔にまとめて欲しいです。


**A:**
4つのPRの内容とコメントをまとめます：

---

**1. ryosuketc/leetcode_grind75/pull/3**

C++で解こうとしてポインタの扱いに苦戦。主なコメント：
- **ポインタと参照の違い**: `ListNode* a = b` はアドレスの値を渡す「値渡し」であり、「参照渡し」ではない。C++の参照 (`&`) は変数のエイリアス。
- **const の位置**: `const ListNode*` と `ListNode const*` は指す先が定数、`ListNode* const` はポインタ自体が定数。
- **アロー演算子**: `a->b` は `(*a).b` の糖衣構文。
- **Googleスタイルガイド**: `else` は改行しない方が読みやすい。
- **using namespace std**: 大規模コードでは名前衝突を起こすので無闘に使わない方がよい。

---

**2. huyfififi/coding-challenges/pull/3**

iterativeとrecursive両方を実装。主なコメント：
- **変数名**: `curr` より `merged_tail`（マージ済みの最後尾）の方が意図が明確。
- **共通化の是非**: `curr = curr.next` を if/else の外に出して共通化するより、「先頭を取って最後尾に追加」という一連の操作を分岐内にまとめる方が論理的に自然。
- **非破壊的実装**: 新しい `ListNode` を作成して元のリストを変更しない方法も考えられる。
- **番兵なしとの比較**: dummyを使わないと条件分岐が増えて冗長になる。

---

**3. azriel1rf/leetcode-prep/pull/6**

コメントなし。Step 1で7分16秒、Step 2（少し時間をあけて）で2分程度。「まとめられる処理はまとめたい」との振り返り。

---

**4. kzhra/Grind41/pull/3**

C++での実装。主なコメント：
- **メソッド引数の更新を避ける**: 引数を直接変更しない方がよい。
- **変数名**: `dummy_node` より `head` のような先頭を示す名前が直感的（ただし oda さんは「dummy も趣味の範囲」と補足）。
- **swapでelse節を削除**: `if (list1->val > list2->val) swap(list1, list2);` をループ先頭で行えば、else節が不要になりコードがシンプルに。変数名も `smaller`, `bigger` にすると関係性が明示できる。
- **ファイル分割**: ステップごとにファイルを分けるとレビューしやすい。
- **インデント**: 修正が必要。

---

ポインタと参照の違い、のトピックについてLLMにPythonではどうか聞いたところ、

> Pythonは「すべてがオブジェクト参照」という一貫したモデルなので、C++のポインタ/参照の区別より単純ですが、mutable/immutableの違いを意識することが重要です。

とのことだった。概ね同じ認識。全ては参照渡しとみなせ、ダブルポインタなどはmutableのmutationにポインタを忍ばせることで擬似的に実現できる。

### 実装2

- リファクタ内容
  - 番兵を使っても書いてみる。
  - swapを使う。

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:        
        smaller = list1
        bigger = list2
        dummy = tail = ListNode()
        while smaller is not None and bigger is not None:
            if smaller.val > bigger.val:
                smaller, bigger = bigger, smaller
            tail.next = smaller
            tail = tail.next
            smaller = smaller.next
        
        tail.next = bigger or smaller
        return dummy.next
```

- これはいいなぁと思った。選択肢として使えるようになりたい。

## Step 3

[実装2](#実装2)

## Step 4

レビューコメント：https://github.com/docto-rin/leetcode/pull/62

**Q:** それぞれについて、もう少し詳しく教えて。


**A:**
それぞれの論点について詳しく説明します。

---

**1. 認知負荷**

`if list1 is None or list2 is None: return list1 or list2` を読むとき、頭の中で以下を確認する必要があります：

| list1 | list2 | 条件部 | 返り値 |
|-------|-------|--------|--------|
| None | None | True | `None or None` → None ✓ |
| None | X | True | `None or X` → X ✓ |
| X | None | True | `X or None` → X ✓ |
| X | X | False | （通過） ✓ |

一方、分けて書くと：

```python
if list1 is None: return list2
if list2 is None: return list1
```

各行が「list1がNoneなら、list2を返す」「list2がNoneなら、list1を返す」と自然言語で読めます。組み合わせを考える必要がなく、各行が独立して理解できます。

---

**2. `__bool__` 問題**

Pythonの `or` は「最初のtruthyな値」を返します：

```python
0 or 5      # → 5 (0はfalsy)
"" or "hi"  # → "hi" (空文字はfalsy)
None or []  # → [] (Noneはfalsy)
```

問題は、カスタムクラスが `__bool__` を定義している場合です：

```python
class ListNode:
    def __init__(self, val=0, next=None): self.val, self.next = val, next
    def __bool__(self): return self.val != 0  # val=0ならFalse扱い

node_zero = ListNode(0)
node_five = ListNode(5)

node_zero or node_five  # → node_five（node_zeroがFalse扱いされる！）
```

本来 `node_zero` を返してほしいのに、`val=0` だと `False` 扱いになり `node_five` が返されてしまいます。

LeetCodeの `ListNode` は `__bool__` を定義していないので大丈夫ですが、レビュー時に「このクラスは `__bool__` を定義していないか？」と確認する手間が生じます。

安全に書くなら：

```python
tail.next = bigger if bigger is not None else smaller
```

または：

```python
if bigger is not None: tail.next = bigger
else: tail.next = smaller
```

---

**3. 言語間の移植性**

Pythonの `or` は「値を返す」：

```python
a or b  # aがtruthyならa、そうでなければb
```

C++の `||` は「bool値を返す」：

```cpp
a || b  // true または false
```

そのため、C++で同様のことをするには三項演算子を使います：

```cpp
tail->next = node1 ? node1 : node2;
```

Javaも同様に `||` は boolean を返すので：

```java
tail.next = (node1 != null) ? node1 : node2;
```

面接で「普段はPythonだけどC++で書いてください」と言われた場合、`or` のイディオムがそのまま使えないことを意識しておく必要があります。また、チームに複数言語を使う人がいる場合、Python特有のイディオムは混乱を招く可能性があります。

---

どれも「書ける」ことと「書くべきか」は別、という話ですね。コンテキストに応じて判断するのが良さそうです。

### 実装3

- レビューを受けて書き直し。
  - bigger or smallerは避ける。

```python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:        
        smaller = list1
        bigger = list2
        dummy = tail = ListNode()
        while smaller is not None and bigger is not None:
            if smaller.val > bigger.val:
                smaller, bigger = bigger, smaller
            tail.next = smaller
            tail = tail.next
            smaller = smaller.next
        
        if bigger is not None:
            tail.next = bigger
        else:
            tail.next = smaller
        return dummy.next
```