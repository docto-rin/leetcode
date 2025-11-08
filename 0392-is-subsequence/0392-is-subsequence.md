## Step 1

- 問題文
  - 2つの文字列`s`と`t`が与えられる。`s`が`t`のsubsequneceならTrueを返し、そうでなければFalseを返せ。
  - A subsequence of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., "ace" is a subsequence of "abcde" while "aec" is not).
    - 制約
    - 0 <= s.length <= 100
    - 0 <= t.length <= 10^4
    - s and t consist only of lowercase English letters.

 ### 実装1

- アルゴリズムの選択
  - s、tをダブルポインタで前から線形走査していけば良さそう。
  - subsequenceが複数作れる場合でも、最も貪欲的なパターンだけ見つければ十分。
- 実装
  - ダブルポインタはwhile一重ループで書ける。
    - while内の条件分岐によりインクリメントの仕方が変わる。
    - 終了時にsのポインタがout of rangeまで進んでいればTrue
  - s, tが空文字列やNoneの場合を先に対処する。
- 計算量
  - Time: O(s.length + t.length) = O(t.length)
  - Extra Space: O(1)

```python3
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        if not s:
            return True
        if not t:
            return False
        
        s_index = 0
        t_index = 0
        while s_index < len(s) and t_index < len(t):
            if s[s_index] == t[t_index]:
                s_index += 1
            t_index += 1
        
        if s_index == len(s):
            return True
        return False
```

- ここまで5分。

### 実装2

- > Follow up: Suppose there are lots of incoming s, say s1, s2, ..., sk where k >= 10^9, and you want to check one by one to see if t has its subsequence. In this scenario, how would you change your code?
- アルゴリズムの再考
  - ターゲットtは固定だが、クエリsが次から次に来る時、コードを工夫できるかという問い。
  - tを前処理しておくことを考える。
    - char_to_positions: dict、つまり、文字種と、そのt内での出現位置をmappingするhashmapを作っておく。
    - すると、sのある文字種に対し、t内での次の出現位置にO(1)でアクセスでき、s1つあたりの時間計算量がO(t.length) -> O(s.length)に短縮する。
  - pros and cons
    - pros: クエリsが1回きりなら冗長な手順になるが、同じtに複数のsが来るのであればお得になる。
    - cons: 補助空間はhashmapを構築する分大きくなる。
    - pros > cons と判断できる。
- 実装
  - メソッドを分解し、維持しやすく書く。
  - char_to_positionsはクラスの状態として持つことにする。
- 計算量
  - 前処理
    - Time: O(t.length)
    - Space: O(t.length)
  - クエリ1回
    - Time: O(s.length * log(char_count)) <= O(s.length * log(t.length))

```python3
from typing import Optional
import collections


class Solution:
    def __init__(self):
        self.char_to_positions = None

    def build_char_positions_map(self, t: str) -> None:
        self.char_to_positions = collections.defaultdict(list)
        for i, char_ in enumerate(t):
            self.char_to_positions[char_].append(i)

    def are_subsequences(
        self,
        list_s: list[str],
        t: Optional[str] = None
    ) -> list[bool]:
        if not list_s:
            return []
        if t:
            self.build_char_positions_map(t)
        if self.char_to_positions is None:
            return [False for _ in range(len(list_s))]

        return [self.isSubsequence(s) for s in list_s]

    def isSubsequence(self, s: str, t: Optional[str] = None) -> bool:
        if not s:
            return True
        if t:
            self.build_char_positions_map(t)
        if self.char_to_positions is None:
            return False

        char_counter = collections.defaultdict(int)
        previous_position = -1
        for char_ in s:
            positions = self.char_to_positions[char_]
            char_counter[char_] += 1
            while char_counter[char_] <= len(positions):
                position = positions[char_counter[char_] - 1]
                if position > previous_position:
                    previous_position = position
                    break
                char_counter[char_] += 1
            else:
                return False
        return True
```

- 30分かかった。

## Step 2

- [実装1](#実装1)に対するGPT-5によるレビュー
  - > 返り値は`return s_index == len(s)` で十分です。
  - なるほど。
- [実装2](#実装2)に対するGPT-5によるレビュー
  - > * ただし現実のクエリ時間は O(|s|) ではなく、実装のままでは位置リストを前から舐めるため、各文字ごとにスキップ分だけ余計に回ります。クエリごとに最悪 O(|t|) まで膨らみます。
    - なるほど、follow-upの求める水準には達していなさそう。
    > * `defaultdict(list)` を `self.char_to_positions[char_]` で参照すると、存在しない文字にアクセスした際に空リストが辞書へ追加されます。副作用を避けるなら `dict.get(char_)` を使うのが無難です。
    - その通り。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ftci24or3a8g)
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1231637671831408821
    - （文字列を扱うのもあり）ネイティブコードを使って書けないか考えたが、findの開始インデックスを指定すれば制約を組み込めた。思いつかなかった。
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1243290893671465080
    - 関数型の表現と、Python（手続き型）での表現パターン。

### 実装3

- [実装2](#実装2)で、positionの検索に二分探索を使う。
- 計算量
  - 前処理
    - Time: O(t.length)
    - Space: O(t.length)
  - クエリ1回
    - Time: O(s.length * log(char_count)) <= O(s.length * log(t.length))

```python3
from typing import Optional
import collections
import bisect


class Solution:
    def __init__(self):
        self.char_to_positions: Optional[dict[str, list[int]]] = None

    def build_char_positions_map(self, t: str) -> None:
        self.char_to_positions = collections.defaultdict(list)
        for i, char_ in enumerate(t):
            self.char_to_positions[char_].append(i)

    def are_subsequences(
        self,
        list_s: list[str],
        t: Optional[str] = None
    ) -> list[bool]:
        if not list_s:
            return []
        if t:
            self.build_char_positions_map(t)
        if self.char_to_positions is None:
            return [False for _ in range(len(list_s))]

        return [self.isSubsequence(s) for s in list_s]

    def isSubsequence(self, s: str, t: Optional[str] = None) -> bool:
        if not s:
            return True
        if t:
            self.build_char_positions_map(t)
        if self.char_to_positions is None:
            return False

        last_positions = {}
        previous = -1
        for char_ in s:
            positions = self.char_to_positions.get(char_)
            last = last_positions.get(char_, -1)
            if positions is None:
                return False
            index = bisect.bisect_right(positions, previous, lo=last + 1)
            if index == len(positions):
                return False
            last_positions[char_] = index
            previous = positions[index]

        return True
```

## Step 3

### 実装４

- [実装1](#実装1)をネイティブコードを使えるように書いた。
- 計算量
  - Time: O(s.length + t.length) = O(t.length)
  - Extra Space: O(1)

```python3
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        if not s:
            return True
        if not t:
            return False
        
        position = -1
        for query in s:
            position = t.find(query, position + 1)
            if position == -1:
                return False
        return True
```
