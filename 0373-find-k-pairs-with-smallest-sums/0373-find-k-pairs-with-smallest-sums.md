## Step 1

昇順（広義単調増加）で整数が格納された2つの配列`nums1`, `nums2`から、要素の値の和が最小な`k`個のペアを返す問題。

- アルゴリズムの思考過程
  - 制約：1 <= nums1.length, nums2.length <= 10^5 より、前もって全ての和を計算してソートしておくのは非現実的
  - nums1, nums2はソート済みなので、明らかに後ろの方のペアは考えなくて良さそう or 優先順位が低い
  - その逆で、明らかに[nums1[0], nums2[0]]はk個のペア（返り値）に含むべき。
  - すると同様に、（k>=2のとき、）次は[nums1[1], nums2[0]]と[nums1[0], nums2[1]]のうち少なくとも一方は明らかにk個のペアに含むべき。
  - この考え方を一般化するには、返り値のペアの決定をインデックスペア(i=0, j=0)から始め、その度に次の候補を(i+1, j), (i, j+1)の2つ追加する。
  - ペアがk個決定したら終了。
  - nums1[i]+nums2[j] を優先度とした優先度付きキュー（heap）を使って実装できる
  - ここまで20分
- アルゴリズムの選択
  - 上のものしか思いつかない...
- 実装の方針
  - 上のアイデアだけで実装したが、ジャッジシステムを使って以下のデバッグしてしまった。
    1. heappushしていいのは、i < len(nums1) - 1, j < len(nums2) - 1 の場合に限る
    2. 上記の方針だと、インデックスペア(i, j)が加えられるタイミングは、
       - (nums1[i-1], nums2[j]) が返り値に含まれると決定したとき
       - (nums1[i], nums2[j-1]) が返り値に含まれると決定したとき
       の2パターンある。なので、加えられたかどうかのフラグをset()などで管理しないと同じペアが複数回queueに入ってしまう
  - やっと思いついたアルゴリズムを実装することに精一杯で、脳内デバッガが十分に働いていなかった（言い訳）

### 実装1

- 時間計算量: O(klogk)
  - while loopはk回まわる
  - heapはループ1回につき1回popされ、最大2回pushされるので、heapの大きさは高々O(k)、1回の操作はO(logk)
- 空間計算量: O(k)
  - heapとsetからO(k) + O(k) = O(k)

```python3
import heapq

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        priority_queue = []
        return_pairs = []
        already_added = set()
        heapq.heappush(priority_queue, (nums1[0] + nums2[0], [0, 0]))
        while True:
            _, (i, j) = heapq.heappop(priority_queue)
            return_pairs.append([nums1[i], nums2[j]])
            if len(return_pairs) == k:
                return return_pairs
            if i + 1 < len(nums1) and not (i+1, j) in already_added:
                heapq.heappush(priority_queue, (nums1[i+1] + nums2[j], [i+1, j]))
                already_added.add((i+1, j))
            if j + 1 < len(nums2) and not (i, j+1) in already_added:
                heapq.heappush(priority_queue, (nums1[i] + nums2[j+1], [i, j+1]))
                already_added.add((i, j+1))
```

ここまで35分。

```python3
if len(return_pairs) == k:
    return return_pairs
```
の2行を削除し、`while len(return_pairs) < k`にした方が可読性が高い気がした。

ごく僅かに遅くなるが、それを上回る可読性・メンテナビリティが得られる。

## Step 2

GPT-5にアルゴリズムを改善できないか聞いたところ、

> 最初に`priority_queue` に `(nums1[i] + nums2[0], [i, 0])` を `i=0..min(k, len(nums1))-1` まで入れておき、pop した `(i, j)` に対しては **同じ行の次** `(i, j+1)` だけを push します。
こうすると重複が発生せず、`already_added` も `while True` も不要で、計算量は **O(k log min(k, len(nums1)))**、メモリは **O(min(k, len(nums1)))** まで下がります。

とのことだった。

- queueの初期状態で、nums1の要素を網羅しておくというのは思いつかなかった。
  - ハッシュテーブルalready_addedが不要で、ループごとのnot inのチェックも不要なので、定数倍の時間計算量、空間計算量が改善する。
- あと、queueの初期状態を作る際、i=0..min(k, len(nums1))-1 となっているのは地味に賢い。
  - k < len(nums1) のとき、nums1[k]は絶対に使われない、という事実から考えるのだろう。

実装する。

### 実装2

m = min(k, nums1.length)とすると、
- 時間計算量: O(klogm)
- 空間計算量: O(m)

```python3
import heapq

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        priority_queue = []
        return_pairs = []

        for i in range(min(len(nums1), k)):
            heapq.heappush(priority_queue, (nums1[i] + nums2[0], [i, 0]))

        while len(return_pairs) < k:
            _, (i, j) = heapq.heappop(priority_queue)
            return_pairs.append([nums1[i], nums2[j]])
            if j + 1 < len(nums2):
                heapq.heappush(priority_queue, (nums1[i] + nums2[j + 1], [i, j + 1]))

        return return_pairs
```

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.527w0lse8gbd)
  - [実装1](#実装1)について、
    ```python3
                if i + 1 < len(nums1) and not (i+1, j) in already_added:
                heapq.heappush(priority_queue, (nums1[i+1] + nums2[j], [i+1, j]))
                already_added.add((i+1, j))
    ```
    のところを関数化してadd_to_heap_if_necessary(i + 1, j)などとするなど。
    - 2回登場するので、繰り返しを避ける意味でも関数化は好ましい
  - [yield generatorによる実装](https://discord.com/channels/1084280443945353267/1235829049511903273/1246118347863621652)
    - > 実装の候補を十分にといったばかりに、変なのを出さざるを得なくなったので。
    ```python3
    # pseudo code
    gen_smallest_after(0)
    = merge(S0, merge(S1, merge(S2, merge(...))))
    ```
    - [K-way Lazy Merge Sort](https://leewc.com/articles/k-way-lazy-merge-sort/)
    - 再帰構造（木構造）が左偏重になっており、一番深いところ（一番右）の木の深さがlen(nums2)になっている。
      - k回の間に深い再帰が何度も呼ばれると計算量が悪くなる。
      ```python3
                  for latter in gen_smallest_after(j + 1):
                      while i < len(nums1) and sum(p := pair(i, j)) <= sum(latter):
                          yield p
                          i += 1
                      yield latter
      ```
        - while i < len(nums1) and sum(pair(i, j)) <= sum(latter): が回らずに、forループ（再帰）が捗る状況が苦手。
    - 使われているpython文法について補足：
      - https://docs.python.org/ja/3.13/reference/expressions.html#yield-expressions
        - yieldは、generator functionを定義するときのみに使う
        - generator function内のreturnは、そこに達するとStopIteration例外になる。 
      - https://docs.python.org/ja/3.13/glossary.html#term-generator
        - 定義内にyieldを含む関数はgenerator functionとして扱われ、呼ぶとgenerator iteratorが返る
      - https://docs.python.org/ja/3.13/library/itertools.html#itertools.islice
        - islice は、「iterable(つまりはiteratorも含む)の途中をスライスしたように扱う」ための明示的な関数
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1206515949579145216
    > 問題設定では、必ず k あるはずなんでしたっけ、でも、なかったときに
    >
    > IndexError is raised.
    > https://docs.python.org/3/library/heapq.html#heapq.heappop
    >
    > するわけです。
    - 例えば[実装2](#実装2)では、while len(return_pairs) < k:のところを、
      ```python3
      while len(return_pairs) < k and priority_queue:
      ```
      とすることで、k個集まる前にqueueが尽きたとしても集まった分返せる。
  - 

## Step 3

```python3
import heapq
from itertools import islice

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        def pair(i, j):
            return [nums1[i], nums2[j]]
        def gen_smallest_pairs():
            priority_queue = []
            for j in range(min(len(nums2), k)):
                heapq.heappush(priority_queue, [sum(pair(0, j)), (0, j)])
            while priority_queue:
                _, (i, j) = heapq.heappop(priority_queue)
                yield pair(i, j)
                if i + 1 < len(nums1):
                    heapq.heappush(priority_queue, [sum(pair(i + 1, j)), (i + 1, j)])
        return list(islice(gen_smallest_pairs(), k))
```
