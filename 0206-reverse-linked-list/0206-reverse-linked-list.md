## Step 1

一方向連結リストの連結の向きを反転させて返す問題。

- アルゴリズムの選択
  - iterativeとrecursiveが考えられる。
  - 本問においてはiterativeであってもstackを用いてO(n)の空間計算量が必要となる。
  - しかし、recursiveにするとstack overflowやデバッグのしづらさなどの問題が生じる。上、目立ったメリットがない。
  - iterativeで問題ないのでiterativeでやる
- 実装の方針
  - stack内を繋ぎ変えていくとき、番兵より、ループ内で初回分岐する方がノード新規作成が不要なのでこちらを選択。
  - reversed linked listのtail.nextをNoneにするのを忘れないようにする

実装。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        stack = []
        node = head
        while node is not None:
            stack.append(node)
            node = node.next
        head_rev = None
        for i in range(len(stack)):
            node = stack.pop()
            if head_rev is None:
                head_rev = tail_rev = node
            else:
                tail_rev.next = node
                tail_rev = tail_rev.next
            tail_rev.next = None
        return head_rev
```

ここまで6分。

> Follow up: A linked list can be reversed either iteratively or recursively. Could you implement both?

recursiveを実装する。

帰りがけで完成する再帰。（the reversal is performed on the way back.）

また、下流からもらったreversed_tailと上流からもらったheadを入れ替えて、上流に渡すやり方をイメージしたので、これで実装する。

この場合再帰関数は引数は未完成なheadだけでよく、返り値が完成品のtailと、完成品のheadが必要。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return None
        if head.next is None:
            return head
        def reverse_list_recursive(head):
            if head.next.next is None:
                head_reversed = head.next
                head.next = None
                head_reversed.next = head
                return head, head_reversed
            tail_reversed, head_reversed = reverse_list_recursive(head.next)
            head.next = None
            tail_reversed.next = head
            return tail_reversed.next, head_reversed
        tail_reversed, head_reversed = reverse_list_recursive(head)
        return head_reversed
```

行きがけで完成する再帰（the reversal is performed on the way down.）の場合、
- 完成品のheadと未完成品のheadを引数に渡す方法と、
- 受け取ったheadとhead.nextを入れ替え、一つ進めて下流にheadを渡す方法
などが考えられる。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.x5w37bodndgj)
  - > 繋ぎかえる方法は実は3種類あるようです。
    - 気づかなかったが、n-1個のlinkを中心に見て、whileループ1回で繋ぎかえを行う方法もある。
    - これが好みなのでStep 3で採用する。

## Step 3

```python3
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if head is None:
            return None
        left_of_link = None
        right_of_link = head
        while right_of_link is not None:
            next_right_of_link = right_of_link.next
            right_of_link.next = left_of_link
            left_of_link = right_of_link
            right_of_link = next_right_of_link
        return left_of_link
```
