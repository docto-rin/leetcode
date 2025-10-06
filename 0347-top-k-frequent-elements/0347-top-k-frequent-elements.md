## Step 1

整数の入った配列`nums`と、配列長以下の自然数`k`が与えられ、配列内で出現頻度の高いtop-k個の要素を返す問題。

- アルゴリズムの選択
  - まずは`nums`から、要素: 出現回数 のような辞書を構成することを考える。
    - このメリットとしては、線形走査O(n)でこの辞書は作成できるし、作ってしまえば任意の要素に対し、出現回数をランダムアクセスO(1)で高速に知れることである。
      - fyi: [pythonの辞書はハッシュテーブルでの実装](https://docs.python.org/ja/3/c-api/dict.html)
    - この辞書さえ手に入れば、元の配列`nums`はお役御免になることはすぐにわかった。
  - 次に、辞書から頻度の高いtop-k個の要素を知る方法だが、頻度を優先度とした優先度付きキューが使えることを思いついた。
  - より効率の良い方法は思いつかないので、これでやってみる
- 実装方針
  - 辞書については、普通の辞書でもいいが、新しいキー（要素）がくるたびにvalueを初期化するのは手間と考えたので、[defaultdict](https://docs.python.org/ja/3.13/library/collections.html#collections.defaultdict)を使うことにした。
  - 優先度付きキューについては、heapqモジュールを使う。heapqはmin-heapなので、優先度 = -frequency とすることに注意する

ここまで2分。実装する。

配列長をn、ユニークな要素数をmとして、

- 時間計算量: O(n + mlogm)
- 空間計算量: O(m)

```python3
from collections import defaultdict
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        frequency_counter = defaultdict(int) # default: 0
        for num in nums:
            frequency_counter[num] += 1
        
        heap = [] # min-heap
        for num, frequency in frequency_counter.items():
            heapq.heappush(heap, (-frequency, num))
        
        k_most_frequent_nums = []
        for _ in range(k):
            _, num = heapq.heappop(heap)
            k_most_frequent_nums.append(num)
        
        return k_most_frequent_nums
```

ここまで8分。

> Follow up: Your algorithm's time complexity must be better than O(n log n), where n is the array's size.

上の実装だとm = nのとき、O(nlogn)なので、改善の必要がある。

あ、m回heappushしているのが無駄になってそう。ヒープの節点の個数は高々k個で良いはず。

また、[heapq.nlargest](https://docs.python.org/ja/3/library/heapq.html#heapq.nlargest)を使ったら良いのではないか？（内部実装を見ると、深さlogkでの処理がすでにされていた。）

時間計算量: O(n + mlogk) に減らせそう。

補足：二分木の深さlogm -> logk、走査の回数m -> m

問題文の制約：1 <= k <= m より、平均的に改善する。（これは[前回の問題](https://leetcode.com/problems/kth-largest-element-in-a-stream/)でやったことだった）

しかし最悪時間計算量は依然としてO(nlogn)のまま...

- 時間計算量: O(n + mlogk)
- 空間計算量: O(m)

```python3
from collections import defaultdict
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        frequency_counter = defaultdict(int) # default: 0
        for num in nums:
            frequency_counter[num] += 1
        top_k_frequent_items = heapq.nlargest(k, frequency_counter.items(), key=lambda kv: kv[1])
        return [num for num, _ in top_k_frequent_items]
```

defaultdictではなく[Counter](https://docs.python.org/ja/3.13/library/collections.html#collections.Counter)のが適していそう。

そう思う理由は今のところ、カウント用途に特化したクラスだから良さそう、というくらいしかない。

後半のtop_k_frequent_itemsを求めるところだが、Counterクラスのmost_common()メソッドなるものが使えそう。

しかしこのメソッドの内部実装を確認するとheapq.nlargestのwrapperだった。コードの見た目しか変わらない

```python3
from collections import Counter

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        counter = Counter(nums)
        top_k_frequent_items = counter.most_common(k)
        return [num for num, _ in top_k_frequent_items]
```

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.dkkbub5o1tvz)
  - top_k_frequent_items = heapq.nlargest(k, frequency_counter.items(), key=lambda kv: kv[1]) のkey=について、
    - > lambda x: x[1] は itemgetter(1) というのがあったのでメモしておきます。
      https://discord.com/channels/1084280443945353267/1337642831824814192/1371893051823358046
    - > sorted(num_count, key=num_count.get, reverse=True) で、key だけが並んでくれるでしょう。
      https://discord.com/channels/1084280443945353267/1339428945845555252/1343566807184838676
    - 今回の例で言えば、heapq.nlargest(k, frequency_counter, key=frequency_counter.get) と書けた。
https://docs.python.org/3/library/operator.html#operator.itemgetter
  - > この問題では解が一意に定まると制約にあるので大丈夫ですが、同率 k 位があるようなケースでは全件出力されてしまいませんか。
    - 確かに。自分の実装だといずれも同率k位は1個だけ入るようになっているが、この視点は意識できていなかった...
  - Quick Select
    - [クイックセレクト - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BB%E3%83%AC%E3%82%AF%E3%83%88)
      - クイックソートによく似た選択アルゴリズム。k番目に大きい要素と、それより大きい群、小さい群の3種類を取得できるので、本問題に使える。
      - 平均計算量はクイックソートがΘ(nlogn)なのに対してクイックセレクトはΘ(n)。
      - 最悪計算量はクイックソートと同じくΘ(n²)。
      - follow upに対する回答？
    - https://discord.com/channels/1084280443945353267/1183683738635346001/1185972070165782688
      > クイックソートについて知っていなくてはいけないことは
      > - 最悪・平均計算量
      > - 末尾再帰最適化
      > - ピボット選択
      > - マージソートとのプロコン
      > あたりです。
      > ピボット選択については、ソート済みで頭をピボットにするなどすると、最悪を引きます。それを避けるために、ランダム選択、3値のメジアンを取る、などが知られています。メジアン of メジアンズというアルゴリズムがあって、この再帰アルゴリズムを使うと、O(n)で下3割から7割のピボットが選べるんですが、これは常識範囲から外れる気がします。
      > https://en.wikipedia.org/wiki/Median_of_medians
      - 奥が深い。

<details>
  <summary>クイックソートに関する常識まとめ（by GPT-5）</summary>

  非常に整理されています。
挙げた4点が「クイックソートを本質的に理解するうえで外せない部分」です。
それぞれに加えて、通常はもう少しだけ踏み込むべき補足があります。以下で、
**(1)〜(4)はあなたの列挙した4項目を補強し、(5)以降は補足的だが実務で重要な観点**
として説明します。

### (1) 最悪・平均計算量

| 状況     | 計算量        | 発生条件                                              |
| ------ | ---------- | ------------------------------------------------- |
| **平均** | O(n log n) | pivot が毎回だいたい中央付近を分割する（データが偏っていない）                |
| **最悪** | O(n²)      | pivot が常に最小/最大を選び、分割が極端に偏る（例: 既に整列済みのデータ＋末尾pivot） |

対策として：

* pivot をランダムに選ぶ（**randomized quicksort**）
* 三点中央値（first/mid/last の中央値を pivot に）
* ある程度偏ったら挿入ソートに切り替える（現実の実装）

平均 O(n log n) でも、定数因子が小さいため実用上かなり速いです。

### (2) 末尾再帰最適化（tail recursion optimization）

Python は最適化を行いませんが、クイックソートは理論的に**片方の再帰が末尾再帰**になります。

```python
def quicksort(lo, hi):
    if lo >= hi: return
    p = partition(lo, hi)
    quicksort(lo, p - 1)      # ← こちらが先
    quicksort(p + 1, hi)      # ← 末尾再帰
```

右側の再帰は末尾呼び出しなので、ループに置き換えると**スタックを O(log n)** に抑えられます。
再帰が深くなって stack overflow するのを防ぐため、実装では普通に反復化します。
（Quickselect が while ループで書けたのはこの性質の応用です。）

### (3) ピボット選択

分割効率を決める最重要点。

* **固定位置 pivot**（例: 末尾要素）
  実装が簡単だが最悪ケースを誘発。
* **ランダム pivot**
  偏りを確率的に避ける。理論的に平均 O(n log n) を保証。
* **中央値近似 pivot**
  first/middle/last の中央値（median-of-three）で分割バランスを良くする。
* **真の中央値 pivot**
  理論上 O(n) で選べるが、オーバーヘッドが大きく実用的でない。

※クイックセレクトは「k 番目を求めるときに median-of-medians」で最悪 O(n) を保証するバリアントもあります。

### (4) マージソートとの比較（プロコン）

| 観点      | クイックソート                | マージソート             |
| ------- | ---------------------- | ------------------ |
| 平均計算量   | O(n log n)             | O(n log n)         |
| 最悪計算量   | O(n²)                  | O(n log n)         |
| 空間使用量   | **O(log n)**（再帰スタックのみ） | **O(n)**（マージ用バッファ） |
| 安定性     | 不安定                    | 安定                 |
| インプレース性 | ほぼインプレース               | バッファ必要             |
| 実効速度    | 速い（定数が小さい）             | 安定だがやや遅い           |
| 並列化     | 難しい（in-placeの副作用）      | 容易（マージ単位で分割できる）    |

したがって：

* **メモリ制約が厳しい場合** → QuickSort
* **安定性が必要な場合** → MergeSort
* **安定＋高速の折衷** → Timsort（Python の sort の実装）

### (5) 分割戦略と安定性（補足）

QuickSort は「スワップを行うため、同値要素の相対順序が壊れる」。
つまり**安定ソートではない**。
もし安定にしたい場合は、比較演算の際に「<」「==」「>」を別けて三分割パーティションにします（Dutch National Flag 法）。
これで平均的に安定しやすくなり、重複が多いとき高速化もします。

### (6) 実装上の定数因子

クイックソートが実際に速い理由は：

* 比較回数は同程度でも、メモリアクセスが局所的（キャッシュヒット率が高い）
* 分割のたびに大きな連続領域を順走査するだけ
  → CPU キャッシュと分岐予測に優しい。

### まとめ

クイックソートの「理解すべき核心」は次の6点に整理できます。

1. 平均 O(n log n)、最悪 O(n²)（分割のバランスが鍵）
2. 末尾再帰最適化（片側をループ化して O(log n) スタック）
3. ピボット選択戦略（ランダム化や中央値近似）
4. マージソートとの比較（安定性・空間・速度）
5. 安定性は無いが三分割で緩和可能
6. 実用ではキャッシュ効率が良く、最速クラス

この6点を押さえていれば、クイックソートおよび Quickselect の設計意図はすべて説明できます。

</details>

さて、クイックセレクトを実装する。

- 時間計算量: 平均O(n)
- 空間計算量: recursiveならO(m)、iterativeならO(1)

```python3
from collections import Counter
import random

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        frequency = Counter(nums)
        items = list(frequency.items())  # [(num, frequency), ...]

        def partition(left, right, pivot_index):
            pivot_frequency = items[pivot_index][1]
            items[pivot_index], items[right] = items[right], items[pivot_index]
            store_index = left
            for i in range(left, right):
                if items[i][1] > pivot_frequency:
                    items[i], items[store_index] = items[store_index], items[i]
                    store_index += 1
            items[right], items[store_index] = items[store_index], items[right]
            return store_index

        def select_recursive(left, right, target):
            if left == right:
                return
            pivot_index = random.randint(left, right)
            pivot_index = partition(left, right, pivot_index)
            if pivot_index == target:
                return
            elif pivot_index > target:
                select_recursive(left, pivot_index - 1, target)
            else:
                select_recursive(pivot_index + 1, right, target)

        def select_iterative(left, right, target):
            while left <= right:
                pivot_index = random.randint(left, right)
                pivot_index = partition(left, right, pivot_index)
                if pivot_index == target:
                    return
                elif pivot_index > target:
                    right = pivot_index - 1
                else:
                    left = pivot_index + 1

        select_iterative(0, len(items) - 1, k - 1)
        return [num for num, _ in items[:k]]
```

- GPT-5に時間計算量O(nlogn)より速い他の方法がないか聞くと、バケット法（バケツソート）があるとのことだった。
  - 各要素の頻度の値域は0~nなので、長さn+1の配列bucketsを用意し、各インデックス（バケツ）が頻度を表すようにする。
  - counterの情報から、各要素を適切なバケツに入れていく。線形走査なのでO(n)
  - その後、bucketsの後ろのインデックス（=頻度の高い方）から順に、要素を抜いてきて、合計k個抜いたらreturnする。
  - これもfollow upに対する適切な回答となりそう。頻度の離散性を用いた方法。

バケット法（バケツソート）の実装。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
from collections import Counter

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        counter = Counter(nums)
        buckets = [[] for _ in range(len(nums)+1)]
        for num, frequency in counter.items():
            buckets[frequency].append(num)
        k_most_frequent_nums = []
        for bucket in reversed(buckets):
            for num in bucket:
                k_most_frequent_nums.append(num)
                if len(k_most_frequent_nums) == k:
                    return k_most_frequent_nums
```

Submissionしたところ、Runtime, Memoryともに悪化した。しかし納得できる。

Test caseの分布的に、m, kが比較的小さかったのだろう。

使い分けは、

* Heap: k ≪ n、順序づけに使う値のパターンが多い or 連続値
* Quick Select: 平均計算量をO(n)に抑えたい、kが小さいとは限らない、順序づけに使う値のパターンが多い or 連続値
* Buckets: Quick Selectの要件に加え、最悪計算量をO(n)にしたい、順序づけに使う値のパターンが少ない

という感じだろうか。GPT-5にまとめさせたら以下のとおり：

| 手法           | 概要                      | 平均計算量      | 最悪計算量      | 特徴                   |
| ------------ | ----------------------- | ---------- | ---------- | -------------------- |
| ヒープ法         | 最小ヒープで上位 k 個を保持         | Θ(nlogk) | Θ(nlogk) | 安定して速く、実装が簡単         |
| Quickselect法 | クイックソートの分割を使って k 番目を探す  | Θ(n)       | Θ(n²)    | 比較ベース最速クラス、原理的にソート不要 |
| バケット法        | 頻度ごとに分類（Counting Sort系） | Θ(n)       | O(n)       | 出現回数が整数範囲に収まれば最適    |

## Step 3

Heap
```python3
from collections import defaultdict
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        counter = defaultdict(int) # default: 0
        for num in nums:
            counter[num] += 1
        return heapq.nlargest(k, counter, key=counter.get)
```
