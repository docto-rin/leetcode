## Step 1

- 問題文
  - 整数の配列`nums`を与えられ、`nums`のnext permutaionに変換せよ。
  - 変換はin placeで行い、定数の追加メモリで済ませること。
  - 制約：
    - 1 <= nums.length <= 100
    - 0 <= nums[i] <= 100

### 実装1

- アルゴリズムの選択
  - 辞書順に樹形図を書くことを考える。
  - 末尾から根に向かって遡っていき、next permutationに分岐しているところまでいく方針。
  - 分岐する場所になるには、
    ```
    common part ---- current permutation part
                 └--    next permutation part
    ```
    のようになる必要がある。
  - 具体的には、[1, 3, 4, 3, 2] -> [1, 4, 2, 3, 3] においては、
    ```
    [1] ---- [3, 4, 3, 2]
         └-- [4, 2, 3, 3]
    ```
    のような感じ。
  - 分岐元[3, 4, 3, 2]の先頭3は、末尾から走査して降順が初めて崩れる地点とわかる。（降順である限り、下に分岐できないため。）
  - 分岐先[4, 2, 3, 3]の先頭4の決め方だが、[4, 3, 2]の中で、元の先頭3より大きい最小の数字を使えばいい。
  - 先頭以外の数字は単に昇順に並べれば良い。
- 実装
  - 空間の制限が強いので、swapベースで操作を組み立てていく。
  - 右から走査していくとき、あらかじめ逐次swapを使って昇順にしておくとスマート。
  - 元の先頭より大きい最小の数字とのswapは、bisect_rightを使えば良い。
  - あとは、境界条件などに気をつけて書く。
- 計算量
  - Time: O(n^2)
  - Extra Space: O(1)

```python3
import bisect


class Solution:
    def nextPermutation(self, nums: list[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        if len(nums) <= 1:
            return
        
        def swap(index1, index2):
            nums[index1], nums[index2] = nums[index2], nums[index1]
        
        def send_to_tail(from_index):
            for i in range(from_index, len(nums) - 1):
                swap(i, i + 1)

        index = len(nums) - 2
        previous_max = nums[index + 1]
        while index >= 0 and nums[index] >= previous_max:
            previous_max = nums[index]
            send_to_tail(index)
            index -= 1
        
        if index < 0:
            return
        
        swap_target = bisect.bisect_right(nums, nums[index], lo=index + 1)
        if swap_target < len(nums):
            swap(index, swap_target)
```

- ここまで18分。
- send_to_tailをswapを使って行う方法に悩んだ。
  - reversed()やlist.reverse()はコピーが生じざるをえないので使えない。
  - start, endなどの引数がないか調べたが、なさそう。
  - https://docs.python.org/3/tutorial/datastructures.html#more-on-lists
- whileの継続条件に悩んだ。
  - 数字はユニークとは限らない。その上で、継続条件に=は必要。（=のとき、まだ下に分岐できないので）

## Step 2

- GPT-5によるレビュー
  - > ただし `send_to_tail` による逐次スワップで末尾へ送る実装は、最悪で二重ループになりやすく、時間計算量が O(n^2) になります。`n ≤ 100` なら通りますが、標準解は O(n) で書けます。
    - 確かに。自分の癖で、1回の走査に2つのやりたいことを混ぜようとしがちだが、特に役に立っていない。可読性・保守性の観点でも好ましくない。
    - 改善した実装 -> [実装2](#実装2)
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.5y5ifenu4u32)
  - c++実装 -> https://en.cppreference.com/w/cpp/algorithm/next_permutation.html
    - [実装2](#実装2)をよりモジュール化した感じ。
  - https://discord.com/channels/1084280443945353267/1237649827240742942/1353878925117227113
    - > 初期化するために繰り返し変更すると意図が追いにくくなるでしょう。
    - 確かに。
- https://github.com/olsen-blue/Arai60/pull/59/
  - inner functionに分けているのが良いと思った。

### 実装2

- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
import bisect


class Solution:
    def nextPermutation(self, nums: list[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        if n < 2:
            return
        
        def swap(index1, index2):
            nums[index1], nums[index2] = nums[index2], nums[index1]

        pivot = -1
        for i in range(n - 2, -1, -1):
            if nums[i] < nums[i + 1]:
                pivot = i
                break
        
        if pivot == -1:
            nums.reverse()
            return
        
        left = pivot + 1
        right = n - 1
        while left < right:
            swap(left, right)
            left += 1
            right -= 1

        swap_target = bisect.bisect_right(nums, nums[pivot], lo=pivot + 1)
        assert swap_target < n
        swap(pivot, swap_target)
```

- pivotが見つかっている時点でswap_target < nは保証されていた...

### 実装3

- [実装2](#実装2)を関数化。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
import bisect


class Solution:
    def nextPermutation(self, nums: list[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        if n < 2:
            return
        
        def swap(index1, index2):
            nums[index1], nums[index2] = nums[index2], nums[index1]

        def is_sorted_until(first, last, step=1):
            for i in range(first, last, step):
                if nums[i] > nums[i + step]:
                    return i
            return last
        
        def reverse_in_section(left, right):
            while left < right:
                swap(left, right)
                left += 1
                right -= 1
        
        pivot = is_sorted_until(n - 1, 0, -1) - 1
        if pivot == -1:
            nums.reverse()
            return
        
        reverse_in_section(pivot + 1, n - 1)

        swap_target = bisect.bisect_right(nums, nums[pivot], lo=pivot + 1)
        assert swap_target < n
        swap(pivot, swap_target)        
   
```

- 参考
  - https://en.cppreference.com/w/cpp/algorithm/next_permutation.html
  - このc++の実装では、is_sorted_until(n - 1, 0, -1)の返り値を変数において、これを軸に書いている。
  - しかし、自分はpivotという概念を軸にまとめたいと思った。

## Step 3

[実装3](#実装3)
