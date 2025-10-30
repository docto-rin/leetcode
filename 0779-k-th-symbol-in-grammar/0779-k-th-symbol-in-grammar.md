## Step 1

- 問題文
  - `n`行（1-indexed）の表を作る。
  - 0行目に`0`を書き、以降の行は、前の行の`0`を`01`、`1`を`10`に置き換えたものを書く。
  - 整数`n`、`k`を与えられたとき、`n`行目の`k`番目（1-indexed）の記号（数字）を返せ。
  - 制約：
    - 1 <= n <= 30
    - 1 <= k <= 2^(n - 1)

### 実装1

- アルゴリズムの選択
  - 2重ループを回して表を完成させればいいと思ったが、n行目は2^(n - 1)より、合計2^n - 1回計算する必要があり、重い。
  - n行目のk番目を知るためには、k番目以外は知る必要がない。ceil(k / 2)が前の行での位置を表し、その状態が分かれば良い。
  - top-downで考えるとスムーズそう。
- 実装
  - loopでも書けそうだが、top-downなので書きやすい再帰関数で始める。
- 計算量
  - Time: O(n)
  - Space: O(n)

```python3
import math


class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if n == 1:
            return 0
        
        source = self.kthGrammar(n - 1, math.ceil(k / 2))
        mod = k % 2  # 1: first, 0: second
        if source == 1:
            return mod
        return 1 - mod
```

- ここまで10分。

### 実装2

- bottom-upをloopで実装。
- 途中でこの表の法則性にも気づいた。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
import math

# 0
# 01
# 0110
# 01101001

class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        # is odd num of 1 when expressing integer (k - 1) in (n - 1) bit?
        is_odd = False
        for shift in range(n - 1):
            if k - 1 >> shift & 1:
                is_odd = not is_odd
        
        return int(is_odd)
```

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.adit16u7jkla)
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1216054396161622078
    - [実装2](#実装2)だが、1を数える組み込みメソッドint.bit_count()があるらしい。
- 備忘録的に整数型のビット演算のdocsを貼っておきます。
  - https://docs.python.org/3/library/stdtypes.html#bitwise-operations-on-integer-types

### 実装3

- [実装2](#実装2)をbit操作だけで書く。
  - XORが、0と行うとそのまま、1と行うと反転する性質を使った。
  - 可読性は悪化していると考える。（練習目的）
- k.bit_length()回だけの走査に変更。
  - ゼロ埋め部分は一括でbit変更なし、と処理できるため。
  - これは明確な改善点。

```python3
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        bit = 0
        for shift in range(k.bit_length()):
            bit ^= k - 1 >> shift & 1
        return bit
```

### 実装4

- [実装2](#実装2)、[実装3](#実装3)を組み込みメソッドで実行

```python3
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        return (k - 1).bit_count() % 2
```

## Step 3

[実装4](#実装4)
