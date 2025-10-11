## Step 1

- 問題文
  - 整数の配列`nums`と整数`k`を受け取り、和が`k`に等しくなるような*subarraysの個数*を返す問題。
  - 制約:
    - 1 <= nums.length <= 2 * 10^4
    - 1000 <= nums[i] <= 1000
    - 10^7 <= k <= 10^7
- アルゴリズムの選択
  - 全探索しか思いつかない。
    - nums[i]が正と負の値を両方とるため、二重ループになってしまう。
    - 事前に累積和をO(n)で計算しておくことで、任意の区間和をO(1)で求められるようになることは使えそう。
    - それでもO(n^2)。走査回数が10^8オーダーになるため多分違う。

- 時間計算量: O(n^2)
- 空間計算量: O(n)

```python3
# Time Limit Exceeded
from typing import List
from itertools import accumulate


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        accumulates = [0] + list(accumulate(nums))
        count = 0
        for right in range(1, len(accumulates):
            for left in range(right):
                if accumulates[right] - accumulates[left] == k:
                    count += 1
        return count
```

Time Limit Exceeded。

Hintを見てみると、

> What about storing sum frequencies in a hash table? Will it be useful?

と書いてあるが、意図を理解できなかった。のでLLMに聞いてみる。

> 1.  配列の先頭から順に要素を走査し、その時点までの累積和 `current_sum` を計算していきます。
> 2.  あるインデックス `i` までの累積和が `current_sum` だとします。もし、それより前のインデックス `j` (`j < i`) までの累積和 `prev_sum` が存在し、`current_sum - prev_sum = k` を満たすなら、インデックス `j+1` から `i` までの部分配列の和が `k` になることを意味します。
> 3.  この式を変形すると `prev_sum = current_sum - k` となります。
> 4.  つまり、各ステップで `current_sum` を計算したとき、「`current_sum - k`」という値の累積和が過去に何回現れたかを調べれば、その回数分だけ条件を満たす部分配列が存在することになります。
> 5.  この「過去の累積和が何回現れたか」を効率的に記録・参照するためにハッシュマップを使います。

3番目がトリッキー。累積に従った和の変化は単調でないが、とにかくこれまでの全配列をある位置より左側を和current_sum - kで切り出せば、右側の和がkになるように残る。このような切り出せる位置をループごとに数えるということ。

思いつけない。。。ひとまず理解したので実装する。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
from typing import List
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        sum_counter = defaultdict(int)
        sum_counter[0] += 1
        accumulated = 0
        num_subarrays = 0
        for number in nums:
            accumulated += number
            if (accumulated - k) in sum_counter:
                num_subarrays += sum_counter[accumulated - k]
            sum_counter[accumulated] += 1
        return num_subarrays
```

## Step 2

- https://github.com/goto-untrapped/Arai60/pull/28/
  - step1がO(n^2)ですが、TLEにならなかったのでしょうか。Java速いですね。
  - > endという変数名だとexclusiveな範囲を想像する人が多いかも？と思いました。
    - 確かに。
- https://discord.com/channels/1084280443945353267/1195700948786491403/1253847901969580082
  - > これは自明な書き換えに見えますか?
    - こういう現象は自分も遭遇しがち。
- https://discord.com/channels/1084280443945353267/1303257587742933024/1322011962598625280
  - sum_counter[0] += 1 をループの先頭へまとめることも可能。
    ```python3
        sum_counter = defaultdict(int)
        accumulated = 0
        num_subarrays = 0
        for number in nums:
            sum_counter[accumulated] += 1
            accumulated += number
            if (accumulated - k) in sum_counter:
                num_subarrays += sum_counter[accumulated - k]
    ```
    - うーん。sum_counter[accumulated] += 1がラグいので確かにわかりにくい。

## Step 3

```python3
from typing import List
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        sum_frequency = defaultdict(int)
        sum_frequency[0] = 1
        current_sum = 0
        count = 0
        for number in nums:
            current_sum += number
            if current_sum - k in sum_frequency:
                count += sum_frequency[current_sum - k]
            sum_frequency[current_sum] += 1
        return count
```

- if current_sum - k in sum_frequency:について、inよりも+, -が先に評価されるとのことで、冗長なカッコを外した。
  - https://docs.python.org/ja/3.13/reference/expressions.html#operator-precedence
- if current_sum - k in sum_frequency:の分岐を無くしても動くが、分岐がある方が空間的に小さい（自明）のと、時間も高速。
  - current_sum - kが未出現だったとしても都度hashtableへの挿入が明示的に生じるため、衝突の可能性やhashtableの動的拡張により、inチェックより遅くなる。
