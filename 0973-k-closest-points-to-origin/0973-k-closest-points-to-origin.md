# 973. K Closest Points to Origin

- URL: https://leetcode.com/problems/k-closest-points-to-origin/
- Difficulty: Medium
- Tags: Array, Math, Divide and Conquer, Geometry, Sorting, Heap (Priority Queue), Quickselect
- Notebook: https://share.solve.it.com/d/86115365713db4c20ea1d86f97c3fc23

## Step 1

### 実装1

- アルゴリズムの選択
  - 全ての点と原点との距離を計算し、小さい順にk個の点を特定して返す他ない。
- 実装
  - 小さい順にk個を求めるのはheapq.nsmallest()を使うと良いだろう。
    - k < len(points)のとき、計算量で有利
  - それでk個の点のpointsにおけるインデックスが求められるので、pointsに対しfilterする。
- 計算量
  - Time: ~~O(n)~~ -> O(n + nlogk)
  - Extra Space: O(n)

```python
import math
import heapq


class Solution:
    def kClosest(self, points: list[list[int]], k: int) -> list[list[int]]:
        def calc_distance(point1, point2=(0, 0)):
            x1, y1 = point1
            x2, y2 = point2
            return math.sqrt((x1 - x2) ** 2 + (y1 - y2) ** 2)
        
        distances = list(map(calc_distance, points))
        top_k_indices = heapq.nsmallest(
            k, range(len(points)), key=lambda index: distances[index]
            )

        return [points[i] for i in top_k_indices]

```

- 振り返り
  - フィルタリングにfilter()を使いたかったが、やり方を思いつかなかった。

## Step 2

🙋**Q:** ここまでのStep 1のレビューとそれに関するやり取りの内容をまとめてください。


🤖**A:**
**Step 1 コードレビューまとめ**

1. **`math.sqrt()` は不要**
   - 距離の大小比較だけなら、平方根を取らなくても結果は同じ
   - 計算コストを省ける

2. **`point2=(0, 0)` デフォルト引数**
   - 一般化の意図は理解できるが、この問題では使われていない
   - YAGNIの観点から削除も選択肢（ケースバイケース）

3. **計算量の訂正**
   - O(n) → O(n + n log k) または O(n log k)
   - `heapq.nsmallest` は O(n log k)

4. **`filter()` について**
   - 書けなくはない：
     ```python
     top_k_index_set = set(top_k_indices)
     top_k_index_and_points = filter(lambda x: x[0] in top_k_index_set, enumerate(points))
     return list(map(lambda x: x[1], top_k_index_and_points))
     ```
   - ただし、今回は「欲しいインデックスがすでにわかっている」ので、リスト内包表記 `[points[i] for i in top_k_indices]` のほうがシンプルで適切

---

Step 2 に進みますか？

- コメント集：空
- PR
  - https://github.com/huyfififi/coding-challenges/pull/28

🙋**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


🤖**A:**
PRの内容を確認しますね。

**PR #28 (huyfififi) のまとめ**

**アプローチ**
- 複数のソリューションを段階的に実装（step 1, 2, 3）
- Min heap, Max heap, Sort, Quickselect, `heapq.nsmallest()` などの手法を試している

**レビューコメント（nodchipさんより）**

1. **マイナスの距離について**
   - 距離にマイナスをかけている部分（max heapの実装のため）は自明ではないので、コメントで補足したほうが良い

2. **`pow(x, 2)` vs `x * x`**
   - `x * x` のほうが読みやすく、関数呼び出しのオーバーヘッドもない
   - ただしPythonインタープリター自体が重いので、あまり気にしなくても良いかもしれない

**その後の対応**
- huyfififiさんが `pow()` と `x * x` の `dis.dis()` 結果を確認するコミットを追加

---

他にも確認すべきPRはありますか？

🙋**Q:** Step 2で検討した内容をまとめてください。


🤖**A:**
**Step 2 まとめ**

**PR #28 レビューから学んだこと：**

1. **Max Heap のテクニック**
   - Python の `heapq` は min heap のみ
   - max heap が欲しいときは値にマイナスをかける
   - このテクニックは定番だが、コメントで意図を説明すると親切

2. **`pow(x, 2)` vs `x * x`**
   - `x * x` のほうが命令数が少ない（3命令 vs 5命令）
   - `x * x` は関数呼び出しのオーバーヘッドがない
   - 可読性も `x * x` のほうが良いという意見

3. **`dis.dis()` でバイトコードを確認する方法**
   - 命令の比較でパフォーマンスの違いを可視化できる

**実装の改善：**

4. **`itertools.starmap()` の活用**
   - `map()` は要素をそのまま渡す
   - `starmap()` は要素を unpack して渡す
   - `starmap(func, points)` で `[x, y]` を `func(x, y)` として呼べる

---

Step 3 に進みますか？

### 実装2

- リファクタ内容
  - 距離は2乗したままでも大小関係変わらないので、sqrtは省略する。
  - また、2点間の距離を求める関数として書くと過剰なので、YAGNIの観点から必要十分な機能にする。
  - itertools.starmap()の活用。

```python
import itertools
import heapq


class Solution:
    def kClosest(self, points: list[list[int]], k: int) -> list[list[int]]:
        def calc_square_distance(x, y):
            """calculate square distance from origin (0, 0)"""
            return x * x + y * y
        
        distances = list(itertools.starmap(calc_square_distance, points))
        top_k_indices = heapq.nsmallest(
            k, range(len(points)), key=lambda index: distances[index]
        )

        return [points[index] for index in top_k_indices]
```

### 実装3

- リファクタ内容
  - distancesリストとtop_k_indicesを経由する必要がなかった。
  - calc_square_distanceの引数のunpackだが、itertools.starmap()が使えないので、代わりに：
    - calc_square_distance(point)内でやる
    - key=lambda p: calc_square_distance(*p)でやる
  - の2通りある。シンプルな前者を選択。

```python
import heapq


class Solution:
    def kClosest(self, points: list[list[int]], k: int) -> list[list[int]]:
        def calc_square_distance(point):
            """calculate square distance from origin (0, 0)"""
            x, y = point
            return x * x + y * y
        
        return heapq.nsmallest(k, points, key=calc_square_distance)
```

## Step 3

[実装3](#実装3)

## Step 4

自分のPR：（自分のPRへのリンク）

🙋**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装4

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```