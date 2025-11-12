## Step 1

- 問題文
  - 入力文字列`s`を、`numRows`行のzigzag patternで書いて、行ごとに読み上げた結果に変換せよ。
  - 制約：
    - 1 <= s.length <= 1000
    - s consists of English letters (lower-case and upper-case), ',' and '.'.
    - 1 <= numRows <= 1000

### 実装1

- アルゴリズムの選択
  - 行ごとに`numRows`個のリストを作り、入力文字列を線形走査して正しい行にappendしていき、最後に"".join()を使ってくっ付ければ良さそう。
  - 行インデックスは、numRows - 1回のインクリメントとnumRows - 1回のデクリメントを繰り返すので、この2つのブロックに分け、往復合わせて1サイクルのイメージ。
  - 一応、s_index -> 行インデックス への変換は解析的に求められそうで、逆もなんとかできそうなので、逐次的に返す必要があれば補助空間O(1)にもできるな、というくらいだが、可読性が下がる。
- 実装
  - 最後にoutputを準備するとき、遅い方法にならないように気をつける。
    - str + strは使わないようにする。
- 計算量
  - Time: O(n)
  - Space: O(n)

```python3
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        if not s:
            return ""
        if numRows <= 0:
            raise ValueError(f"numRows '{numRows}' must be positive integer")
        if numRows == 1:
            return s
        
        s_index = 0
        row = 0
        rows = [[] for _ in range(numRows)]
        while s_index < len(s):
            while s_index < len(s) and row < numRows - 1:
                rows[row].append(s[s_index])
                s_index += 1
                row += 1
            while s_index < len(s) and row > 0:
                rows[row].append(s[s_index])
                s_index += 1
                row -= 1
        
        result = []
        for row in rows:
            result.extend(row)
        return "".join(result)
```

### 実装2

- ジェネレータを使って書いてみた。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)
  - Output Space: O(n)

```python3
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        if not s:
            return ""
        if numRows <= 0:
            raise ValueError(f"numRows '{numRows}' must be positive integer")
        if numRows == 1:
            return s
        
        def generate_zigzag():
            cycle = 2 * (numRows - 1)
            for row in range(numRows):
                cycle_count = -1
                while True:
                    cycle_count += 1

                    s_index = cycle * cycle_count + row
                    if s_index >= len(s):
                        break
                    yield s[s_index]

                    if row == 0 or row == numRows - 1:
                        continue
                    
                    s_index = cycle * (cycle_count + 1) - row
                    if s_index >= len(s):
                        break
                    yield s[s_index]
        
        return "".join(generate_zigzag())
```

- [実装1](#実装1)とはdownとupの分割の仕方を変えた。upは 0 < row < len(s) - 1 のような中間の行だけを考える。
- cycle_count = -1の初期化は気持ち悪いが、whileループの末尾がrowに依存して変わるので、共通して書くならループ冒頭になるため、やむなく。

## Step 2

- コメント集
  - https://discord.com/channels/1084280443945353267/1347375192065703986/1353577012593426502
    - > return "".join(c for line in display_board for c in line)
    - 目が横に振れる... ジェネレータで2重ループしたいときは便利そう。
    - GPT-5にlist.extendの方法と比較させた。
      > - 比較対象の中で `method_extend` はだいたい `0.34x`〜`0.64x` 程度（つまり 34%〜64%）の時間で済んでおり、今回のベンチでは内包表記の方が遅いという結果になりました。
      > - `extend` の方は既存のリストに対して要素をまとめて追加してから一度だけ `join` を呼ぶため、Python の内部処理で効率的に動いているようです。
  - https://discord.com/channels/1084280443945353267/1237649827240742942/1355850131483918428
    - > Java や Python など文字列が immutable な言語では重要な話ですが、C++ では、mutable で後ろに文字をつける分には大きな問題になりません。
      > 前後に付けたり分割しらたりなどする必要があるならば、Rope というデータ構造などを使えばいいですが、そこまでする必要があることは少ないです。
    - C++はmutableらしい。
    - RopeはSWEの常識には含まれなさそう。
      - https://en.wikipedia.org/wiki/Rope_(data_structure)
      - Ropeは二分木の一種。
      - 葉は文字列をもち、すべての葉を左からconcatすると文字列が復元される。
      - 各ノードの重みは、左部分木の葉の文字数の合計。
      - 細かい分割を諦めることで軽くなるという理解。
      - tokenizeとかと関係しそう。（tokenごとに葉に分ければ十分、など。）
  - https://discord.com/channels/1084280443945353267/1347375192065703986/1353609363427819521
    - > 手続き型の手法で構造を組み合わせられるかが想定だろうなと思います。
    - Generatorは自然と思いつけたので良かった。
    - 最近は、可能なら関数型で書けないかをなんとなくイメージする習慣が芽生えた。
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1360627334792614041
    - > itertools.batched ... あんまりぱっとしませんかね。
    - 結局上のrowから全体をyieldするためには、batchedを何周もするのでぱっとしないということだと思う。
- https://github.com/shintaro1993/arai60/pull/64/
  - len(s) <= numRows: return sは、空配列を余計に作らないで済むパターンのearly return。
  - 一重ループで書いて、インクリメントの方向（単位）を両端にいくごとに変えていく。これは読みやすい。
  - 一重ループにすればfor c in s: で回せるのも良さそう。

## Step 3

### 実装3

- [実装1](#実装1)のリファクタ

```python3
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        if not s:
            return ""
        if numRows <= 0:
            raise ValueError(f"numRows '{numRows}' must be positive")
        if numRows == 1 or len(s) <= numRows:
            return s
        
        row = 0
        rows = [[] for _ in range(numRows)]
        row_direction = 1
        for ch in s:
            rows[row].append(ch)
            row += row_direction

            if row == numRows - 1:
                row_direction = -1
            if row == 0:
                row_direction = 1
        
        result = []
        for row_chars in rows:
            result.extend(row_chars)
        return "".join(result)
```
