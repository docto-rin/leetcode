## Step 1

- 問題文
  - ユニークな要素からなる整数配列`nums`が与えられる。
  - `nums`の可能なすべての部分集合（subsets）を返せ。
  - 制約：
    - 1 <= nums.length <= 10
    - -10 <= nums[i] <= 10
    - All the numbers of nums are unique.

### 実装1

- アルゴリズムの選択
  - 0 ~ 2^(nums.length) - 1の整数は、nums.length個それぞれ使うかどうかをビットパターンで表す。
    - すべてのビットパターンを生成し、それぞれに対応するsubsetを作っていく方針。
    - 素朴な発想だが、特に無駄は生じていないはず。
  - そのほかには、Backtrackingも使えそう。
- 実装
  - 二重ループ
  - ビット操作を正しく書く。
- 計算量
  - Time: O(n * 2^n)
  - Additional Space: O(n)
  - Output Space: O(n * 2^n)

```python3
class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = []
        
        for use_flag in range(1 << len(nums)):
            subset = []
            index = 0
            bit = 1
            while bit <= use_flag:
                if use_flag & bit:
                    subset.append(nums[index])
                index += 1
                bit <<= 1
            results.append(subset)
        
        return results
```

- ここまで6分。

### 実装2

- GPT-5に教えてもらったが、下記で十分だった。

```python3
class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = [[]]
        for value in nums:
            results += [subset + [value] for subset in results]
        return results
```

- 例えばlen(nums) == 3なら、len(results)はforのたび、1->2->4->8でreturn

### 実装3

- Backtrackingを再帰関数で実装
- 計算量
  - Time: O(n * 2^n)
  - Additional Space: O(n)
  - Output Space: O(n * 2^n)

```python3
class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = []
        path = []

        def traverse_paths(index):
            if index == len(nums):
                results.append(path.copy())
                return
            num = nums[index]
            traverse_paths(index + 1)
            path.append(num)
            traverse_paths(index + 1)
            path.pop()
        
        traverse_paths(0)
        return results
```

### 実装4

- Backtrackingをloop + stackで実装
- 計算量
  - Time: O(n * 2^n)
  - Additional Space: O(n)
  - Output Space: O(n * 2^n)

```python3
class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = []
        stack = [([], 0)]

        while stack:
            path, index = stack.pop()
            if index == len(nums):
                results.append(path)
                continue
            stack.append((path, index + 1))
            stack.append((path + [nums[index]], index + 1))

        return results
```

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.2x7tej35wxct)
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1265713750259011614
    - > 変な物を書いてみました。
    - -> [実装5](#実装5)
   
### 実装5

- [実装2](#実装2)の遅延評価バージョン
- https://docs.python.org/ja/3.14/library/itertools.html
- https://docs.python.org/ja/3.14/library/itertools.html#itertools.tee
- https://docs.python.org/ja/3.14/library/itertools.html#itertools.chain

#### 実装5-1

※誤った実装です。

```python3
# wrong code
import itertools


class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = [[]]

        for value in nums:
            a, b = itertools.tee(results)
            results = itertools.chain(a, (subset + [value] for subset in b))

        return list(results)

# Input
#   nums = [1,2,3]
# Output
#   [[],[3],[3],[3,3],[3],[3,3],[3,3],[3,3,3]]
# Expected
#   [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```
> 誤った出力をする原因は「遅延評価 × 変数の遅延束縛」です。
> 
> ジェネレータ式 `(subset + [value] for subset in b)` は評価時に `value` を参照します。
> ループが終わってから `list(results)` で初めて展開されるため、すべての箇所で最後の `value`（この例では 3）に置き換わってしまいます。

#### 実装5-2

- 修正案その1：ヘルパーで値を引数として束縛

```python3
import itertools
from typing import Iterable


def extend_with(v: int, iterable: Iterable[list[int]]) -> Iterable[list[int]]:
    for s in iterable:
        yield s + [v]


class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = [[]]
        for value in nums:
            a, b = itertools.tee(results)
            results = itertools.chain(a, extend_with(value, b))
        return list(results)
```

#### 実装5-3

- 修正案その2：map のデフォルト引数で束縛
- https://docs.python.org/ja/3/library/functions.html#map

```python3
import itertools


class Solution:
    def subsets(self, nums: list[int]) -> list[list[int]]:
        results = [[]]
        for value in nums:
            a, b = itertools.tee(results)
            results = itertools.chain(a, map(lambda s, v=value: s + [v], b))
        return list(results)
```

## Step 3

[実装2](#実装2)、[実装4](#実装4)あたりを確認した。
