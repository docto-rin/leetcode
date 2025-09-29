## Step 1

linked list を知らなかったため、アルゴリズムイントロダクション「10.2 連結リスト」を参照した。

本問題のリストは一方向であり、set()で訪問済みノードを管理しながら線形探索すれば良さそう。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        node = head
        visited = set()
        while node is not None:
            if node in visited:
                return True
            visited.add(node)
            node = node.next
        return False
```

最初、visited.add(node)をvisited.add(node.val)としたが、誤りに気づき修正。

ここまで7分。

> Follow up: Can you solve it using O(1) (i.e. constant) memory?

全く思いつかないので、GPT-5にヒントを聞いた。

> さて、ご質問の **O(1) メモリで解く方法**ですが、これは「追加データ構造を使わずにポインタだけで検出する」手法が有名です。
>
> ### ヒント
>
> * ノードを2つのポインタで追跡します。
> * 1つは「1歩ずつ」進むポインタ、もう1つは「2歩ずつ」進むポインタ。
> * もしサイクルがなければ、速い方はやがて `None` に到達して終了。
> * サイクルがあるなら、2つのポインタは必ずどこかで出会います。
>
> これは **Floyd の Cycle Detection (Tortoise and Hare)** と呼ばれるアルゴリズムです。

アルゴリズムは多分わかったので、実装。

- 時間計算量: O(n)
- 空間計算量: O(1)

```python3
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        if head is None:
            return False
        node_slow, node_fast = head, head

        def advance(node_slow, node_fast):
            node_slow = node_slow.next
            node_fast = node_fast.next
            if node_fast is not None:
                node_fast = node_fast.next
            return node_slow, node_fast
        
        node_slow, node_fast = advance(node_slow, node_fast)
        while node_fast is not None:
            if node_slow == node_fast:
                return True
            node_slow, node_fast = advance(node_slow, node_fast)
        return False
```

ここまで30分。やりたいことは明確なのに、実装に落とし込むのに苦戦した。しかもコードがわかりにくい。

if node_slow == node_fast:は==かisかどっちがいいのかわからなかった。

https://docs.python.org/ja/3.13/library/stdtypes.html

==は等価性、isは同一性なので、今回はisの方が好ましい。

あと、if head is Noneを書かかずにsubmissionしたらAttributeError: 'NoneType' object has no attribute 'next'が出た。advance関数はNone処理がfastの2歩目にしか行わない設計なのが不自然な気がする。

## Step 2

- https://github.com/Mike0121/LeetCode/pull/37 
  - Floyd's cycle-finding algorithmの実装で、while node_fast and node_fast.next: の方が明瞭かつ無駄がなくていいなと思った。
    - この実装なら、node_fastが1歩目でNoneなら即座に止まる。
    - また、while内にnode_fastのNone処理が集約されているのが明瞭。
    - 一方自分の実装だと、node_fast is Noneをチェックする箇所が3箇所に散在している（あとif ... is not Noneはくどい）。
  - やはり if node_slow is node_fast:の方が好ましい
- https://github.com/smallmonkeykey/arai60/pull/1

## Step 3

`node_slow`, `node_fast`はくどいので`slow`, `fast`に修正。

```python3
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = head
        fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False
```
