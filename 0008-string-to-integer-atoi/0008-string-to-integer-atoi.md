## Step 1

- 問題文
  - 文字列を32ビット符号付き整数に変換するmyAtoi(s: str)関数を実装せよ。
  - 例：
    - "42" -> 42
    - " -042" -> -42
    - "1337c0d3" -> 1337
    - "0-1" -> 0
    - "words and 987" -> 0

### 実装1

- アルゴリズムの選択
  - ナイーブにはsを線形走査しても可能だが、正規表現が一挙にパースできて良さそう。
- 実装
  - 規則をpatternで表現し、matchしなければreturn 0とし、matchしたらint(sign + digits)のように整数化する。
  - 整数を作った後に32bitに丸めるようにreturnする。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
import re


class Solution:
    def myAtoi(self, s: str) -> int:
        whole_match = re.match("\s*([\+\-]?)([0-9]+)", s)
        if whole_match:
            sign = whole_match.group(1)
            digits = whole_match.group(2)
        else:
            return 0
        
        digits_match = re.match("0*(.+)", digits)
        if digits_match:
            digits = digits_match.group(1)
        else:
            return 0
        
        signed_integer = int(sign + digits)
        if signed_integer < -2**31:
            return -2**31
        if signed_integer > 2**31 - 1:
            return 2**31 - 1
        return signed_integer
```

- ここまで26分。
- reモジュールを使い慣れておらず、動作を確かめながら書いたため時間がかかった。
  - https://docs.python.org/ja/3/library/re.html
  - https://docs.python.org/ja/3/library/re.html#match-objects
  - https://github.com/docto-rin/leetcode/pull/14/
- 条件分岐が適当で、動きはするが全然ダメに感じる。

## Step 2

- GPT-5によるレビュー
  > * 先頭ゼロの除去ロジックにバグがあります。`re.match("0*(.+)", digits)` は `digits="0"` のとき不一致になり `return 0`。`atoi("0")` は `0` を返すべきなので、ここは `lstrip('0')` で空になったら `"0"` にするのが安全です。
  - バグではないが、ロジックが洗練されてない。
  > * 正規表現リテラルは Python では raw 文字列にするのが通例です（`\s` や `\+` を含むため）。`r"^\s*([+\-]?)(\d+)"` のように記述してください。
  - パターンの方にrをつけないといけない。
  > * 正規表現の先頭アンカー `^` を付けると意図が明確になります（`re.match` は先頭からですが、読み手に優しい）。
  - 一応re.matchは先頭を見るので冗長だが、あっても良いかも。
  > * 2 回目の正規表現は不要です。先頭ゼロ除去は文字列操作で十分です。
  - 正規表現マッチと文字列操作を組み合わせるというのは前の問題でも使った。
  - https://docs.python.org/ja/3/library/stdtypes.html#text-and-binary-sequence-type-methods-summary
  - https://docs.python.org/ja/3/library/stdtypes.html#str.lstrip
  - https://github.com/docto-rin/leetcode/pull/14/
  > * パフォーマンス観点では Python の `int` は任意精度なのでオーバーフローは起きませんが、必要なら走査中にしきい値で早期クランプする実装の方が堅牢です。
  - これも、桁数が膨大な時にint()で重くなるのを防ぐために必要だが、サボってしまった。
  > * マジックナンバーを定数に。`INT_MIN = -2**31`, `INT_MAX = 2**31 - 1`。
  - これも重要。他言語では実際定数になっているはず。
  > * 例外系の早期 return は維持しつつ、分岐数を減らして読みやすく。
  > * 正規表現版を残す場合は `re.compile` を使い、毎回のパターンコンパイルを避ける。
  - https://docs.python.org/ja/3/library/re.html#re.compile
  - > 注釈 re.compile() やモジュールレベルのマッチング関数に渡された最新のパターンはコンパイル済みのものがキャッシュされるので、一度に正規表現を少ししか使わないプログラムでは正規表現をコンパイルする必要はありません。
  - しかし、クラス属性（≠ インスタンス属性）として書いておけば便利なので、本問もコンパイルした方がいいだろう。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.j1ggirt9v3pc)
  - https://discord.com/channels/1084280443945353267/1251052599294296114/1281670288194539540
    - > 一応、MIN = - MAX - 1 は念頭にありますね。
    - また、MIN == ~MAX
    - なお、2の補数における符号反転は、ビット反転して1を加算するので、-MAX = ~MAX + 1、故にMIN = ~MAX = -MAX - 1
  - https://discord.com/channels/1084280443945353267/1237649827240742942/1354495583494082580
    - > 漢字などでも True になるのは Python の isdigit の話ですね。
    - `str.isdecimal()`, `str.isdigit()`, `str.isnumeric()`があるらしい。
      - `isdecimal()`: 最も厳密。Unicode の一般カテゴリ"Nd"（Number, Decimal unit）、つまり全角も含めたアラビア数字（十進数字という）だけを許す。
        - Numeric_Type=Decimal
      - `isdigit()`  : `isdecimal()` が許すものに加えて、上付き数字、丸数字、カッコ数字などのあらゆる数字文字（Digit）もTrueにする。bytesに対しても同名のメソッドがある。
        - Numeric_Type=Decimal or Numeric_Type=Digit
      - `isnumeric()`: 最も広い。数字文字と、Unicode数値プロパティを持つすべての文字をTrueにする。ローマ数字や漢数字などもここでやっと入る。ただし、bytesには未定義。
        - Numeric_Type=Decimal or Numeric_Type=Digit or Numeric_Type=Numeric 
      - https://docs.python.org/ja/3/library/stdtypes.html#str.isdecimal
      - https://docs.python.org/ja/3/library/stdtypes.html#str.isdigit
      - https://docs.python.org/ja/3/library/stdtypes.html#bytes.isdigit
      - https://docs.python.org/ja/3/library/stdtypes.html#str.isnumeric
      - 本問はisdecimalが良さそう。
      - isdecimalは例えば上付き文字を許すが、int()は許さない。
      - isnumericは広すぎる上、そもそも整数が保証されないので適さない。
    - Numeric_TypeはUnicodeの数値プロパティ。定義が曖昧。
      - https://www.unicode.org/L2/L2012/12310-numeric-type-def.html
    - Unicodeの数字を表すカテゴリは3種類ある。Numeric_Typeとは別の分類であることに注意。
      - Number, Decimal unit: いわゆる数字。Ndで表す。
      - Number, Letter: ローマ数字（Ⅷとか）など。Nlで表す。
      - Number, other: その他の数字(⅔など)。いわゆる機種依存文字(⓴など)の多くもここに掃き寄せられている。Noで表す。
      - https://www.fileformat.info/info/unicode/category/Nd/list.htm
      - https://www.fileformat.info/info/unicode/category/Nl/list.htm
      - https://www.fileformat.info/info/unicode/category/No/list.htm
    - pythonの標準モジュールでUnicode Character Database (UCD)にアクセスして数値を取得してくれる関数がある。
      - https://docs.python.org/ja/3/library/unicodedata.html
      - https://docs.python.org/ja/3/library/unicodedata.html#unicodedata.numeric
  - https://discord.com/channels/1084280443945353267/1237649827240742942/1354495583221579887
    - > 7 は、numeric_limits::max() % 10 ということですかね。おそらく、コンパイラが定数にしてくれるので、素直にこう書いてしまったらいいでしょう。
    - Constant Folding（定数畳み込み）というらしい。pythonにも存在するらしい。
    - GPT-5曰く、
      > * Python（CPython）では「リテラルだけの式」はコンパイル時に畳み込まれるが、`sys.maxsize % 10` のように名前や属性が入る式は畳み込まれない。
      > * 可読性優先で素直に書いてよい。高頻度のパスならモジュール定数化やローカルキャッシュを使うと速くなる。プロファイラで確認してから最適化することを推奨します。 

```python3
import unicodedata

'123'.isdecimal()    # True
'123'.isdigit()      # True
'123'.isnumeric()    # True
int('123')           # 123

# 全角
'１２３'.isdecimal()  # True
'１２３'.isdigit()    # True
'１２３'.isnumeric()  # True
int('１２３')         # 123

# 上付き数字
'²'.isdecimal()      # False
'²'.isdigit()        # True
'²'.isnumeric()      # True
int('²')             # ValueError: invalid literal for int() with base 10: '²'

# ローマ数字（Unicode文字）
'Ⅳ'.isdecimal()     # False
'Ⅳ'.isdigit()       # False
'Ⅳ'.isnumeric()     # True
int('Ⅳ')            # ValueError: invalid literal for int() with base 10: 'Ⅳ'
unicodedata.numeric("Ⅳ")  # 4.0

# ローマ数字（アルファベットで入力したもの, ASCII）
'IV'.isdecimal()     # False
'IV'.isdigit()       # False
'IV'.isnumeric()     # False
int('IV')            # ValueError: invalid literal for int() with base 10: 'IV'

# 漢数字
'四'.isdecimal()     # False
'四'.isdigit()       # False
'四'.isnumeric()     # True
int('四')            # ValueError: invalid literal for int() with base 10: '四'
unicodedata.numeric("四")  # 4.0

# 分数
'½'.isdecimal()      # False
'½'.isdigit()        # False
'½'.isnumeric()      # True
int('½')             # ValueError: invalid literal for int() with base 10: '½'
unicodedata.numeric('½')  # 0.5

# bytes オブジェクト
b'1'.isdecimal()     # AttributeError: 'bytes' object has no attribute 'isdecimal'
b'1'.isdigit()       # True
b'1'.isnumeric()     # AttributeError: 'bytes' object has no attribute 'isnumeric'
int(b'1')            # 1
unicodedata.digit(b'1')  # TypeError: digit() argument 1 must be a unicode character, not bytes
```

### 実装2

- [実装1](#実装1)をリファクタ
- 桁数によるオーバーフロー判定は未実装
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
import re


class Solution:
    INT_MIN = -2**31
    INT_MAX = 2**31 - 1
    _PATTERN = re.compile(r"^\s*([+-]?)(\d+)")

    def myAtoi(self, s: str) -> int:
        m = self._PATTERN.match(s)
        if not m:
            return 0

        if m.group(1) == "-":
            sign = -1
        else:
            # "+" or ""
            sign = 1
        
        digits = m.group(2).lstrip("0") or "0"

        value = sign * int(digits)
        if value < self.INT_MIN:
            return self.INT_MIN
        if value > self.INT_MAX:
            return self.INT_MAX
        return value
```

### 実装3

- forループ
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python3
class Solution:
    INT_MAX = 2**31 - 1
    INT_MIN = -2**31
    
    def myAtoi(self, s: str) -> int:
        i, n = 0, len(s)

        # skip spaces
        while i < n and s[i] == " ":
            i += 1
        if i == n:
            return 0
        
        # determine sign
        if s[i] == "-":
            sign = -1
            i += 1
        elif s[i] == "+":
            sign = 1
            i += 1
        else:
            sign = 1
        
        if sign == -1:
            abs_limit = -self.INT_MIN
        else:
            abs_limit = self.INT_MAX
        
        # read digits
        value = 0
        while i < n and s[i].isdecimal():
            digit = int(s[i])
            # check overflow: 10 * value + digit > abs_limit
            if value > (abs_limit - digit) // 10:
                return sign * abs_limit
            value = value * 10 + digit
            i += 1
        
        return sign * value
```

- if value > (limit_abs - digit) // 10で、/ -> //へは、入力の離散性から同値変形できる。（非自明）
- `# determine sign`のところは、sign = 1と初期化する書き方もあるが、対比的な上の書き方がわかりやすいと思った。

## Step 3

### 実装4

- [実装2](#実装2)ベース。

```python3
import re


class Solution:
    INT_MAX = 2**31 - 1
    INT_MIN = -2**31
    _PATTERN = re.compile(r"\s*([+-]?)(\d+)")
    
    def myAtoi(self, s: str) -> int:
        match = self._PATTERN.match(s)
        if not match:
            return 0
        
        if match.group(1) == "-":
            sign = -1
            abs_limit = -self.INT_MIN
        else:
            sign = 1
            abs_limit = self.INT_MAX
        
        digits = match.group(2).lstrip("0") or "0"
        if len(digits) > len(str(abs_limit)):
            return sign * abs_limit

        value = int(digits)
        if value > abs_limit:
            return sign * abs_limit

        return sign * value
```
