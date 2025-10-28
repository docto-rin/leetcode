## Step 1

- 問題文
  - 前回の問題に、売買の回数上限がなくなった版。（ただし、常に最大1株しか持てない。）
  - 前回の問題文：
    - 非負整数の配列`prices`が与えられる。prices[i]はi日目の株価である。
    - どこかの日で株を買い、それより未来の日で株を売って得られる収益の最大値を知りたい。
    - 取引は1回までできる。（`prices`が単調増加する場合などで）取引しない場合は収益は0になる。
  - 制約：
    - 1 <= prices.length <= 3 * 10^4
    - 0 <= prices[i] <= 10^4

### 実装1

- アルゴリズムの選択
  - 一見複雑だが、これも一手先の未来をみるようにすれば貪欲的に解けそう。
  - 株の教科書には大体「株価の谷で買い、山で売るのが理想。だけど実際には株価が次上がるか下がるかわからないのでそれは無理で、...」みたいな書き出しがある。
  - しかし今回は変動が完全に見える。（プロットすれば山谷がわかる状態。）
  - 具体的には、prices[i + 1] - prices[i]の正負がこれまでと切り替わるタイミングで売買をすればいい。
- 実装
  - forループで1周すればいける。
  - 最後の山or谷の価格（極値）と増加中or減少中のフラグを管理すれば貪欲的に売買可能。
  - 価格差が0のときは、売買してもしなくても正しいが、わかりやすくしないことにする。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0

        total_profit = 0
        extremum = prices[0]
        increasing = True
        for i in range(len(prices) - 1):
            if increasing:
                if prices[i + 1] - prices[i] >= 0:
                    continue
                total_profit += prices[i] - extremum
                extremum = prices[i]
                increasing = False
                continue
            
            if prices[i + 1] - prices[i] <= 0:
                continue
            extremum = prices[i]
            increasing = True
        
        if increasing:
            total_profit += prices[-1] - extremum
        return total_profit
```

- ここまで25分ほど。（動くコードは20分くらいで書けたが、リファクタした）

## Step 2

- レビュー by GPT-5
  - > 「上がった日だけ都度差分を足す」貪欲解が最適。
  - 確かに。現実の株の売買回数を減らしたい気持ちが邪魔してこれは思いつかなかった。

## Step 3

```python3
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices:
            return 0

        total_profit = 0
        for i in range(1, len(prices)):
            total_profit += max(0, prices[i] - prices[i - 1])
        return total_profit
```
