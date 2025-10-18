## Step 1

- 問題文
  - 昇順にソートされた後ローテートされた配列`nums`を受け取る。要素は全てユニーク。
  - さらに整数`target`を受け取り、`nums`に`target`が存在すればそのインデックスを返し、存在しなければ-1を返す。
  - 時間計算量: O(log n)で書くこと。
  - 制約：
    - 1 <= nums.length <= 5000
    - -10^4 <= nums[i] <= 10^4
    - All values of nums are unique.
    - nums is an ascending array that is possibly rotated.
    - -10^4 <= target <= 10^4

### 実装1

- アルゴリズムの選択
  - 二分探索を使う。
  - 配列全体で見るとソートされていないのでそのままでは適用できない。
  - ..., max, min, ...を境に左右を分割してみると、それぞれはソート済みになる。
    - この特定は[前回の問題](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)で実装した。単純な二分探索でできる。
  - nums[-1]との比較により、左右どちらにtargetが存在しうるかを判断し、再度二分探索。
- 時間計算量: ~~O(log n)~~ <- スライスを使ってしまったので実際にはO(n)
- 空間計算量: ~~O(1)~~ <- スライスを使ってしまったので実際にはO(n)

```python3
from typing import List
import bisect


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        min_index = bisect.bisect_left(nums, True, key=lambda x: x <= nums[-1])
        if target <= nums[-1]:
            target_index = bisect.bisect_left(nums[min_index:], target) + min_index
        else:
            target_index = bisect.bisect_left(nums[:min_index], target)
        if nums[target_index] == target:
            return target_index
        return -1
```

## Step 2

- レビュー by GPT-5
  - 配列スライスによる計算量悪化
    - `nums[min_index:]` と `nums[:min_index]` は部分配列をコピーするのでO(n)。
    - `bisect_left` の `lo`, `hi` 引数で範囲指定すればコピーを避けられる。
      - またこのやり方なら、返り値のインデックスをoffsetで調整する手間も省けて嬉しい。
  - インデックス境界チェック不足
    - `bisect_left` の戻り値は上限（つまり引数 `hi`）に等しくなることがある。
    - そのまま `nums[target_index]` を参照すると `IndexError` の可能性がある。
    - `i < hi` を先にチェックすべき。

### 実装2

[実装1](#実装1)の改良。

- 時間計算量: O(logn)
- 空間計算量: O(1)

```python3
from typing import List
import bisect


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        min_index = bisect.bisect_left(nums, True, key=lambda x: x <= nums[-1])
        if target <= nums[-1]:
            lo, hi = min_index, len(nums)
        else:
            lo, hi = 0, min_index
        target_index = bisect.bisect_left(nums, target, lo=lo, hi=hi)
        if target_index < len(nums) and nums[target_index] == target:
            return target_index
        return -1
```

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.427rioitx1u6)
  - > key関数をいじる方法
    - numsの要素を元に、単調非減少に並ばせるkey空間を作る事ができれば、numsを引数に、1回のbisect_leftで答えが出せてしまう。
    - https://discord.com/channels/1084280443945353267/1262688866326941718/1344194431800184852
      - タプルで、順序づけのための値を優先順に書いていく。

### 実装3

- 時間計算量: O(logn)
- 空間計算量: O(1)

```python3
from typing import List
import bisect


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        key_function = lambda x: (x <= nums[-1], x)
        index = bisect.bisect_left(
            nums,
            key_function(target),
            key=key_function
        )
        if index < len(nums) and nums[index] == target:
            return index
        return -1
```

- key関数の計算は複雑になるが、bisect_leftは1回で済んでいる。
- x <= nums[-1]がFalse->Trueという単調述語をまず優先し、同じbool判定内ではxにより順序づけがされる、という構造。
  - 1個目の判定がrotationをほどいた順序づけを回復させることに相当する。
- key_function = lambda x: (x < nums[0], x)でもいい。

## Step 3

### 実装4

[実装3](#実装3)のkey_functionを少しだけ変えた。

```python3
from typing import List
import bisect


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        key_function = lambda x: (x < nums[0], x)
        index = bisect.bisect_left(
            nums,
            key_function(target),
            key=key_function
        )
        if index < len(nums) and nums[index] == target:
            return index
        return -1
```
