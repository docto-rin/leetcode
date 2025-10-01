## Step 1

valueがソートされた一方向連結リストをdeduplicateする問題。

変数は、

- node: 今いるノード
- level: deduplicatedで一番大きい値
- deduplicated: deduplicateされた連結リスト

があればよさそうか？

whileループ内にて、

```python3
deduplicated.next = ListNode(val=node.val)
```

と書いたが、これだと2回目以降は.next.next, ...となってしまう。

どうやったら繰り返せるように書けるだろうかと考えた。

無意識にheadを主役にしていることに気づいた。新しいポインタ先をtail変数として逐次更新すれば、以降も常にポインタ元はtail.nextと書ける。

実装します。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return None
        node = head
        level = head.val
        deduplicated = ListNode(val=head.val)
        tail = deduplicated
        while node is not None:
            if node.val != level:
                level = node.val
                tail.next = ListNode(val=node.val)
                tail = tail.next
            node = node.next
        return deduplicated
```

Acceptedされたが、Runtime、Memoryともに受験者の中で下位20パーセントくらいだった。なぜだろう...

ここまで9分。

Follow upはないらしいので、終わりにする。

## Step 2

- https://github.com/TrsmYsk/leetcode/pull/3
  - 破壊的にポインタを繋ぎかえるという発想がなかったです。
    - 自分の実装は非破壊的。値の見た目は同じなのでacceptedにはなったが、入力のノードとは別にインスタンスを新規作成している。
    - 問題文を読み直すと、破壊的にポインタを繋ぎかえる方が自然に思えてきた。
  - 変数は1つで良かったんですね。
    - ポインタを繋ぎかえるという発想でdeduplicated変数（返り値用のリスト）は消せる。
    - また、levelはよく考えたらtail.valで表せるので不要。
  - while二重ループにすることでlevel変数を消せる上、level変数よりわかりやすいと感じました。
  - > 問題文に記載はなかったがノードの値はおそらくint型なので値の比較は == で行った。
    - 自分も同じ考えで!=を使いました。
  - > 重複チェックの対象になるノードをforwardという変数に格納しようかとも思ったが、読みやすさは変わらなそうなのでやめた。
    - 自分の実装でいうlevel変数と似た発想でしょうか。自分は今回は変数を増やさない方がいいと思いました。
  - Step 2 -> Step 3で内側のwhile条件2個目がnode.nextを主役にした書き方に改善されていて、いいなと思いました。

ここまでで、自分の実装は下記のように簡素化できる。変数の対応は、

- node -> frontier
- level -> 削除
- deduplicated -> 削除
- tail -> tail

```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return None
        frontier = tail = head
        while frontier is not None:
            if frontier.val != tail.val:
                tail.next = frontier
                tail = tail.next
            frontier = frontier.next
        tail.next = None
        return head
```
これであれば少なくともノードの新規作成はしてないのでして無いので禁忌は踏んでなさそう。

あとは空間計算量をO(1)に減らせている。

- https://github.com/hiroki-horiguchi-dev/leetcode/pull/3
  - すぐ上の実装に似ている。
  - 変数名の対応はfrontier->next, tail->currentですね。
- https://github.com/yas-2023/leetcode_arai60/pull/3
  - 再帰を検討されていて素晴らしい。
    - 本問ならリスト長は高々300なので、全部値が同じでもスタックオーバーフローは起きなさそう。
      - https://docs.python.org/ja/3.13/library/sys.html#sys.setrecursionlimit
        > limit の最大値はプラットフォームによって異なります。深い再帰処理が必要な場合にはプラットフォームがサポートしている範囲内でより大きな値を指定することができますが、この値が大きすぎればクラッシュするので注意が必要です。
    - https://discord.com/channels/1084280443945353267/1261259843671687188/1262997002891821056
      > 前の人から数字を教えてもらうか、後ろの人から数字を教えてもらうか、なんらかの情報がないと仕事ができないですね。
      - この例えで行くとyas-2023-sanの方法は、後ろの人から1つ数字を教えてもらい、自分でも1つ数字を見て比較し、deduplicateするか決めて前の人に完成品を渡している。
  - > c/c++系とpythonで演算子の優先順位が異なるケースがあり、痛い目を見たのでそれ以来、ミス防止のために()で括るように癖づけています。
    - 大事ですね。勉強になります。

## Step 3

while二重ループ
- 空間計算量: O(1)
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        while node is not None:
            while node.next is not None and node.next.val == node.val:
                node.next = node.next.next
            node = node.next
        return head
```

再帰（nextと比較）
- 空間計算量: O(n)
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None or head.next is None:
            return head
        head.next = self.deleteDuplicates(head.next)
        return head if head.val != head.next.val else head.next
```

再帰（prevと比較）
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return head
        def _deleteDuplicates(head, prev_val):
            if head is None:
                return head
            head.next = _deleteDuplicates(head.next, head.val)
            return head if head.val != prev_val else head.next
        head.next = _deleteDuplicates(head.next, head.val)
        return head
```
