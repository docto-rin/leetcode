## Step 1

前回の問題 linked-list-cycle で、返り値が複雑になった問題。
ひとまずset()で訪問済みを管理するやり方なら返り値を変えるだけでいいので、実装。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        visited = set()
        while node is not None:
            if node in visited:
                return node
            visited.add(node)
            node = node.next
        return None
```

... is not Noneは暗黙的falseを嫌った書き方。

ここまで3分。

さて、予想できていたが、以下、
> Follow up: Can you solve it using O(1) (i.e. constant) memory?

Floydの方法の拡張だと思うが、脳内シミュレーションをしても、出会う場所はリストの長さとサイクルの長さによってまちまち。
わからないので、GPT-5にヒントをもらう。

> * まず **slow**（1歩ずつ）と **fast**（2歩ずつ）を動かすと、ループがある場合いつか同じ場所で会う。
> * その後、**一方を head に戻して、両方を 1 歩ずつ進める**と、次に会う場所がループの入口になる。

おそらく、リストの長さとサイクルの長さを文字で置いて証明できる気がする。

ひとまずはこのアルゴリズムを実装する。

- 時間計算量: O(n)
- 空間計算量: O(1)

```python3
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = head
        fast = head
        meet = None
        late = head
        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next
            if fast is slow:
                meet = fast
                break
        if meet is None:
            return None
        while late is not meet:
            late = late.next
            meet = meet.next
        return late
```

変数名、meetはいいけどlateはこれでよかったのかな、という気持ちが残る。

restartとかのがいいのか。でも本人はrestartしてないので違和感があり、遅刻的なニュアンスでlateにした。

でもlateは遅刻というよりlatestのニュアンスで取られそう。他の方がどう命名されたか気になる点。

ここまで16分。

## Step 2

- https://github.com/TrsmYsk/leetcode/pull/2 
  - Floydの方法の前半、whileの条件に、fastのNoneチェックを集約させた方が良さそう。
  - Floydの方法の後半、meet, lateの変数名に、from_start, from_meetingとしていて、fromが位置を元に命名していることを明確にしていていいなと思った。
  - ネストが深くなることを躊躇されていて、素敵だなと思った。
- https://github.com/yas-2023/leetcode_arai60/pull/2
  - Floydの方法の後半、変数にslow, fastを使い回すと、何しているかが一見してわかりにくいという印象を持った。
- https://github.com/Kaichi-Irie/leetcode-python/pull/20
  - Floydの方法の前半と後半で、変数の初期化を直前に行なっているのが、とてもわかりやすいなと思った。
    - 自分の実装は、最初に4つ全てを初期化しているが、読み手のワーキングメモリを食わせてしまう。
    - 自分の実装で、if fast is slow:が成り立つ瞬間meet=fastと慌てて代入しているが、別にfast（の位置）はすぐに失われないので、ここはフラグを保持すれば十分。

## Step 3

```python3
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        fast = head
        slow = head
        has_cycle = False
        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next
            if fast is slow:
                has_cycle = True
                break
        if not has_cycle:
            return
        
        from_start = head
        from_meeting = fast
        while from_start is not from_meeting:
            from_start = from_start.next
            from_meeting = from_meeting.next
        return from_start
```

no cycleでNoneを返すところ、冗長だと思ったので、
```python3
if not has_cycle:
    return
```
としてみました。
