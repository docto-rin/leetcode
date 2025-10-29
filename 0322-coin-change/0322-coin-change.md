## Step 1

- 問題文
  -  異なる額面のコインを表す整数配列`coins`と、合計額を表す整数`amount`が与えられる。
  -  合計額を作るためのコイン枚数の最小値を返せ。もし作れない場合は-1を返せ。
  -  各種類のコインは無限枚持っていると仮定できる。
  -  制約：
    -  1 <= coins.length <= 12
    -  1 <= coins[i] <= 2^31 - 1
    -  0 <= amount <= 10^4

### 実装1

- アルゴリズムの選択
  - コインの種類は多くても12種類らしい。現実的。
  - また、金額も10^4までと言うことで、優しい。
    - もし大きい場合は座標圧縮的なテクニックが必要になりそう。
    - coins + [amount]を、それらの最大公約数で割っておくなど。
  - amountの小さい問題から解いていき、bottom-upでいけそう。
- 実装
  - amountのforループと、coinsのforループ
- 計算量
  - m = coins.length, n = amountとして、
  - Time: O(mn)
  - Space: O(n)

```python3
class Solution:
    def coinChange(self, coins: list[int], amount: int) -> int:
        min_coins = [0] + [float("inf")] * amount
        for i in range(1, amount + 1):
            for coin in coins:
                if i - coin < 0 or min_coins[i - coin] == float("inf"):
                    continue
                min_coins[i] = min(min_coins[i], min_coins[i - coin] + 1)
        
        if min_coins[-1] == float("inf"):
            return -1
        return min_coins[-1]
```

- ここまで9分。

## Step 2

- レビュー by GPT-5
  - > * `float("inf")` は演算上問題ありませんが、配列型が `int` と混在します。整数のみで完結させたい場合は `INF = amount + 1` のような十分大きい整数を使うとよいです。
    - 確かに。inf系からさらにもう一歩工夫できないか考えるようにしてもいいかもしれない。
      - ダミーにNoneを使う代わりに配列の0番目や-1番目を使うのに似た発想だろうか。
    - INF役として、以下の4種類を試してGPT-5にベンチマークをさせたところ：
      > * `none_sentinel` 平均 **0.325 ms** 最速
      > * `int_inf` 平均 **0.565 ms**
      > * `math_inf` 平均 **0.740 ms**
      > * `float_inf` 平均 **0.976 ms** 最遅
  - > * 最内の条件 `i - coin < 0` は、外側の `i` に依存します。`coins` を昇順に並べ、内側で最初に `coin > i` になったら break するか、あるいはループ順を「硬貨外側・金額内側」に変えると分岐を減らせます。
    > * 最小枚数問題では、配列を上書きしても将来値への影響は正しく、`for coin in coins: for i in range(coin, amount + 1)` の順でも正解を保ちます。こちらの方が分岐が少なく高速になりやすいです。
    - 確かに、coin > amountなどは弾けたらかなりの定数倍高速化になる。
  - > * 0 以下の硬貨値が混入している場合は不正です。防御的に除外または例外を出すと堅牢になります。
    > * `amount == 0` は即時 0 を返せます。
    > * 同額の重複硬貨は一意化しても結果は変わりません。
    - 入力に対する例外、あまり意識が回せていなかった。

### 実装2

- [実装1](#実装1)を元に、GPT-5のアドバイスを反映。
  - INFをありえないintの値で表現。
  - coinは小さい順に試し、部分問題のamountより大きくなったらbreak。次の部分問題へ。
  - 異常入力に対する堅牢性、早期return 
- 計算量
  - m = coins.length, n = amountとして、
  - Time: O(mn)
  - Space: O(m + n)

```python3
class Solution:
    def coinChange(self, coins: list[int], amount: int) -> int:
        if amount < 0:
            return -1
        
        coins_sorted = sorted({c for c in coins if c > 0})
        INF = amount + 1
        min_coins = [0] + [INF] * amount

        for total in range(1, amount + 1):
            for coin in coins_sorted:
                if coin > total:
                    break
                min_coins[total] = min(
                    min_coins[total],
                    min_coins[total - coin] + 1
                )

        if min_coins[amount] == INF:
            return -1
        return min_coins[amount]
```

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ic8466had15a)
  - https://discord.com/channels/1084280443945353267/1226508154833993788/1307706069648146462
    - > 下の遅くなっている原因を >= に直したとすると、
    - 訪問済みを距離とboolの2つの配列で管理していることも混乱の一因かと思った。
    - seen[sum_val] and num_coins > amount_to_needed_coin[sum_val]で、andじゃなくてorなら助かってはいた。
  - https://discord.com/channels/1084280443945353267/1245404801177616394/1303611194795626557
    - > (コインの価格が十分に近いと Queue の中身は)、2^n で増えていきます。

### 実装3

- DFS。
  - 参考：https://github.com/nittoco/leetcode/pull/38/ の

```python3
class Solution:
    def coinChange(self, coins: list[int], amount: int) -> int:
        stack = [(0, 0)]
        INF = amount + 1
        fewests = [INF] * (amount + 1)
        # fewests[i] is the fewest number of coins to make up amount i
        sorted_coins = sorted(coins)   
        while stack:
            num_coins, sum_val = stack.pop()
            if sum_val > amount:
                continue
            if num_coins >= fewests[sum_val]:
                continue
            fewests[sum_val] = num_coins
            for coin_val in sorted_coins:
                stack.append((num_coins + 1, sum_val + coin_val))
        
        if fewests[amount] == INF:
            return -1
        return fewests[amount]
```

### 実装4

- 最短距離問題とみなしたBFS。
  - 参考：https://github.com/nanae772/leetcode-arai60/pull/39/
- 残額が0になったらreturnを、エンキューする前にやる分少し速くなる可能性がある。
  - 入力時点でamount==0を返すのを忘れないようにする。

```python3
class Solution:
    def coinChange(self, coins: list[int], amount: int) -> int:
        if amount < 0:
            return -1
        if amount == 0:
            return 0
        
        valid_coins = {coin for coin in coins if 0 < coin <= amount}
        sorted_coins = sorted(valid_coins, reverse=True)
        
        num_used = 1
        rests = [amount]
        visited = [False] * amount + [True]
        while rests:
            next_rests = []
            for rest in rests:
                
                for coin in sorted_coins:
                    next_rest = rest - coin 
                    if next_rest == 0:
                        return num_used
                    if next_rest < 0 or visited[next_rest]:
                        continue
                    next_rests.append(next_rest)
                    visited[next_rest] = True
            
            rests = next_rests
            num_used += 1
        
        return -1
```

## Step 3

[実装2](#実装2)
