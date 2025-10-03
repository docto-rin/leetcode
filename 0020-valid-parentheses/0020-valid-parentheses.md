## Step 1

- 3種類のカッコのみが並ぶ文字列sが、正しくカッコが閉じているか判定する問題。
- stackが良さそう。stack以外は思いつかない。
  - 前から一文字ずつ取得し、openが来た時はstackにpushし、closeが来た時はstackからpopしたopen文字との整合を見る。
  - return Falseとなる場面は、stackがpopできない時、popしたopenとcloseの不整合、文字列処理後にstackが空じゃない、の3つの状況。
- stackの実装方法は、
  - list(), collections.deque() が思いついた。特に今回の用途において違いは思いつかない。
  - O(1)でアクセスできるのが一端ずつに限られているlist()の方がLIFOに則していると思い、list()を採用。
  - https://docs.python.org/ja/3.13/library/queue.html をみると、queue.LifoQueue()なるクラスがある。
    - > The queue module implements multi-producer, multi-consumer queues. It is especially useful in threaded programming when information must be exchanged safely between multiple threads. The Queue class in this module implements all the required locking semantics.
    - list()のmulti threads用のwrapperクラスらしい。
    - 今回はstackへのアクセスはsingle threadで行うので使わないでおく。

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
class Solution:
    def isValid(self, s: str) -> bool:
        open2close = {
            "(": ")",
            "{": "}",
            "[": "]"
        }
        stack = []
        for char in s:
            if char in ["(", "{", "["]:
                stack.append(char)
            if char in [")", "}", "]"]:
                if stack == []:
                    return False
                expected = open2close[stack.pop()]
                if char != expected:
                    return False
        if stack != []:
            return False
        return True
```

ここまで7分。

境界条件の処理に対し、選択肢を引き出せずナイーブに処理してしまった。工夫の余地ありそうだなと感じる。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ns0bie22a6m)
  - > スタックの底に番兵を置いておくのも一応手としてはあります。
    - 境界条件に対する工夫の一案ですね。選択肢として出てこなかった。
  - > "(aiu)[eo]" が入力としてきたときに、プログラムの挙動として好ましいのは何だと考えますか?
    - 自分はTrueが返るべきだと考える。偶然にも自分の実装はこれを満たす。
    - [異常な入力への対処](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.jdtk9v35bca4)
    - "(aiu)[eo]" が万一に入力されることも考えられるようになりたい。
  - > ({[ の場合を先に処理して continue にしてインデント下げたほうが読みやすいでしょう。
    - 条件分岐の順番を工夫することで、インデントを下げられる（不要なandをなくせる）
- > stackよりopenbracketsの方がわかりやすい

その他、自分で思ったこと：

- あとは、["(", "{", "["] や [")", "}", "]"] について、
  - listで定義するよりsetで定義した方が良かった。{"(", "{", "["}
  - ハードコードせず、open2closeの.keys()や.values()を使って判定したい。
    - https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects
      > Keys views are set-like since their entries are unique and hashable. Items views also have set-like operations since the (key, value) pairs are unique and the keys are hashable. If all values in an items view are hashable as well, then the items view can interoperate with other sets. (Values views are not treated as set-like since the entries are generally not unique.) For set-like views, all of the operations defined for the abstract base class collections.abc.Set are available (for example, ==, <, or ^). While using set operators, set-like views accept any iterable as the other operand, unlike sets which only accept sets as the input.
    - .keys(), .items() はエントリが一意でhashableであるため、集合のようなビュー。
    - 一方.values() は一意とは限らないので集合として扱えない。
    - 丁寧に if char in set(open2close.values()): と書いた方がいいだろう。

## Step 3

```python3
class Solution:
    def isValid(self, s: str) -> bool:
        open2close = {
            "(": ")",
            "{": "}",
            "[": "]"
        }
        closes = set(open2close.values())
        found_opens = []
        for char in s:
            if char in open2close:
                found_opens.append(char)
            elif char in closes:
                if not found_opens:
                    return False
                expected = open2close[found_opens.pop()]
                if char != expected:
                    return False
        return not found_opens
```
補足：
- if not found_opens:をelif char in closes:内に入れた理由は、"a(iu)[eo]"も許容したいため。
- charは入力にカッコ以外がきても違和感ないような命名。
