## Step 1

文字列が格納された配列`strs`を受け取る。アナグラムになっているものをグルーピングして別々の配列に格納し、グループごとの配列を並べた配列を返す問題。

- アルゴリズムの選択
  - strs[i]がどのアナグラムグループに所属するかを一意に決めるものは、その文字列内で各文字が何回出現するかを記録した辞書、つまりcounter = collections.Counter(strs[i])だと考えた。
  - strsを線形に走査し、counterにし、見たことのあるcounterならそのアナグラムグループに追加し、見たことがなければ新しいアナグラムグループを作る、という方針。
  - 他には思いつかず、このアルゴリズムを選択することにした。
- 実装の方針
  - 文字列はiterableなので、counter = Counter(strs[i])で文字のカウントはできる。
  - counter同士の比較（実質はdict同士の比較）は、==でできるか（特にkeyの挿入順の違いの扱いなど）が心配だったので、自前で実装。

### 実装1

n = strs.length, l = strs[i].length として、

- 時間計算量: O(n^2 * l)
- 空間計算量: O(n)

1 <= strs.length <= 10^4 なので、最悪計算時間が割とひどそう。
二重ループになっているので、一重にする工夫が必要か。

```python3
from typing import List
from collections import Counter

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        def is_same_dict(dict1, dict2):
            if dict1.keys() != dict2.keys():
                return False
            for key in dict1:
                if dict1[key] != dict2[key]:
                    return False
            return True
        
        def categorize_query(counter_query, counters_for_groups):
            for i, counter_key in enumerate(counters_for_groups):
                if is_same_dict(counter_query, counter_key):
                    return i
            return None
        
        strs_group_by_anagrams = []
        counters_for_groups = []
        for str_query in strs:
            counter_query = Counter(str_query)
            group_index = categorize_query(counter_query, counters_for_groups)
            if group_index is None:
                counters_for_groups.append(counter_query)
                strs_group_by_anagrams.append([str_query])
                continue
            strs_group_by_anagrams[group_index].append(str_query)
        return strs_group_by_anagrams
```

特に実装においてつまずかなかった。

https://discord.com/channels/1084280443945353267/1221030192609493053/1225674901445283860 で紹介されている構造化を用いた。

ここまで17分。

提出したところ、Runtime 7116 msで、やはり遅い。

### 実装2

- dictは==で比較すれば良いとのことだった。
  - https://docs.python.org/ja/3/library/stdtypes.html#mapping-types-dict
    - 再帰的にkey: valueペアが一致しているかをチェックしてくれる。
  > Dictionaries compare equal if and only if they have the same (key, value) pairs (regardless of ordering).
  - https://docs.python.org/3/library/collections.html#collections.Counter
  > Counters support rich comparison operators for equality, subset, and superset relationships: ==, !=, <, <=, >, >=. All of those tests treat missing elements as having zero counts so that Counter(a=1) == Counter(a=1, b=0) returns true.
  > Changed in version 3.10: Rich comparison operations were added.
  > Changed in version 3.10: In equality tests, missing elements are treated as having zero counts. Formerly, Counter(a=3) and Counter(a=3, b=0) were considered distinct.

- そして、counterは==で比較可能なので、in演算子も使えるということ。
  - 配列の代わりに、counter: [anagrams] という辞書`counter_to_anagrams`を管理し、if counter in counter_to_anagrams:でチェックするようにすれば、ループを1個削れる。
  - しかし実際には、Counterはunhashableなので辞書のkeyにはできない。hashableなkeyへマッピング（正規化）すればよい。([character, count], ...)というタプルにする。

- 時間計算量: O(n*l)
- 空間計算量: O(n)

```python3
from typing import List
from collections import defaultdict, Counter

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        counter_to_anagrams = defaultdict(list)
        for string in strs:
            counter = Counter(string)
            counter_to_anagrams[tuple(sorted(counter.items()))].append(string)
        return [anagrams for anagrams in counter_to_anagrams.values()]
```

Runtime 38 ms まで落とせた。

### 実装3

Counterを経由することで、定数倍分遅くなるので、直接タプルをビルドした方が速い。

その際、ordを使うアイデアをGPT-5からもらった。

https://docs.python.org/ja/3.11/library/functions.html#ord

```python3
from typing import List, Tuple
from collections import defaultdict, Counter

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        NUM_CHARACTERS = 26
        BASE_CHARACTER = 'a'

        def count(string: str) -> Tuple[int]:
            counts = [0] * NUM_CHARACTERS
            base = ord(BASE_CHARACTER)
            for character in string:
                counts[ord(character) - base] += 1
            return tuple(counts)

        counter_to_anagrams = defaultdict(list)
        for query_string in strs:
            query_counter = count(query_string)
            counter_to_anagrams[query_counter].append(query_string)
        return list(counter_to_anagrams.values())
```

Runtime 19 ms まで落とせた。アルゴリズムは常に定数倍が重要。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.e5dwa7yj3tv0)
  - > 小文字アルファベット以外が来ると何が起きるか考えておきましょう。どうでなくてはいけないというよりは、その帰結としてありうるシナリオの幅を広く考えておきたい、くらいの意図です。
    - 入力に対するチェック（要件定義）は習慣づけたいと思っているが、余裕がないと忘れてしまう。
    - 本問は少し余裕がなく、[実装1](#実装1)を書き終わってから初めて意識しだした。
  - > ord からフォローアップの質問でユニコードのコードポイントの話などが想定されます。また、入力が、小文字アルファベットでないものが来たときに、どのような振る舞いをするか、どのような振る舞いをするべきかは追加質問が来てもおかしくないでしょう。
    - > ありがとうございます。ふと、ASCIIのコードポイントを確認したのですが、大文字の方が先だったり、アルファベットで連続してるわけではなかったり、ちゃんと知らなかった発見がありました
    - > つまり、Z がくると-7なので後ろから7番目が増えますね。そういう認識があるかです。
      - この観点では、[実装2](#実装2)の方が[実装3](#実装3)より柔軟といえる。
    - ASCII (American Standard Code for Information Interchange): 1960年代に作られた、英数字と制御文字のみを扱う 7ビットの文字コード表。
      | 種類   | 範囲              | 例                       |
      | ---- | --------------- | ----------------------- |
      | 制御文字 | 0–31, 127       | 改行, タブ, BEL など          |
      | 記号   | 32–47, 58–64, … | `! @ # $ %` など          |
      | 数字   | 48–57           | `'0' = 48`, `'9' = 57`  |
      | 大文字  | 65–90           | `'A' = 65`, `'Z' = 90`  |
      | 小文字  | 97–122          | `'a' = 97`, `'z' = 122` |
    - Unicode は「全世界の文字を一意に表す」ための拡張規格。
      - ASCII を含む形で、すべての文字に一意な「コードポイント (U+XXXX)」 を割り当てている。
      - Unicodeの先頭128コードポイント (U+0000〜U+007F) がそのまま ASCII
  - > この場合の問題文ではないようですが、数字が入力に来ると衝突しますね。どういうエンコードにすると衝突しなくなりますか。
    - > エスケープシーケンスとかが模範解答でしょうかね。可変長数値表現も参照。
      > https://ja.wikipedia.org/wiki/%E5%8F%AF%E5%A4%89%E9%95%B7%E6%95%B0%E5%80%A4%E8%A1%A8%E7%8F%BE
      > あと、タプルにするのも手です。

## Step 3

### 実装4

([character, count], ...) をキーとする方法。（可変長）

```python3
from typing import List
from collections import defaultdict, Counter

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        counter_to_anagrams = defaultdict(list)
        for string in strs:
            counter = Counter(string)
            counter_to_anagrams[tuple(sorted(counter.items()))].append(string)
        return list(counter_to_anagrams.values())
```

### 実装5

(a_count, b_count, ...) をキーとする方法。（固定長）

```python3
from typing import List, Tuple
from collections import defaultdict

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        BASE_CHARACTER = 'a'
        NUM_CHARACTERS = 26
        
        def count_character(string: str) -> Tuple[int]:
            base = ord(BASE_CHARACTER)
            counter = [0] * NUM_CHARACTERS
            for character in string:
                counter[ord(character) - base] += 1
            return tuple(counter)
        
        counter_to_anagrams = defaultdict(list)
        for string in strs:
            counter_to_anagrams[count_character(string)].append(string)
        return list(counter_to_anagrams.values())
```

- 実装4の方が汎用的だが、実装5の方がこの問題のスコープに最適化（特化）されている。
- 実装5はアルファベットの小文字に限定しており、それ以外を使う場合は内部パラメータを変える。
  - 空白地帯が生じる場合は別途キーの表現を検討する余地がある。
