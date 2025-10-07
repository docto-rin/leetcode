## Step 1

整数の配列`nums`と整数`target`が与えられ、`nums`から、要素の値の和がtargetとなるようなインデックスペアを返す問題。
同じ要素は2回使うことはできない。

- アルゴリズムの選択
  - 単純には、numsを二重ループで全探索して和がtargetになればreturn、とすればいいが、2 <= nums.length <= 10^4で、O(n^2)はきつい。
  - なのであらかじめhashtable（今回はcollections.Counter）でnums内の要素を管理し、numsを一重ループで回してtarget-numがCounterで存在するか確認する方針。O(n)になる。
  - O(n)を超えるのは厳しそうだが、nums内に整数の重複がある場合は、1回だけチェックするようにしたら高速化が期待できる。
- 実装の方針
  - 基本的には上記の通り。
  - Only one valid answer exists. とはなっているが、そうでない入力にも耐える仕様にしたい。
    - 例1：正解が存在しない場合はNoneを返す。
    - 例2：正解が複数存在する場合は最初に見つかったものを返す。

### 実装1

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
from collections import Counter
from typing import List, Optional

class Solution:
    def twoSum(self, nums: List[int], target: int) -> Optional[List[int]]:
        counter = Counter(nums)
        for num in counter.keys():
            partner = target - num
            count_threshold = 1
            if partner == num:
                count_threshold += 1
            if counter[partner] >= count_threshold:
                num_idx = nums.index(num)
                # `partner` should be after `num`
                partner_idx = num_idx + 1 + nums[num_idx + 1:].index(partner)
                return [num_idx, partner_idx]
        return None
```

この方法において、Counter（dictのサブクラス）の挿入順序を記憶しておく機能（[python3.7より実装](https://docs.python.org/ja/3.13/library/collections.html#ordereddict-objects)）を活用し、常にnum_idx < partner_idxが成り立つ性質を活用した。

こうすることで、num_idx == partnerの場合もnum_idx != partnerの場合も同じロジックでインデックスを取得できる。

ここまで13分。

### 実装2

count_threshouldだが、下記のような書き方はpythonっぽくていいのかなと思った。

```python3
from collections import Counter
from typing import List, Optional

class Solution:
    def twoSum(self, nums: List[int], target: int) -> Optional[List[int]]:
        counter = Counter(nums)
        for num in counter.keys():
            partner = target - num
            if counter[partner] >= 1 + (partner == num):
                num_idx = nums.index(num)
                # `partner` should be after `num`
                partner_idx = num_idx + 1 + nums[num_idx + 1:].index(partner)
                return [num_idx, partner_idx]
        return None
```

> Follow-up: Can you come up with an algorithm that is less than O(n2) time complexity?

すでに満たしている。

更なる改善を求め...

- 現状、同じペアに対して走査が2回ずつ走るようになっているが、1回ずつになるようにできないだろうか。
  - ナイーブには、num > partnerのパターンを捨て、num <= partnerだけ扱うようにするなど。
  - しかし、counterをkeyでソートする必要が出てくる。O(nlogn)がかかってしまう。
  - その上これをやると `partner` should be after `num` が崩れてしまう。
    - 後者については、インデックス取得の方法をおとなしく、num == partnerの場合とnum != partnerの場合で条件分岐させたら対処可能そう。
    - ヘルパー関数で条件分岐のめんどくささを隔離しよう。

これについては、納得する改善案が思いつかなかったので断念。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.z4zz4wpn0zz0)
  - https://discord.com/channels/1084280443945353267/1183683738635346001/1187326805015810089
    > エクセルみたいな表を作ります。左にnums1、上にnums2を書きます。で、左上が、絶対に一番小さいじゃないですか。まさか、全部の表を手作業で埋めて、全部比較してから一番小さいやつは、これ、ってやらないと思うんですよ。
    - 二重ループでやる場合は探す順番が非常に重要。
    > 目標よりも小さかったら、前の方の着目しているのを一つ後ろにずらします。目標よりも大きかったら、後ろの方の着目しているのを一つ前にずらします。
    - 一種の再帰とみなせ、再帰関数内の再帰呼び出しが末尾のただ1つのみとして書けるので、末尾再帰最適化の考え方から一重ループとして書ける。
    - もし元の配列がソート済みであれば十分有力な方法に思えた。
  - https://discord.com/channels/1084280443945353267/1263078966491877377/1296874739377115243
    > Exception だとすると、ValueError とかもいいかもしれませんね。組み込み例外の一覧を一回見ておくといいでしょう。
    > https://docs.python.org/ja/3/library/exceptions.html#ValueError
    - 例えば以下のような感じか。
      ```python3
      raise ValueError("No pair found in nums whose sum is equal to target")
      ```
    - なかなか書く機会がないが、なんとなく慣れておきたい。

## Step 3

見つからなかったとき、エラーが出るようにした。

個人的にはエラーが出ずにNoneが返ればいいのでは？と思っている。

### 実装3

```python3
from collections import Counter

class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        """
        Find indices of two numbers in nums whose sum equals target.
        If no such pair exists, raise ValueError.
        """
        counter = Counter(nums)
        for num in counter.keys():
            partner = target - num
            if counter[partner] >= 1 + (num == partner):
                num_index = nums.index(num)
                partner_index = num_index + 1 + nums[num_index + 1:].index(partner)
                return [num_index, partner_index]
        raise ValueError("No pair found in nums whose sum is equal to target")
```
