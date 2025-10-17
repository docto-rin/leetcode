## Step 1

- 問題文
  - 長さ`n`で昇順にソートされた配列を1~`n`のいずれかの回数だけ回転することを考える。
  - 例えば、`nums=[0,1,2,4,5,6,7]`を4回回転させたら、`[4,5,6,7,0,1,2]`となる。
  - ソートし、回転されたユニークな要素からなる配列`nums`に対し、最小な要素を返せ。
  - 時間計算量はO(log n)であること。
  - 制約：
    - n == nums.length
    - 1 <= n <= 5000
    - -5000 <= nums[i] <= 5000
    - All the integers of nums are unique.
    - nums is sorted and rotated between 1 and n times.

### 実装1

- アルゴリズムの選択
  - 二分探索が使えそう。
    - 単調述語、つまり何らかの条件をもとに最大値・最小値の間に境界線を引ければ良い。
      - 例えば、nums=[4,5,6,7,0,1,2] を考える。
      - nums[0]より値が大きい左側と、nums[-1]より値が小さい右側に分けられる。これを使う。
    - 不変条件：
      - 全てのi <= leftで、nums[i] >= nums[0]
      - 全てのi >= rightで、nums[i] <= nums[-1]
    - 配列の長さが1の場合や、回転数がnであり、nums=[0,1,2,4,5,6,7]のような時はうまくいかないので、先に別処理しておく。
- 実装
  - whileで書く。
- 時間計算量: O(logn)
- 空間計算量: O(1)

```python3
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        first = 0
        last = len(nums) - 1
        if nums[first] <= nums[last]:
            return nums[first]
        
        left = first
        right = last
        while right - left > 1:
            mid = (left + right) // 2
            if nums[mid] >= nums[first]:
                left = mid
                continue
            if nums[mid] <= nums[last]:
                right = mid
                continue
        return nums[right]
```

ここまで8分。

- first, lastを番兵にとった、未確定域が開区間型(left, last)といえる。
- 別処理が冗長な気がするので、不変条件の考え方が微妙な可能性がある。
  - nums[first], nums[last]のうち、注目するのは一方で良かった。
  - nums[first]を絡めると別処理が必要になる（[実装5](#実装5)）。nums[last]だけで考えると別処理は不要（[実装6](#実装6)）。

### 実装2

- bisect.bisect_leftを使えそう。
- [0, 0, ..., 0, 1, 1, ..., 1]という配列にして一番左の1を探せばいい。
- 時間計算量: O(n) <- 配列を全てバイナリに変えるため。
- 空間計算量: O(n)

```python3
from typing import List
import bisect


class Solution:
    def findMin(self, nums: List[int]) -> int:
        binary_array = [int(num <= nums[-1]) for num in nums]
        min_index = bisect.bisect_left(binary_array, 1)
        return nums[min_index]
```

最初、
```python3
        if nums[0] <= nums[-1]:
            return nums[0]
```
と書いていたが、これを書かなくても大丈夫なことに気づいた。
- len(nums) == 1は常にmin_index = 0
- 回転数nで最小値が一番左のときも、binary_arrayは1のみが並ぶので、min_index = 0

### 実装3

- Python 3.10よりbisect_leftにはkey引数があり、lambdaを渡せる。
  - https://docs.python.org/ja/3/library/bisect.html
- 時間計算量: O(logn)
- 空間計算量: O(1)

```python3
from typing import List
import bisect


class Solution:
    def findMin(self, nums: List[int]) -> int:
        if nums[0] <= nums[-1]:
            return nums[0]

        min_index = bisect.bisect_left(nums, 1, key=lambda x: x <= nums[-1])
        return nums[min_index]
```

- 第2引数はTrueでもいいらしい。
  - Pythonではboolはintのサブクラスなので、0 < 1より、False < True
    - https://docs.python.org/ja/3.13/library/stdtypes.html#boolean-type-bool
  - key=を渡すと、比較はkey()を施した"key空間"で行われるため。
    - https://docs.python.org/ja/3/library/bisect.html#bisect.bisect_left
    - > key specifies a key function of one argument that is used to extract a comparison key from each element in the array.
  - なお、key は配列要素にのみ適用され、検索値には適用しない仕様のため、検索値はTrueのが好ましそう。
    - > To support searching complex records, the key function is not applied to the x value.

### 実装4

- numsのビュワークラスを実装しても良い。
- 時間計算量: O(logn)
- 空間計算量: O(1)

```python3
from typing import List
import collections
import bisect


class BinaryView(collections.abc.Sequence):
    def __init__(self, base, pivot):
        self._base = base
        self._pivot = pivot

    def __len__(self):
        return len(self._base)

    def __getitem__(self, index):
        if self._base[index] <= self._pivot:
            return 1
        return 0


class Solution:
    def findMin(self, nums: List[int]) -> int:
        if nums[0] <= nums[-1]:
            return nums[0]

        view = BinaryView(nums, pivot=nums[-1])
        min_index = bisect.bisect_left(view, 1)
        return nums[min_index]
```

- https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.x8c3aen9xu80
- collections.abc.Sequence のabstractメソッドは__len__と__getitem__。（最低限これらを書けば良い）
  - https://docs.python.org/ja/3.13/library/collections.abc.html#collections.abc.Sequence
  - アンダースコア2個ずつで囲まれたメソッドの呼び方
    - special methods: https://docs.python.org/3/reference/datamodel.html
    - dunder (double underscore) methods: https://docs.python.org/3/glossary.html#term-dunder
- Sequence以外にも、MutableSequence、Mapping、MutableMapping、Set、MutableSetなどがある。
  - https://docs.python.org/ja/3.13/library/collections.abc.html

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.tzbo5j6mbqc2)
  - https://discord.com/channels/1084280443945353267/1230079550923341835/1233971372946882600
    - > nums[0] <= nums[i] な領域と nums[0] > nums[i] な領域の境界を探せ
      >
      > nums[-1] < nums[i]  な領域と nums[-1] >= nums[i] な領域の境界を探せ
    - なるほど、nums[0]とnums[-1]の片方だけ持ち出せばよかったのか。
  - > これ、組み合わせて16通りのソースコードが生成できますが、どれが動いてどれが動かないか答えられますか。
    - > Q1. 2で割る処理がありますが、これは切り捨てでも切り上げでも構わないのでしょうか。
      - A1. midが未確定域に入ればいい。（不変条件が崩れずに必ず縮まる設計であればなんでもいい。）
        - [left, right]、(left, right)はどっちでもいい。
        - [left, right)は切り捨て。
        - (left, right]は切り上げ。
    - > Q2. nums[mid] <= nums[right]とあるが、<でもいいですか？
      - A2. i == right（mid == right）が不変条件の条件文に入っていなければ良い。入っている場合でも命題が成り立つならいい。
    - > Q3. nums[right]は、nums[nums.length - 1]でもいいですか。
      - A3. どちらでもよい。
    - > Q4. rightの初期値はnums.lengthでもいいですか？
      - A4. 不変条件の始まりが「全てのright <= iで」ならいい。「全てのright < iで」だった場合はエッジケースでIndexErrorが起こる。
        - 未確定域の右側が閉区間になり、rightが更新されない限りあたかもはみ出たインデックスnums.lengthが未確定域に含まれているようにpivotが動きうる。
        - いわゆるtargetが右側にout of rangeな場合、最後にはmid = nums.lengthとなり、IndexErrorとなる。

## Step 3

### 実装5

nums[0]との大小比較

```python3
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        if nums[0] <= nums[-1]:
            return nums[0]
        
        left = 1  # i < left, nums[i] >= nums[0]
        right = len(nums) - 1  # i > right, nums[i] < nums[0]
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] >= nums[0]:
                left = mid + 1
            else:
                right = mid - 1
        return nums[left]
```

### 実装6

nums[-1]との大小比較

```python3
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        left = 0  # i < left, nums[i] > nums[-1]
        right = len(nums) - 1  # i >= right, nums[i] <= nums[-1]
        while left < right:
            mid = (left + right) // 2
            if nums[mid] > nums[-1]:
                left = mid + 1
            else:
                right = mid
        return nums[right]
```
