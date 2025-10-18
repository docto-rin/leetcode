## Step 1

- 問題文
  - `weights`は積荷の重さが整数で並んでおり、この順で船に積み込み、`days`日かけて全てを運ぶ。
  - このとき、船のキャパシティの最小値を返す問題。
  - キャパシティとは1日で運べる最大の積荷の量である。
  - 制約：
    - 1 <= days <= weights.length <= 5 * 10^4
    - 1 <= weights[i] <= 500
- アルゴリズムの選択
  - イメージできないので、この問題を手作業で取り組むことを考える。
    - まず、weightsの全体を見ないとcapacityの見積もりができない。
      - 最低でもmax(weights)以上でないと、運べない荷物が発生してしまう。
      - どんなに効率的に運んでも match.ceil(cumulated[-1] / days)以上は必要。
    - このあたりを初期値として、capacityを適当に固定して脳内シミュレーションする。
      - capacityを固定すると、各日に載せる積荷の個数は決定論的であり、最終的に`days`日で全て運べるかも決定論的なことに気づいた。
      - これは、capacityを引数とし、boolを返す関数として実装できる。
  - capacity_lo <= capacity <= capacity_hiの整数列に対し、上記の関数をkeyとして二分探索したときのfirst Trueを求める問題とみなせる。
- 実装
  - 累積和を前もって計算しておくと、key_function（capacityを引数に、boolを返す関数）の実装がしやすいと感じた。
- 計算量
  - n = weights.length
  - C = capacityの取りうる値の数 とし、
  - 時間計算量: O(n + logC * days * log n)
  - 空間計算量: O(n)
    - ※range関数は空間的に軽い。
      - https://docs.python.org/ja/3/library/stdtypes.html#typesseq-range

```python3
from typing import List
from itertools import accumulate
from math import ceil
import bisect


class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        cumulated = list(accumulate(weights))
        max_weight = max(weights)
        capacity_lo = max(max_weight, ceil(cumulated[-1] / days))
        capacity_hi = max_weight * ceil(len(weights) / days)

        def key_function(capacity):
            loaded_total = 0
            lo = 0
            for _ in range(days):
                lo = bisect.bisect_left(cumulated, True, lo=lo, key=lambda x: x - loaded_total > capacity)
                if lo == len(cumulated):
                    return True
                # 0 < lo <= len(cumulated)
                loaded_total = cumulated[lo - 1]
            return False
        
        return capacity_lo + bisect.bisect_left(range(capacity_lo, capacity_hi + 1), True, key=key_function)
```

ここまで27分。

- Runtime: 491 ms (Beats 5.04%)
- memory: 22.76 MB (Beats 16.85%)
- Runtimeの中央値は200msぐらいだったので、2倍遅い。

## Step 2

- レビュー by GPT-5
  - 関数名key_functionは中身に比して素朴すぎするので、可読性のために改善すべき。is_feasibleとか。
  - key_functionの計算量が悪い。weightで一重ループを回して貪欲的に区切れる。
    - 手作業でやってた時は明らかにこう考えていたのに、二分探索に囚われすぎてなぜか無理やりdayごとに二分探索する羽目になった。
    - 時間計算量: O(logC * days * log n) -> O(logC * n)に改善できる。
  - capacity_hiはsum(weights)でいい。
    - 上限・下限値は2倍冗長でも操作は1回しか増えない。

### 実装2

- [実装1](#実装1)の改良。
  - key関数の計算量を改善。
  - key関数の命名を改善。
  - capacityの上限、下限をわかりやすくした。
- 時間計算量: O(logC * n)
- 空間計算量: O(1) <- cumulatedが不要になった

```python3
from typing import List
import bisect


class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        def is_feasible(capacity):
            passed_days = 0
            loaded_today = 0
            for weight in weights:
                if loaded_today + weight <= capacity:
                    loaded_today += weight
                    continue
                passed_days += 1
                loaded_today = weight
                if passed_days == days:
                    return False
            return True

        capacity_lo = max(weights)
        capacity_hi = sum(weights) + 1
        return bisect.bisect_left(range(capacity_hi), True, lo=capacity_lo, key=is_feasible)
```

Runtime: 175 ms (Beats 77.15%)で、計算量オーダーの改善に従い実際に時短した。

- https://github.com/nanae772/leetcode-arai60/pull/43/
  - 異常入力の検討をされている。
  - Step 2で色々と参照されていて、追随側としてはありがたい。
- [実装2](#実装2)の異常入力に対する耐性
  - weightが0や負の値であっても、例外は送出しない。
    - 負の値とかは例えば船から荷物を取り出す操作とかに対応させられそう。（ノーショー）
    - 0や負の値を弾くかどうかはユーザーに任せたいと思います。
  - daysが0や負の値のとき、is_feasibleが常にTrueを返してしまう。
    - となると、このメソッドはエラーを吐かずに適当な値を出すことになる。
    - これは、最初にチェックしても良さそう。

## Step 3

### 実装3

```python3
from typing import List
import bisect


class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        if days <= 0:
            raise ValueError(f"days '{days}' must be positive integer")
        
        def is_feasible(capacity):
            passed_days = 0
            loaded = 0
            for weight in weights:
                if loaded + weight <= capacity:
                    loaded += weight
                    continue
                passed_days += 1
                if passed_days == days:
                    return False
                loaded = weight
            return True
        
        capacity_low = max(weights)
        capacity_high = sum(weights) + 1
        return bisect.bisect_left(
            range(capacity_high),
            True,
            lo=capacity_low,
            key=is_feasible
            )
```
