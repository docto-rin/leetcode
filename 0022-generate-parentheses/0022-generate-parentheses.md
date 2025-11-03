## Step 1

- 問題文
  - `n`対のカッコが与えられる。
  - 整合しているカッコのすべての組み合わせを生成する関数を記述せよ。
  - 制約
    - 1 <= n <= 8
- アルゴリズムの選択
  - 貪欲法が最初に思いついた。
    - カッコの組み合わせは常に長さ2 * nになる。
    - なので、2 * n回ループを回し、整合がつくように貪欲的に左から決めていけばいい。
    - "("の個数 - ")"の個数 を状態として逐次記録し、これを元に整合をチェックしていく。
  - 他には、最適部分構造を確立すればDPでもいけそう。
  - ひとまず1つ目の貪欲法ならsetの管理が不要なので1つ目でやる。

### 実装1

- 貪欲法（枝刈りつきBFS）
- 2重ループで実装
- 計算量
  - n対のカッコからなるすべての整合な組み合わせの数をmとすると、
  - Time: O(mn^2)
  - Space: O(mn)

```python3
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        frontiers = [([], 0)]  # (combination, num_open - num_close)
        for i in range(2 * n):
            next_frontiers = []
            for combination, diff in frontiers:
                if diff < 2 * n - 1 - i:
                    next_frontiers.append((combination + ["("], diff + 1))
                if diff > 0:
                    next_frontiers.append((combination + [")"], diff - 1))
            frontiers = next_frontiers
        
        return ["".join(combination) for combination, _ in frontiers]
```

- 6分で書き上げ、+3分でリファクタした。
- リストの新規作成型の結合を毎回行なっているのが良くなさそう。
- diff < 2 * n - 1 - iはパズルだけど、この時は避ける書き方がわからなかった。

### 実装2

- [実装1](#実装1)のcombinationをbitパターンで管理し、後で文字列に変換する方針。
- for2重ループ内でのリストの結合がビット操作に置き換わったので、時間計算量が落ちているはず。
- 計算量
  - Time: O(mn)
  - Space: O(mn)

```python3
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        dummy = 0
        frontiers = [(dummy, 0)]  # (bit_mask, num_open - num_close)
        for i in range(2 * n):
            next_frontiers = []
            for bit_mask, diff in frontiers:
                next_mask = bit_mask << 1
                if diff < 2 * n - 1 - i:
                    # add "("
                    next_frontiers.append((next_mask + 1, diff + 1))
                if diff > 0:
                    # add ")"
                    next_frontiers.append((next_mask, diff - 1))
            frontiers = next_frontiers
        
        def to_combination(bit_mask):
            combination = []
            for shift in range(2 * n - 1, -1, -1):
                if bit_mask >> shift & 1:
                    combination.append("(")
                else:
                    combination.append(")")
            return "".join(combination)
        
        return [to_combination(bit_mask) for bit_mask, _ in frontiers]
```

### 実装3

※誤った実装

- DP
  - n - 1 -> nにサイズが成長する時、以下の3つの操作を試せば網羅できそう。
    1. 左に"()"を追加
    2. 右に"()"を追加
    3. 左に"("、右に")"を追加
  - 異なる操作から同じ組み合わせが生成されうるので、setでの管理となりそう。

```python3
# wrong code

class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        bit_masks = set()
        dummy = 1
        bit_masks.add(dummy)
        for i in range(1, n + 1):
            nexts = set()
            for bit_mask in bit_masks:
                # bit 1 -> "(", bit 0 -> ")"

                # "()" + combination
                nexts.add(2 << 2 * i + bit_mask)
                # combination + "()"
                nexts.add(bit_mask << 2 * i + 2)
                # "(" + combination + ")"
                nexts.add(2 << 2 * i + bit_mask )
            bit_masks = nexts
        
        def to_combination(bit_mask):
            combination = []
            for shift in range(2 * n - 1, -1, -1):
                if bit_mask >> shift & 1:
                    combination.append("(")
                else:
                    combination.append(")")
            return "".join(combination)
        
        return [to_combination(bit_mask) for bit_mask, _ in frontiers]
```

- 網羅しきれていないらしい。
  - n==4の時、正解は14個あるが、この実装は13個しか返さなかった。
  - 漏れたのは '(())(())'
  - 確かにn==3の組み合わせがどこにもない。より小さいnも考慮しないといけないか。
- GPT-5にネタバレしてもらった。
  > カタラン数の分割則に基づく DP を使います。
  > - 有効な括弧列 `G(n)` は、ある `k` に対して `"(" + a + ")" + b` の形で書けます。ここで `a ∈ G(k)`、`b ∈ G(n-1-k)`。
  > - これで `dp[0] = [""]` を初期値として、`n` までbottom-upで構築します。
  - なるほど、新規追加するカッコは、"("は左端に固定して、")"を動かすということらしい。
- 備忘録
  - ビット操作：https://docs.python.org/ja/3.14/library/stdtypes.html
    > 二項ビット単位演算の優先順位は全て、数値演算よりも低く、比較よりも高くなっています; 単項演算 ~ の優先順位は他の単項数値演算 (+ および -) と同じです。

## Step 2

- カタラン数
  - https://ja.wikipedia.org/wiki/%E3%82%AA%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%B3%E6%95%B4%E6%95%B0%E5%88%97%E5%A4%A7%E8%BE%9E%E5%85%B8%E3%81%AE%E3%83%AA%E3%82%B9%E3%83%88
  - https://ja.wikipedia.org/wiki/%E3%82%AB%E3%82%BF%E3%83%A9%E3%83%B3%E6%95%B0
  - https://oeis.org/A000108
  - 確かに、カタラン数になっている。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.lqk42foli42r)
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1230529256358940722
    - remain_left, remain_rightのように2次元で状態を持つと可読性の高いコードが書けた。
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1356544668174651504
    - > はじめの括弧とそれに対応する括弧に注目して「(A)B」と分けるのも分類ですね。
    - なるほど。DPはこの言語化が最もしっくりくる。

### 実装4

- BacktrackingするDFSを再帰関数で実装。
- 計算量
  - Time: O(mn)
  - Additional Space: O(n)
  - Output Space: O(mn)

```python3
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        results = []
        path = []

        def dfs(open_used, close_used) -> None:
            if open_used == n and close_used == n:
                results.append("".join(path))
                return

            # Place '(' if we still have budget.
            if open_used < n:
                path.append("(")
                dfs(open_used + 1, close_used)
                path.pop()

            # Place ')' if we can close something.
            if close_used < open_used:
                path.append(")")
                dfs(open_used, close_used + 1)
                path.pop()

        dfs(0, 0)
        return results
```

## Step 3

### 実装5

- Backtrackingをloopで実装。

```python3
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        results = []
        stack = [([], n, n)]
        while stack:
            path, open_remain, close_remain = stack.pop()
            
            if open_remain == 0 and close_remain == 0:
                results.append("".join(path))
                continue
            if open_remain > 0:
                stack.append((path + ["("], open_remain - 1, close_remain))
            if close_remain > open_remain:
                stack.append((path + [")"], open_remain, close_remain - 1))

        return results
```
