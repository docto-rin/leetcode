## Step 1

- 問題文
  - 非負整数の配列`prices`が与えられる。prices[i]はi日目の株価である。
  - どこかの日で株を買い、それより未来の日で株を売って得られる収益の最大値を知りたい。
  - 取引は1回までできる。（`prices`が単調増加する場合などで）取引しない場合は収益は0になる。
  - 制約：
    - 1 <= prices.length <= 10^5
    - 0 <= prices[i] <= 10^4

 ### 実装1

 - アルゴリズムの選択
   - brute forceする場合は計算回数が1 + 2 + ... + (n-1) = n(n-1)/2回になり、時間計算量はO(n^2)となる。
   - 今回も最適部分構造を見つけて動的計画法をやりたい。
     - 何の情報を返すかが大事で、部分問題の最大収益だけ分かっても意味がない。
     - 時系列に沿って考えるのが自然とみて、最終日を右に広げていくことを考える。
       - なお、空売りしてから買うことにすればおそらく右から考えることもできそうだが、特にやる意味はない。
     - 少なくとも、部分問題での価格の最小値は必要。逆にそれさえ分かればいけそう。
       - 価格の最小値より小さい価格が来たら、買いどきの更新。それ以外は売りどきかだけ判断すれば問題は順次解ける。
   - 動的計画法でO(n)。これ以上小さくするのは不可能だろう。
- 実装
  - loopで書けば良さそう。
  - 最初の日だけ例外的に処理（必ず暫定の買いどき）し、以降はループで処理するのがわかりやすい。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0
        
        max_profit = 0
        min_price = prices[0]
        for price in prices[1:]:
            if price < min_price:
                min_price = price
                continue
            if price - min_price > max_profit:
                max_profit = price - min_price
        return max_profit
```

- ここまで8分。
- レビュー by GPT-5
  - > `prices[1:]` はリストのスライスを新規作成するので、微小ながら無駄なメモリを使います。インデックスで 1 から回すか、`math.inf` 初期化で全要素を一度に処理するとより素直です。

## Step 2

- https://github.com/nanae772/leetcode-arai60/pull/36/
  - 初期化やスライスに関して吟味されている。
  - https://docs.python.org/ja/3/library/itertools.html#itertools.islice
    - itertools.islice(...) は「選択された要素を返すイテレータを作る」ので、部分列を事前に作らない＝コピーを作らない使い方になる。
    - prices[1:]prices[1:] のようなリストスライスは「新しいリストを返す」ため、その部分列分のメモリ割り当てが発生する。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.8qw2um7il4s5) 
  - https://discord.com/channels/1084280443945353267/1196472827457589338/1196473519689703444
    - > 本質的には、
      > - scanl min でその日までの最安の値。
      > - zipWith (-) で利益。
      > - max を取る。
    - pythonで実装してみる -> [実装2](#実装2)
    - Haskellでは遅延評価により各ステップで要素を一つずつしか保持しないので、入力pricesが実質一回の走査で処理され、追加メモリは定数オーダで済むとのこと。
    - 関連して、itertoolsの公式ドキュメントに、Haskellに着想を得て...と書いてあり、なるほどと思いました。
    - https://docs.python.org/3/library/itertools.html
    - > This module implements a number of iterator building blocks inspired by constructs from APL, Haskell, and SML. Each has been recast in a form suitable for Python.

### 実装2

- Haskell風（遅延評価）
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
from typing import List
from itertools import accumulate


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        """scanl min -> zipWith (-) -> maximum"""
        if not prices:
            return 0
        prefix_mins = accumulate(prices, func=min)  # scanl1 min
        profits = (p - m for p, m in zip(prices, prefix_mins))  # zipWith (-)
        return max(profits)  # maximum
```

- 補足
  - prefix_minsはiterator
    - func引数はHaskellのscanlの第一引数と同じ。なお第二引数はHaskellと異なり明示せず、暗黙でリストの1番目を使う。
    - https://docs.python.org/ja/3/library/itertools.html?ref=trap.jp#itertools.accumulate
  - profitsはgenerator expressionで定義されたgenerator
    - https://docs.python.org/ja/3.11/reference/expressions.html#generator-expressions
  - profits[0]が必ず0になるので、デフォルティング（取引しない）はそこが担っている。（ややパズル？でもシンプル）

## Step 3

### 実装3

```python3
from typing import List
from itertools import islice


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        min_price = prices[0]
        for price in islice(prices, 1, None):
            if price < min_price:
                min_price = price
                continue
            potential = price - min_price
            if potential > max_profit:
                max_profit = potential
        return max_profit
```
