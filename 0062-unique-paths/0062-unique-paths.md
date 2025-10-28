## Step 1

- 問題文
  - m x nのグリッドを左上から右下へ行くユニークな経路の総数を返せ。
  - 制約：
    - 1 <= m, n <= 100
- アルゴリズムの選択
  - 案1：組み合わせの総数は階乗を用いて直接計算できる。
  - 案2：進む方向が右 or 下しかないなので、m, nの小さい版の答えを利用して解ける（最適部分構造がある）ため、動的計画法
  - 案3：BFS/DFSをearly returnなしで実施し、右下に到達するたびにカウントする。
  - この順に思いついた。実装としても、この順番で好ましいと思えた。
  - 案3 については、違う経路から同じ点に来た点を別々に処理するため不利に思えた。

### 実装1

- m - 1個の下矢印とn - 1個の右矢印を並べる方法の総数。
- m + n - 2のプレースホルダーのうち下矢印を置くm - 1箇所を決めれば方法が定まるので、

$${}_{m + n - 2} \mathrm{C}_{m - 1}$$

- 計算量
  - Time: O(m)
  - Space: O(1)

```python3
import math


class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        return math.comb(m + n - 2, m - 1)
```

- 4分かかった。（math.combを調べるのに時間がかかった）

### 実装2

- 動的計画法
- 計算量
  - Time: O(mn)
  - Space: O(n)

```python3
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        count = [1] * n  # by column
        for _ in range(1, m):
            for col in range(1, n):
                count[col] += count[col - 1]
        return count[n - 1]
```

- 0行目、0列目の境界条件に少し頭を使い、9分かかった。
- セルフfollow-up: m, nのどちらかが非常に大きい場合、そのままにしますか？
  - 空間計算量の節約のため、m > nになるようにswapできる。時間計算量はほぼ不変なのでお得。
  - n, m = sorted([m, n])

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.brtd7l7oqr0f)
  - https://discord.com/channels/1084280443945353267/1339428945845555252/1360645783300341760
    - > あー、このコード動くのかと思ったが、そうか、これ動くのか。1次元テーブルでの解法になっていますね。
    - oda-sanの反応に思わず笑ってしまった。
    - 実質的に[実装2](#実装2)が行われている。
    - list * n は常にlistの中身の参照を複製している（shallow copy）。
      - 要素が mutable（リストなど）のとき
        - 共有しているオブジェクトをある一箇所で変更すると、全ての位置から同じ変更が見える状態。
        - -> 全ての要素が変更される。
      - 要素が immutable（intなど）のとき
        - 共有しているオブジェクトをある一箇所で変更すると、要素がimmutableゆえオブジェクトが新規作成（参照が変更）され、他からは見えない。
        - -> 別々に変更ができる。
    - なので、2次元テーブルの準備には、ミュータブルの複製のためにforループを使ってオブジェクトを明示的に新規作成する。
      - table = [[0] * 10 for _ in range(10)]

## Step 3

[#実装1](#実装1)
