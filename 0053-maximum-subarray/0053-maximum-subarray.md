## Step 1

- 問題文
  - 整数の配列`nums`が与えられる。和が最大なsubarray（連続な部分配列）を見つけ、その和を返せ。
  - 制約：
    - 1 <= nums.length <= 10^5
    - -10^4 <= nums[i] <= 10^4

### 実装1

- アルゴリズムの選択
  - subarrayは連続しているので、貪欲的に解けそうだなと感じた。
  - 和が最大なsubarrayが、負の整数を跨いでいる場合が厄介。これを重点的に考察した。
  - そこまでの積み重ねのリターンが負の整数分差し引いても上回る場合、跨ぐ価値があると考察した。
  - 一般化して、nums[i]を第i世代の生涯収支とみなし、負になったら子供に相続させるのを取りやめるという例えがはまった。
  - この例えで、（相続分を含めた）財産が最大だった世代の財産を答えればよい。
- 実装
  - 1重なforループで左から貪欲的に調べられる。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        largest_sum = -float("inf")
        cumulated = 0
        for num in nums:
            cumulated += num
            largest_sum = max(largest_sum, cumulated)
            if cumulated < 0:
                cumulated = 0
        return largest_sum
```

- ここまで8分ほど。
- 後で知ったが、Kadaneのアルゴリズムという名前がついているらしく、この問題の最適解。
  - https://en.wikipedia.org/wiki/Maximum_subarray_problem

### 実装2

- > Follow up: If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.
  - 最適部分構造を見つけ、動的計画法（bottom-up）やメモ化再帰（top-down）などを適用せよということだろう。
  - 自力で書いてみたが、誤ってしまった。
    - 「区間内で一番良かった部分区間の開始・終了インデックス」を返すような再帰的なヘルパー関数を定義したが、不正だった。（引き継ぎに漏れがあった。）
    - 「中央をまたぐ最大部分和」になるケースで、「左半分の最大区間の開始」~「右半分の最大区間の終了」と接続していたが、そうとは限らない。
    - 例：[4,-1,2], [2,-5,4] に分けると、
      - それぞれ[4,-1,2], [4]が半分ずつの最大区間になる。
      - 自分の方法だと、最大をまたぐのが最良と判断するが、その結果できるマージ最大区間が[4,-1,2,2,-5,4]となってしまう。
      - 実際にはマージ後は[4,-1,2,2]が最大。
      - なので、[2,-5,4]から、マージ時は[2]まで使いなさい、というフィードバックがないといけない。
  - GPT-5に改善案を聞いた。
    - > 
      > 各区間 `[L, R]` について、次の4つを返すようにします：
      >
      > 1. `total`: 区間全体の和
      > 2. `best_prefix`: 区間の先頭から始まる最大部分和
      > 3. `best_suffix`: 区間の末尾で終わる最大部分和
      > 4. `best_subarray`: 区間内部の最大部分和
      >
      > これならマージができます。左右を `A`（左）と `B`（右）とすると、
      >
      > - `total = A.total + B.total`
      > - `best_prefix = max(A.best_prefix, A.total + B.best_prefix)`
      > - `best_suffix = max(B.best_suffix, B.total + A.best_suffix)`
      > - `best_subarray = max(A.best_subarray, B.best_subarray, A.best_suffix + B.best_prefix)` ←これが“中央をまたぐ”
      >
      > ベースケースは単一要素 `x` のとき
      > `total = best_prefix = best_suffix = best_subarray = x`。
    - 理解した。インデックスの情報は不要だが、和を様々なパターンに分類している。
    - 正しい最適部分構造を見つけられていないので、アルゴリズム力の問題。
- 実装
  - 簡単のため再帰関数を使う。
- 計算量
  - Time: O(n)
    - 初項n, 公比1/2の等比級数より。またはT(n)=2T(n/2)+O(1)より。
  - Space: O(logn)
    - コールスタック

```python3
from typing import List
from dataclasses import dataclass


@dataclass
class MaxSection:
    total: int
    prefix: int
    suffix: int
    inner: int


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        def helper(start, last):
            if start == last:
                return MaxSection(
                    total=nums[start],
                    prefix=nums[start],
                    suffix=nums[start],
                    inner=nums[start]
                )

            mid = (start + last) // 2
            left = helper(start, mid)
            right = helper(mid + 1, last)
            
            total = left.total + right.total
            prefix = max(left.prefix, left.total + right.prefix)
            suffix = max(right.suffix, left.suffix + right.total)
            inner = max(left.inner, right.inner, left.suffix + right.prefix)
            return MaxSection(
                total=total,
                prefix=prefix,
                suffix=suffix,
                inner=inner
            )
        
        return helper(0, len(nums) - 1).inner
```

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.qgjy53psjkn2)
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1207749510797992026
    - もう少し計算量が重いナイーブな方法から、ステップバイステップで考えていく方が自然ではないかという意見。
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1208414507735453747
    - > ある程度関数化するとかせずに、累積和の、i の位置は i 未満を足したものなのかそうでないのかとかを適当にやって、頭のメモリーを使っている状態で、上の二重ループを書こうとすると破綻するとかいうような話です。
    - 確かに、[実装1](#実装1)ではここに行き着いたあと、相続のアナロジーが偶然降って来て解決した感じがある。
    - 技術面接では、最適解の発想をいきなり狙って黙り込むより、確実に解ける方法をひとまず提案し、改善の方向性を話していく方がSWEとして自然に思える。

## Step 3

[実装1](#実装1)
