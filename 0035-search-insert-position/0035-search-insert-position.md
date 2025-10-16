## Step 1

- 問題文
  - ユニークな整数が昇順にソートされた配列`nums`とターゲットの整数`target`を受け取り、`target`の挿入位置を示すインデックスを返す。
  - 時間計算量はO(logn)である必要がある。
  - 制約：
    - 1 <= nums.length <= 10^4
    - -10^4 <= nums[i] <= 10^4
    - nums contains distinct values sorted in ascending order.
    - -10^4 <= target <= 10^4
- アルゴリズムの選択
  - 線形で確かめていくとO(n)になるので、比較回数を最小限に済ます必要がある。二分探索でいいだろう。
  - recursiveとiterativeがあるが、iterativeの方が好ましいためiterativeで書く。
- 実装
  - 実用ではbisect.bisect_left(nums, target)で終わりだが、この中身を書くつもりでやる。
  - 探索域を示す閉区間の両端インデックスをleft, rightとし、この区間の中央値midとの比較をもとに狭めていく。
  - 特に、区間が狭まってきて区間の更新が止まるとき（再帰でいうbase case）に適切にループを脱出できるようにする。

### 実装1

まず一応bisectで書く。

```python3
from typing import List
import bisect


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        return bisect.bisect_left(nums, target)
```

- CPythonのPython実装
  - https://github.com/python/cpython/blob/e69fb2008e288f57d95fde6f327b4c9bbdb5d878/Lib/bisect.py#L74
- これは一旦見ずに中身を書くつもりでやる。

### 実装2

iterative。

- leftとrightの更新が止まる（＝ループを抜ける条件）はleft == midのとき。
- このとき閉区間[left, right]に含まれる整数は2個 or 1個になる。
- なのでループを抜けた後は、（ここ）<= nums[left] <（ここ）<= nums[right] <（ここ）の3箇所のどこが適切か判断すればいい。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1
        mid = (left + right) // 2
        
        while left < mid:            
            if target == nums[mid]:
                return mid
            
            if target > nums[mid]:
                left = mid
            else:
                right = mid
            
            mid = (left + right) // 2

        if target <= nums[left]:
            return left
        if target <= nums[right]:
            return right
        return right + 1
```

- 場合の整理に少し苦戦し、20分ほどかかってしまった。
- left = mid, right = midの更新について、midを区間に含める必要がないのは良くないかも。

## Step 2

[GPT-5によるStep 1のレビュー](https://chatgpt.com/share/68f0d80f-9b84-800c-bd95-f65dd8778362)
- > 場合分けが3つあるのが複雑。
  - left = mid + 1, right = mid - 1のようにタイトに更新すれば場合が減らせるとのこと。
  - [実装3](#実装3)
  - また、if target == nums[mid]: return midは配列内の整数がユニークであることを使っている。
    - bisect.bisect_leftは重複があっても一番左を探す必要があるのでこのreturnはそもそも書けない。
- > 半開区間を使った方がスマートに書ける。bisectの実装もそうなっている。
  - 発想になかった。
  - [実装4](#実装4)

### 実装3

閉区間、+1/-1を伴うタイトな更新。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1  # [start, end]
        answer = len(nums)
        while left <= right:
            mid = (left + right) // 2
            if target <= nums[mid]:
                answer = mid
                right = mid - 1
            else:
                left = mid + 1
        return answer
```

- +1/-1を伴うことで更新は「常に意味がある」ので、[left, right]に含まれる整数が空になる（left > right）まで更新を続けられる。
- [実装5](#実装5)にて後述する単調述語の観点で見ると、「target以上である」という単調述語の成立を右から見ていったときに最後に（一番左で）成立する位置を特定している。
  - なので、nums[mid] >= targetを確認するたびにanswerはmidの位置まで左に修正される。
  - 初期値 end = len(nums) はこの観点において、nums.append(float('inf'))したことに相当する。
- 空になる一個手前では、含まれる整数が1個のケース（left == mid == right）を必ず通過するが、このときにもanswerの最終調整が必要に応じてなされ、後処理は不要である。

### 実装4

半開区間。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        """Return the first index i with nums[i] >= target."""
        start, end = 0, len(nums)  # [start, end)
        while start < end:
            mid = (start + end) // 2
            if nums[mid] < target:
                start = mid + 1
            else:
                end = mid
        return start
```

- [実装3](#実装3)と同様、常に意味がある更新なので、[start, end)に含まれる整数が空になる（left == right）まで更新を続けられる。
- 更新則は、単調述語の観点だと「target以上である」の成立を左から見ていったときに最初に成立する点をstartでピン留めしている。
  - なので、nums[mid] < targetであるなら、そのような点は[mid + 1, end)にあるので、start = mid + 1の更新。
  - nums[mid] >= targetなら、単に幅を狭めるend = midの更新。

### 実装5

first_true（半開区間 [start, end)）。

* 目的

  単調述語 False → True の切り替わり点、すなわち「最初に True になる最小の i（first True）」を返す。

* 不変条件（lower_bound の例 `nums[i] >= target`）

  * すべての i < start で `nums[i] < target` が成り立つ
  * すべての i ≥ end で `nums[i] >= target` が成り立つ
    
  よって、確定域は `[0, start)` が False、`[end, n)` が True。未確定域は `[start, end)` で、ここを半減していく。

* 応用

  * `lower_bound` 相当: `pred(i) := nums[i] >= target`
  * `upper_bound` 相当: `pred(i) := nums[i] > target`

```python
from typing import List, Callable


def first_true(n: int, pred: Callable[[int], bool]) -> int:
    """Return the smallest index i in [0, n) such that pred(i) is True.

    If no such index exists, return n.

    Invariants for [start, end):
      - For all i < start: pred(i) is False
      - For all i >= end:  pred(i) is True
    """
    start, end = 0, n  # search range is [start, end)
    while start < end:
        mid = (start + end) // 2
        if pred(mid):
            end = mid
        else:
            start = mid + 1
    return start


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        """Lower bound: first i with nums[i] >= target."""
        return first_true(len(nums), lambda i: nums[i] >= target)


def bisect_right_like(nums: List[int], target: int) -> int:
    """Upper bound: first i with nums[i] > target."""
    return first_true(len(nums), lambda i: nums[i] > target)
```

### 実装6

last_true（閉区間 [left, right]）。

* 目的

  単調述語 True → False の切り替わり点に対して「True が成り立つ最後の i（last True）」を返す。

* 不変条件

  番兵を用いて `pred(left) == True`、`pred(right) == False` を保つ。
  具体的には初期値として `left = -1`、`right = n` を置くと常に、

  * `left` は True を満たす最大の位置
  * `right` は False を満たす最小の位置
    
  を指すように更新される。

* lower/upper への落とし込み

  * `lower_bound` 相当: last_true(len(nums), `nums[i] < target`) の結果に `+1`
  * `upper_bound` 相当: last_true(len(nums), `nums[i] <= target`) の結果に `+1`

```python
from typing import List, Callable


def last_true(n: int, pred: Callable[[int], bool]) -> int:
    """Return the largest index i in [-1, n-1] such that pred(i) is True.

    If no such index exists, return -1.

    Invariants for [left, right]:
      - pred(left)  is True   (left starts at -1, a virtual True)
      - pred(right) is False  (right starts at n,  a virtual False)
    """
    left, right = -1, n  # search range is (left, right) open inside; sentinels outside
    while right - left > 1:
        mid = (left + right) // 2
        if pred(mid):
            left = mid
        else:
            right = mid
    return left


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        """Lower bound via last_true: first i with nums[i] >= target."""
        index = last_true(len(nums), lambda i: nums[i] < target)
        return index + 1


def bisect_right_like(nums: List[int], target: int) -> int:
    """Upper bound via last_true: first i with nums[i] > target."""
    index = last_true(len(nums), lambda i: nums[i] <= target)
    return index + 1
```

[コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.e13uiztrq2u9)
- left, rightがどういう不変条件を満たしながら幅を狭めているかを理解するのが重要。
  - [実装2](#実装2)では、[left, right]に正解が含まれていそう、くらいでやっていた。
  - だがここまでで分かったこととして、この外側のいわば「確定域」に対して考えるのが良さそう。
  - 探索対象である不確定域は、確定域の否定としてでしか定義できないことに気づいた。
  - 単調述語による抽象化は理解の助けになった。
- [実装2](#実装2)でループ脱出後に後処理が必要だったのは、left, rightが不変条件を満たすように狭めていなかったから。
  - または、不変条件が明確でなかったから。
  - 不確定域の否定である確定域のことを考えると、自然と不確定域の定義に半開区間を使いたくなってくる。
- Step 3の選択肢としては、以下の3つになるだろう。
  - [実装4](#実装4)、[実装5](#実装5)のような半開区間型。first True。
  - [実装6](#実装6)のような番兵つき開区間型。
    - return leftならlast True
    - return rightならfirst True

## Step 3

### 実装7

半区間[start, end)。不変条件は、

- i < startのとき、nums[i] < target
- end <= iのとき、nums[i] >= target

start == endで停止し、そのときのstart（またはend）が単調述語「nums[i] >= target」のfirst Trueになるなぁ、と考えながら書いた。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        start, end = 0, len(nums)
        while start < end:
            mid = (start + end) // 2
            if nums[mid] < target:
                start = mid + 1
            else:
                end = mid
        return start
```

### 実装8

左の番兵leftと右の番兵right。不変条件は、

- i <= leftのとき、nums[i] < target
- right <= iのとき、nums[i] >= target

right - left == 1で停止し、その時のrightが単調述語「nums[i] >= target」のfirst True。

また、その時のleftが今回はlast False。return left + 1と書いても同じだが、書いてる人の頭の中は違う。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left = -1  # left sentinel: always nums[i] < target
        right = len(nums)  # right sentinel: always nums[i] >= target
        while right - left > 1:
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid
            else:
                right = mid
        return right
```

### 実装9

半開区間(last_true, first_false]を不確定域とした書き方。

脳トレとしてやってみた。

```python3
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        # predicate: nums[i] < target
        # invariant:
        # - for all i <= last_true, nums[i] < target
        # - for all first_false < i, nums[i] >= target
        last_true = -1
        first_false = len(nums) - 1
        while last_true < first_false:
            mid = (last_true + first_false + 1) // 2
            if nums[mid] < target:
                last_true = mid
            else:
                first_false = mid - 1
        return last_true + 1
```
