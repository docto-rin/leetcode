# Step 1

小文字の英語のみからなる文字列`s`が与えられる。1度しか現れない文字で、最初に現れる文字の`s`におけるindexを返す。存在しないなら-1を返す。

- アルゴリズムの選択
  - 問題の性質上、前から1文字ずつ見ていったとして最後まで見ないと答えがわからない。
  - collections.Counterを使おうと思ったが、元の文字列におけるindexが記憶できないので、自分で辞書を構築することにする。
  - 辞書は、初めて出会う文字をkeyとし、出会ったindexをvalueでエントリさせる。2回目に出会うとvalueを-1で上書きする。
  - 他の方法は思いつかなかった。
    - str型のメソッドなどを用いた効果的なやり方があるのだろうか？
- 実装の方針
  - dict型が挿入順序を保持することを2回目のループで活用する。

### 実装1

- 時間計算量: O(n)
- 空間計算量: O(1)

```python3
class Solution:
    def firstUniqChar(self, s: str) -> int:
        seen = {} # character: first_index
        for index, character in enumerate(s):
            if character not in seen:
                seen[character] = index
                continue
            if seen[character] >= 0:
                # repeated
                seen[character] = -1

        for index in seen.values():
            if index >= 0:
                return index
        return -1
```

Runtime: 43 ms

ここまで6分。

Follow-up的なものが来るとしたらどうなるだろうか。

- Q. 大文字も受け入れる場合、実装は変えますか？
  - A. character.lower()で小文字に正規化します。大文字小文字の違いは区別しない方がよいと考えました。
- Q. 文字列が非常に長い場合、アルゴリズムは変更しますか？
  - A. -1が代入される回数をカウントし、それが文字種の数（今回なら26）に達したら早期returnします。

これくらいしか思いつかない。

### 実装2

一応早期return版

```python3
import string


class Solution:
    def firstUniqChar(self, s: str) -> int:
        NUM_UNIQUE_CHARACTERS = len(string.ascii_lowercase)
        
        seen = {} # character: first_index
        num_repeated_characters = 0
        for index, character in enumerate(s):
            if character not in seen:
                seen[character] = index
                continue
            if seen[character] >= 0:
                # repeated
                seen[character] = -1
                num_repeated_characters += 1
                if num_repeated_characters == NUM_UNIQUE_CHARACTERS:
                    return -1

        for character, index in seen.items():
            if index >= 0:
                return index
        return -1
```

Runtime: 44 ms

## Step 2

- https://github.com/colorbox/leetcode/pull/29/files#r1861430039
  - > 1-pass で処理するにあたり、文字をキューに入れていき、 2 回以上出現する文字を取り除いていくというやり方を考えました。
  - 一応実装してみます。

### 実装3

```python3
import string
from collections import defaultdict, deque


class Solution:
    def firstUniqChar(self, s: str) -> int:
        NUM_UNIQUE_CHARACTERS = len(string.ascii_lowercase)
        
        counter = defaultdict(int)
        characters = deque()
        num_repeated_characters = 0
        for index, character in enumerate(s):
            counter[character] += 1
            if counter[character] == 1:
                characters.append((character, index))
                continue
            while characters and counter[characters[0][0]] >= 2:
                characters.popleft()
                num_repeated_characters += 1
                if num_repeated_characters == NUM_UNIQUE_CHARACTERS:
                    return -1
        if characters:
            return characters[0][1]
        return -1
```

Runtime: 95 ms

補足：deque()にはindexだけ入れておけば十分。しかし入力文字列が1文字ずつ揮発していく状況ならcharacterも必要。

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.hdbcrwo3urck)
  - > LinkedHashMap
    - pythonの辞書は挿入順序を配列で記憶してしまうので省メモリにしようがないとのこと。
  - > LRU
    - Least Recently Used という意味で、使わないやつから削除する
    - pythonには[@functools.lru_cache(user_function)](https://docs.python.org/3/library/functools.html#functools.lru_cache)というデコレータがある。
      - スレッドセーフ。
      - 結果のキャッシュに辞書を使うので修飾先の関数の引数はhashableの必要がある。
      - [@functools.cache(user_function)](https://docs.python.org/3/library/functools.html#functools.cache)の方が軽く、速い。
  - https://github.com/quinn-sasha/leetcode/pull/15#discussion_r1969949923
    - > 異常な文字が来た場合には IndexError になりますが、このエラーを見たときに入力の文字列がおかしいと判断するのは難しそうです。入力が正しいかを確認したい場合はそのための Validation を始めに行うのが適切かなと思いました。
      > また Python では some_list[-1] のように負の index でも要素にアクセスできるので、このケースだと -26 ~ -1 の範囲であれば想定外の入力でもエラーが出ずに動いてしまい、思いがけないバグが起きそうです。（たとえば ^ が入った場合は ord('^') - ord('a') = -3 となるのであたかも 'x' かのように動いてしまう）
    - [実装2](#実装2)、[実装3](#実装3)は26種類repeatしたのを確認したら早期終了するので、これは異常な入力に対しやや脆弱。
    - [実装1](#実装1)では、常に最後まで走査する代わりに、小文字英語以外もそのまま扱う。
  - https://discord.com/channels/1084280443945353267/1307605446538039337/1333395364098609279
    > 計算量的には2乗になっていますが、find rfind がネイティブコードなので結構速そうですね。
    - なるほど、左から探しても右から探しても同じ位置、という言い換えは思いつかなかった。

### 実装4

```python3
class Solution:
    def firstUniqChar(self, s: str) -> int:
        for index, character in enumerate(s):
            if s.find(character) == s.rfind(character):
                return index
        return -1
```

Runtime: 67 ms

- 確かに計算量オーダーとは裏腹に速い。C実装の恩恵を感じる。
- str型のメソッドに慣れたい。発想できていない。

## Step 3

```python3
class Solution:
    def firstUniqChar(self, s: str) -> int:
        first_indices = {}
        for index, character in enumerate(s):
            if character not in first_indices:
                first_indices[character] = index
                continue
            if first_indices[character] >= 0:
                # repeated
                first_indices[character] = -1
        
        for index in first_indices.values():
            if index >= 0:
                return index
        return -1
```
