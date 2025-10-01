## Step 1

値でソート済みの連結リストについて、重複する値をもつノードを1つ残さず削除する問題。

まず関数の返り値を考えると、入力のheadノードが必ずしも残らないので、return headとは書けない。

headより前に仮ノードdeleted_headを定義し、return deleted_head.nextとすることにした。

deleted_tailより一つ先、仲間にするかどうか悩んでいる候補をnodeとし、node.next is None or node.next.val != node.valならnodeを仲間にする、という方針。

実装します。

- 時間計算量: O(n)
- 空間計算量: O(1)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def deduplicate(node):
            while node.next is not None and node.next.val == node.val:
                node.next = node.next.next
            return node.next
        deleted_head = deleted_tail = ListNode(None)
        node = head
        while node is not None:
            if node.next is None:
                deleted_tail.next = node
                deleted_tail.next.next = None
                return deleted_head.next
            elif node.next.val != node.val:
                deleted_tail.next = node
                deleted_tail = deleted_tail.next
                node = node.next
            else:
                node = deduplicate(node)
                deleted_tail.next = node
        return deleted_head.next
```

ここまで23分。

変数を過剰に定義した（deleted_tail, node）のと、ヘルパー関数がどこまでやるか、ポインタの付け替えなどに悩み時間を消費した。

それに、まだどうみても冗長な付け替えなどが残っている。

この実装では、deleted_tailとnodeという2変数を考えていたが、候補の置き場所をnodeではなくdeleted_head.nextにしておけばシンプルだったなと思った。

書き直してみる。変数名を変更：

- deleted_head -> pre_head
- deleted_tail -> tail
- node -> 削除 (tail.nextを使う)
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def deduplicate(node):
            while node.next is not None and node.next.val == node.val:
                node.next = node.next.next
            return node.next
        pre_head = tail = ListNode(None)
        tail.next = head
        while tail.next is not None and tail.next.next is not None:
            if tail.next.val != tail.next.next.val:
                tail = tail.next
            else:
                tail.next = deduplicate(tail.next)
        return pre_head.next
```

書きたかったのはこれ。

ポインタは、なるべく根元で更新した方がよかった。（付け替えの回数を減らすため）

ここまでleetcodeを4問解いてきて、自分は変数を過剰に定義する癖があることに気づいた。

これに起因してロジックも冗長になっていたりする。

変数を多めに定義すること自体、常に悪いわけではないと思うが、ちゃんと書き始める前に必要か見極めたい。

早く動くコードを書かないといけないという焦りがよくない気がする。

## Step 2

- https://github.com/hiroki-horiguchi-dev/leetcode/pull/4
  - 似た解き方。
  - dummy.valにマジックナンバーがある。自分はNoneにしたが、これも広義ではリテラルか。
- https://github.com/yas-2023/leetcode_arai60/pull/4
  - いろいろやり方を検討されていて素晴らしい。
    - stack, single loop, double loop, 自分はdouble loop
    - stackもsingle loopも、1つのwhileでノードを1個ずつ動かしている。
    - stack、もちろん空間計算量は犠牲になるがが、わかりやすいのがとてもいいですね。
      - 思いつきたかったという気持ちになリます。
    - single loopの実装について、
      ```python3
      if A:
        X
        contiunue
      if B:
        Y
        contiunue
      Z
      ```
      は、if-elif-elseで書きたいと感じた。
      - https://discord.com/channels/1084280443945353267/1195700948786491403/1197102971977211966
      - ちゃんと書き直されていた。
- https://github.com/kazizi55/coding-challenges/pull/4
  - memo.mdに、他人の回答を選択肢として細かく検討する姿勢が見え、いいと思った。

## Step 3

stack
- 時間計算量: O(n)
- 空間計算量: O(n)
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def build_stack(head):
            if head is None:
                return []
            stack = []
            prev = ListNode(None)
            node = head
            while node.next is not None:
                if prev.val != node.val != node.next.val:
                    stack.append(node)
                prev = node
                node = node.next
            if prev.val != node.val:
                stack.append(node)
            return stack
        stack = build_stack(head)
        if stack == []:
            return None
        for i in range(len(stack)-1):
            stack[i].next = stack[i+1]
        stack[-1].next = None
        return stack[0]
```
single loop
- 時間計算量: O(n)
- 空間計算量: O(1)
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy = tail = ListNode(None)
        curr_val = None
        node = head
        while node is not None:
            if node.val == curr_val:
                # 重複区間中
                pass
            elif node.next is not None and node.next.val == node.val:
                # 重複区間開始
                curr_val = node.val
            else:
                # 単独
                curr_val = node.val
                tail.next = node
                tail = tail.next
            node = node.next
        tail.next = None
        return dummy.next
```
double loop
- 時間計算量: O(n)
- 空間計算量: O(1)
- Step 1で書き直したときはnode変数を削除したが、やっぱり主役なので変数として存在した方がいいと判断。
```python3
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def _delete_duplicates(node):
            while node.next is not None and node.next.val == node.val:
                node = node.next
            return node
        dummy = tail = ListNode(None)
        node = head
        while node is not None:
            if node.next is None or node.next.val != node.val:
                # 単独
                tail.next = node
                tail = tail.next
            else:
                # 重複区間開始
                node = _delete_duplicates(node)
            node = node.next # 常に node.val != node.next.val
        tail.next = None
        return dummy.next

```
補足：
- double loopの方法では、外側のwhileに1回ずつ入るとき、状態としては重複区間開始か、単独の2パターンしか存在しない。
- ヘルパー関数_delete_duplicates()により、重複区間が終わるまでnodeが前進する。
