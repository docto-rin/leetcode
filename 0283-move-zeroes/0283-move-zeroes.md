## Step 1

- 問題文
  - 整数の配列`nums`を与えられる。
  - in-placeですべての`0`を右端に追いやれ。
  - non-zeroの要素の相対順序は保持すること。
  - 制約
    - 1 <= nums.length <= 10^4
    - -2^31 <= nums[i] <= 2^31 - 1

### 実装1

- アルゴリズムの選択
  - 0を取り除く走査と、後ろに除いた分くっつける操作に分けることができる。
- 実装
  - 最初、numsをforで回して0の位置を特定し、2周目のforで除くことを考えたが、popするたびにインデックスがズレる。
  - 0をpopした回数で最初記憶したインデックスをずらせば正しく動くが、少し微妙かなと感じた。
  - while2重ループで、1-passで見つけ次第除去していく書き方にすることにした。
    - 補助空間がO(n) -> O(1)になる。
- 計算量
  - Time: O(n^2)
  - Extra Space: O(1)
  - Input Space: O(n)

```python3
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        """
        return nothing, modify nums in-place instead.
        """
        initial_length = length = len(nums)
        index = 0
        while index < length:
            while index < length and nums[index] == 0:
                nums.pop(index)
                length -= 1
            index += 1
        nums.extend([0] * (initial_length - length))
```

- ここまで6分。
- > Follow up: Could you minimize the total number of operations done?
  - 特に思いつかない...

### 実装2

- [実装1](#実装1)は配列を左から走査しているが、右から走査した方がpop()が軽くなると思った。
  - 特に、nums = [0] * 100などのとき、[実装1](#実装1)は常に0番目をpopすることになるが、右から走査すれば常にラストをpopする。
  - https://wiki.python.org/moin/TimeComplexity
    - > Popping the intermediate element at index k from a list of size n shifts all elements after k by one slot to the left using memmove. n - k elements have to be moved, so the operation is O(n - k). The best case is popping the second to last element, which necessitates one move, the worst case is popping the first element, which involves n - 1 moves. The average case for an average value of k is popping the element the middle of the list, which takes O(n/2) = O(n) operations. 
- operationsの総回数は変わっていない。

```python3
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        return nothing, modify nums in-place instead.
        """
        index = len(nums) - 1
        count_pop = 0
        while index >= 0:
            while index >= 0 and nums[index] == 0:
                nums.pop(index)
                index -= 1
                count_pop += 1
            index -= 1
        nums.extend([0] * count_pop)
```

## Step 2

- GPT-5
  - ダブルポインタが標準解らしい。
- https://github.com/olsen-blue/Arai60/pull/55/
  - 中間要素のpopを使わず、インデックスアクセスだけで完結する方法があった。
  - これならTime ComplexityがO(n)で済む。
  - -> [実装3](#実装3)、[実装4](#実装4)
- https://github.com/shintaro1993/arai60/pull/58/
  - if nums[i] == 0: continue もいいかもしれない。今回は後続の処理が短いので、どちらでもいいかも。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.v62rdhwkdymb)
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1276579345099526267
    - > まとめて 0 fill は、loop unrolling できたりするのでちょっと嬉しいこともあるでしょう。
    - https://ja.wikipedia.org/wiki/%E3%83%AB%E3%83%BC%E3%83%97%E5%B1%95%E9%96%8B
    - GPT-5にPythonにloop unrollingがあるか聞いてみたところ：
      > - PyPy など JIT 系ではホットなループのアンローリングが起きる可能性はありますが、CPython では期待できません。
      > - Python に「有効な loop unrolling テク」を期待するより、スライス代入やリスト乗算のような C 実装の一括処理を使うのが現実的です。
    - -> [実装3](#実装3)
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1276819880569602082
    - Generator
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1356874787800223807
    - > next の第2引数の default を使えば例外がなくせますね。
    - https://docs.python.org/ja/3/library/functions.html#next

### 実装3

- non-zeroを左から順に上書きしていき、その後残りを0で上書きしていく。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        """
        return nothing, modify nums in-place instead.
        """
        write_index = 0

        for value in nums:
            if value != 0:
                nums[write_index] = value
                write_index += 1

        nums[write_index:] = [0] * (len(nums) - write_index)
```

### 実装4

- non-zeroを見つけ次第、swapしていく。
  - swapは、基本的に左がzero、右がnon-zeroに対して入れ替えを行う。
    - 最初の0をスルーするまではnon-zero要素に対し、自分自身と入れ替えをする（つまり何もしない）。
  - non-zeroの相対順序を変えずに右側にzeroを押し込んでいくので、題意が満たされる。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        """
        return nothing, modify nums in-place instead.
        """
        def swap(index1, index2):
            nums[index1], nums[index2] = nums[index2], nums[index1]
        
        swap_target_index = 0
        for i in range(len(nums)):
            if nums[i] != 0:
                swap(i, swap_target_index)
                swap_target_index += 1
```

## Step 3

[実装3](#実装3)、[実装4](#実装4)のいずれか。
