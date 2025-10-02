## Step 1

2つの数字が下の桁からlinked list (l1, l2) で渡される。算術和を同じくlinked listで返す問題。

- l1とl2の長さは必ずしも一致しないので、Noneになるタイミングが異なる。
- 1桁ずつ処理するのが妥当そう。whileか再帰でできる。
- 再帰の方が実装がイメージしやすいと思ったので再帰でやる。
  - l1, l2の両方がNoneになったら再帰の底とする。一方がNoneなだけであれば、val=0を当ててやる。
  - 1桁の足し算だが、繰り上がりも考慮して3つの変数を足すことになりそう。
- あとは、l1とl2が表す数字を先にint型で表すことを目指し、足し算を1回で済ます方法もある。
  - しかし、memoryを食うのと、算術和をlinked listに直すためにもう一度ループを回すのが非効率。
  - 返り値がlinked listではなくintだった場合、ようやく選択肢に入れていいという温度感。

再帰で実装します。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        def set_val_and_next_node(node):
            if node is None:
                return 0, None
            else:
                return node.val, node.next
        def add_two_single_numbers(node1, node2, carry, l_sum):
            if node1 is None and node2 is None:
                if carry >= 1:
                    l_sum.next = ListNode(carry)
                    return
                return
            val1, next1 = set_val_and_next_node(node1)
            val2, next2 = set_val_and_next_node(node2)
            val_sum = val1 + val2 + carry
            next_carry = val_sum // 10
            val_mod = val_sum % 10
            l_sum.next = ListNode(val_mod)
            l_sum = l_sum.next
            add_two_single_numbers(next1, next2, next_carry, l_sum)
            return
        dummy = l_sum = ListNode(None)
        add_two_single_numbers(l1, l2, 0, l_sum)
        return dummy.next
```

ここまで8分。

## Step 2

- https://github.com/hiroki-horiguchi-dev/leetcode/pull/5
  - whileで書かれている。
  - valがNoneでなかればsumに加える、というロジックはシンプルですね。
    - 対して自分の実装はNoneの場合valは0を当てて、常にvalを加えている。
- https://github.com/yas-2023/leetcode_arai60/pull/5
  - 思いついた方法を全て試されていて素晴らしい。
  - None処理は三項演算子よりは関数化する方が好み。
- https://github.com/kt-from-j/leetcode/pull/5
  - whileで書かれている。
  - None処理を関数化しているが、関数の実装が三項演算子だと分かりやすさに対する期待を少し裏切られるような気持ちになった。
- https://github.com/nanae772/leetcode-arai60/pull/6
  - whileで書かれている。
  - while l1 or l2 or carry:は、いいですね。全体的に暗黙のfalseを上手に活用されている。
- https://discord.com/channels/1084280443945353267/1366423240624439378/1372611463872516096
  - zip_longest(digits_1, digits_2, fillvalue=0)での方法。組み込み関数をうまく使っていて非常に分かりやすい。
  - こちらでも、暗黙のfalseを活用している。
- https://github.com/kazukiii/leetcode/pull/6/commits/a9c7637c616aef971424c999f24d49573541daec
  - dummyを使わないことも可能。
  - ループ内で、headがまだない場合は現在のnodeをheadとする分岐を追加している。
  - return headが分かりやすいのが良さですね。
- https://github.com/yus-yus/leetcode/pull/5#discussion_r1945199144
  - ゼロ埋めする方針であれば、valが0なノードを追加する。
  - sumの計算をl1.val + l2.val + carryと統一的なvalアクセスでかける。良さげ。
- https://discord.com/channels/1084280443945353267/1235829049511903273/1238166135489171466
  - > 行き掛けで答えが完成していくのか（Pass-down）、帰り掛けで答えが完成していくのか（Pass-up）という見方もできますね。
  - 再帰するとき、必ず意識的に選択したいですよね。
  - 自分のStep 1は、行き掛けで完成する。代わりに、dummyが必要となった。
  - 帰り掛け（dummyなし）、pythonで実装してみた。
    ```python3
    class Solution:
        def addTwoNumbers(self, l1, l2):
            def safe_node(node):
                if node is None:
                    return ListNode(0)
                return node

            def next_if_exists(node):
                if node is None:
                    return None
                return node.next
    
            def helper(n1, n2, carry):
                if n1 is None and n2 is None and carry == 0:
                    return None
                a = safe_node(n1)
                b = safe_node(n2)
                total = a.val + b.val + carry
                node = ListNode(total % 10)
                node.next = helper(next_if_exists(n1), next_if_exists(n2), total // 10)
                return node
    
            return helper(l1, l2, 0)
    ```
    - 再帰の底に実質的なdummyが暗黙的にできるので、dummyは不要である。
- [Why use a recursive function instead of an iterative function?
](https://www.reddit.com/r/learnprogramming/comments/jjdcrs/why_use_a_recursive_function_instead_of_an/)
  - > There are a few situations where recursive functions really are the right tool for the job. Maybe going over some of those will be illustrative.
      
    > 1. When you are working with recursive data structures. Like if you're parsing a massive tree structure - you don't know how deep it is, or how many entries there are at each level. Although you may be able to find an iterative way to traverse this, it's way easier to do it recursively.
      
    > 2. For working with asynchronous code. Say for example you have a list of 10 jobs, and you need to run them in that specific order, but asynchronously (e.g. on some worker server). With "recursion" it's quitie simple - you make each job enqueue the next job when it's finished. It's more difficult to do iteratively though. You would have to essentially use a "polling" technique where you pause and check repeatedly to see which job is done - not only inefficient - but unwieldy as well
  - 本問含め、linked list系はiterative function (while loop)の方が標準と考えるのが自然。
  - 行き掛け、帰り掛けとも関連するtopic
  - Step 1で選択肢を複数出せたのは成長だったが、その中からの選択（のプロセス）は良くなかった気がする。
  - Step 3はwhileでやります。

## Step 3

while loop

- 時間計算量: O(n)
- 空間計算量: O(1)

```python3
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        head = tail = None
        carry = 0
        while l1 or l2 or carry:
            total = carry
            if l1:
                total += l1.val
                l1 = l1.next
            if l2:
                total += l2.val
                l2 = l2.next
            carry = total // 10
            new = ListNode(total % 10)
            if head is None:
                head = tail = new
            else:
                tail.next = new
                tail = tail.next
        return head
```

ListNodeインスタンスの作成個数が必要最小限で済むようにした。
