## Step 1

- 問題文
  - `pow(x, n)`を実装せよ。 $x^n$ の計算。
  - 制約：
    - -100.0 < x < 100.0
    - -2^31 <= n <= 2^31-1
    - n is an integer.
    - Either x is not zero or n > 0.
    - -10^4 <= x^n <= 10^4

### 実装1

- アルゴリズムの選択
  - nが非常に大きいため、掛け算をn回繰り返すと時間がかかる。
  - 一方、x^nのレンジは比較的狭く、nが大きい場合底xは比較的小さめと思われる。
  - x^10 = x^(8 + 2) = x^8 * x^2 のような分解を考える。
  - 指数法則から、x^8 = (x^4)^2 = ((x^2)^2)^2 となり、x^2を2回自乗してたどり着ける。
  - この方法を使えばO(log n)で計算可能。
- 実装
  - loopで書く。[log n]回まわる。
  - 分解にはabs(n)に対するbit操作が使えそう。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
class Solution:
    def myPow(self, x: float, n: int) -> float:
        abs_n = abs(n)
        result = 1
        power = x
        for shift in range(abs_n.bit_length()):
            if abs_n >> shift & 1:
                result *= power
            power *= power
        
        if n < 0:
            return 1 / result
        return result
```

- ここまで20分。
- `power *= power`を当初`power **= 2`と書いていたところ、OverflowError: (34, 'Numerical result out of range')となり、困っていた。
  - （そもそも、x ** nはよく考えるとpow(x, n)なので禁止技だった。
  - GPT-5に聞くと；
    > - `power *= power` は浮動小数点の**乗算**です。IEEE 754 に従い、オーバーフローしても例外を出さずに `inf`（無限大）になります。
    > - `power **= 2` は内部的に浮動小数点の **べき演算**（libm の `pow`）を通ります。こちらはオーバーフロー時に `errno=ERANGE` を立て、Python では `OverflowError: (34, 'Numerical result out of range')` を送出します。
    >
    > 今回、反復自乗で `power` が `DBL_MAX≈1.797e308` を超えたタイミングで、`*=` だと黙って `inf` に落ち着くのに対し、`**=` だと例外が飛びます。これが差です。
  - ほえー。`power **= 2`の方でもtry-exceptでOverflowErrorをmath.infに置き換えると動いた。
- 補足：
  - n == 0のとき、forループがskipされてreturn 1となる。
    - (0).bit_length()は0。
  - x == 0のとき、forループは[log n]回まわるがずっとpower == 0であり、resultも0で上書きされる。
    - これはエッジケース高速化のために入力時点でreturn 0しても良いと思った。
  - n == 0 and x == 0のとき、forループがskipされてreturn 1となる。
    - 入力制約上は対応する必要はないが、1が返ったらいいと思っていたらたまたまなったのでよかった。
    - pythonのpow(0, 0)も1が返った。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.d9ky6ipkmw98)
- https://discordapp.com/channels/1084280443945353267/1262688866326941718/1351742235854639154
  - > IEEE-754の内部ビットの数も覚えておくと、面接でよく分かっている風が醸せることがあります。exponent が8ビットと11ビットです。符号が1ビットで残りが23ビットと52ビットです。
  - https://github.com/hroc135/leetcode/pull/43
  - https://ja.wikipedia.org/wiki/IEEE_754
  - 浮動小数点の処理は言語間で合意がある。
  - LLMだと表現力を効率よく上げるためにパラメータを量子化していたりする。
- https://discord.com/channels/1084280443945353267/1231966485610758196/1359997459232981234
  - > 私の感覚はこういう方が素直です。n x を破壊していないし、base bit の関係がはっきりしているからですね。
    - bit = 2^shiftで考え、ループ条件をbit <= nと書く方法。
    - 自分はshift == n.bit_length()をループ終了条件と見る方が（わかりやすくて）好みかもしれない。
- https://github.com/nanae772/leetcode-arai60/pull/44
  - https://discordapp.com/channels/1084280443945353267/1410553251409039410/1429045053627568138
    - `1.797e308`はfloat (64bit) で表現できる値の最大値。
    - sys.float_info.maxで閲覧できる。
    - 64bit浮動小数点はexponentが11bitで、biasも考慮してexponentの値域は[−1022, +1023]。よって最大値は概ね2^1024。
    - (2 − 2^−52) × 2^1023 = (1 − 2^−53) × 2^1024 ≈ 2^1024 ≈ 1.797...e+308

## Step 3

### 実装2

- bit = 2^shiftで考え、ループ条件をbit <= nと書く方法。

```python3
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n == 0:
            return 1
        if x == 0:
            return 0

        result = 1
        bit = 1
        power = x
        abs_n = abs(n)
        while bit <= abs_n:
            if abs_n & bit:
                result *= power
            power *= power
            bit <<= 1
        
        if n < 0:
            return 1 / result
        return result
```
