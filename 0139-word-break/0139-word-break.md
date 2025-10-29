## Step 1

- 問題文
  - 文字列`s`と文字列の辞書（配列）`wordDict`が与えられる。
  - `wordDict`内の単語を重複を許して繋げることで`s`が作れるならTrueを返し、無理ならFalseを返せ。
  - 制約：
    - 1 <= s.length <= 300
    - 1 <= wordDict.length <= 1000
    - 1 <= wordDict[i].length <= 20
    - s and wordDict[i] consist of only lowercase English letters.
    - All the strings of wordDict are unique.
    - 以降の説明のため、n = s.length, m = wordDict.length, L = wordDict[i].length, と置いておく。

### 実装1

- アルゴリズムの選択
  - 最適部分構造はほとんど自明に確認できる。
  - 制約の1~3個により、sを左からチェックしていくと検索回数は多くてもn^2/2、setを使ってO(1)で検索すれば十分速い。
    - DFS/BFSで十分そう。
  - s（の一部）をクエリにwordDictを検索するか、wordDictの各単語をクエリにs（の一部）を検索するか。
    - wordDictの単語は必ずしも使うとは限らず、前者の方が比較回数が少なくなりそうに見える。
- 実装
  - DFSをloopで書く。
- 計算量
  - Time: ~~O(n^2)~~ -> スライスを毎回するのでO(n^3)
  - Space: O(n + m)

```python3
class Solution:
    def wordBreak(self, s: str, wordDict: list[str]) -> bool:
        words = set(wordDict)
        starts = [0]  # start index of unconfirmed section of s
        visited = set([0])
        while starts:
            start = starts.pop()
            for i in range(start + 1, len(s) + 1):
                if s[start:i] in words:
                    if i == len(s):
                        return True
                    if i in visited:
                        continue
                    starts.append(i)
                    visited.add(i)
        return False
```

- ここまで18分。
- 最初、スタックstartsにスライスした文字列直接を入れていたが、開始インデックスを入れるように書き直した。

## Step 2

- レビュー by GPT-5
  - > いくつかの定数倍最適化が可能です。
    > * 文字列スライスを避けて `s.startswith(word, start)` を使うとコピーを作らず比較できます。
    > * そもそも `i` を全文字まで進めず、`max_len` だけに制限する。
    > * wordDictの全文字種がs内の文字を網羅していない場合は早期にreturn Falseできます。
    - 2番目に関してはその通り。3番目はやってもいいくらい（網羅的な早期returnではないため）。1番目はwordDictをforループするときに便利そうだが、[実装1](#実装1)だとset(wordDict)を活用しづらい印象。
  - > DPが素直です。
    - らしい。最適部分構造を見つけられなかった。
    - i番目までを分割可能かどうかを、最後に使うべきword毎に検討するが、その時wordによってどの部分問題の結果を使うかが変わるのが今までにない。
    - つまり、i番目までを分割可能かに、可能性上でiより小さい全ての部分問題の結果を使い得る。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.56p9cz7xyy3h)
  - > この問題、まず正規表現で書くことができるので O(n) で解けるはずとまず初めに考えました。
    > ...
    > 単純なバックトラックでは失敗するというのが予想です。
    > あとは、正規表現を解釈するオートマトンがどうやってこういうのを解いているのかを考えて真似をすればいいでしょう。
    - 正規表現やオートマトンは聞きかじった程度なので、この感覚を共有できていない。あとで勉強します。
    - 文字列マッチングを知っていると、文字列sを逐次シフトしていくのが自然な発想なんだろうか。
  - > "a" * 51は、"a" * 2 と "a" * 4 で表せないので、単純なバックトラックでは失敗するというのが予想です。
    - バックトラック型の正規表現エンジンにおけるDoSパターン。
    - 正規表現インジェクション（ReDoS）：https://en.wikipedia.org/wiki/ReDoS
    - ユーザ入力を使った正規表現から生じるDOM based XSS：https://masatokinugawa.l0.cm/2018/01/regex-domxss.html
    - GPT-5に深掘りさせる：
      - > ReDoS を避けたい場面では「正規表現で表せる」ことと「正規表現で処理する」ことを切り分け、オートマトン系アルゴリズムか RE2 系エンジンを選ぶのが安全です。
        - > オートマトン：
          > - NFA: *Nondeterministic Finite Automaton*（非決定性有限オートマトン）
          >   - 同じ状態と入力で複数の遷移が許される。ε遷移も可。
          > - DFA: *Deterministic Finite Automaton*（決定性有限オートマトン）
          >   - 各状態・各入力記号に対して遷移は高々1つ。ε遷移はなし。
          > - NFA と DFA は表現力は同じで、NFA は subset construction により DFA に機械的に変換できる。受理判定の実行は DFA だと入力長 n に対して厳密に O(n)。
      - > Pythonで言うなら、
        > * ユーザー入力をパターンに入れる時は必ず `re.escape` を使う
        > * 可能なら RE2 系エンジンを使う（Python なら `re2` バインディング）
        > * サードパーティ製 `regex` を使い `timeout=` や atomic/possessive を活用
        >   * 例: `regex.match(pattern, text, timeout=0.01)`
        >   * `'(?>...)'` や `++` は `regex` モジュールのみ
        > * 入力長や試行回数に上限を設ける
        > * 正規表現で表せても、辞書照合や分割可能性判定は Trie、DP、Aho–Corasick に置き換える

### 実装2

- [実装1](#実装1)（DFS）の改善
- 計算量：
  - Time: O(n^3) -> O(n * L^2)に改善
  - Space: O(n + m)のまま

```python3
class Solution:
    def wordBreak(self, s: str, wordDict: list[str]) -> bool:
        if not s:
            return True
        if not wordDict:
            return False

        words = set(wordDict)
        max_length = max([len(word) for word in words])
        starts = [0]  # start index of unconfirmed section of s
        visited = set([0])
        while starts:
            start = starts.pop()
            end = min(start + max_length, len(s))
            for i in range(start + 1, end + 1):
                if s[start:i] in words:
                    if i == len(s):
                        return True
                    if i in visited:
                        continue
                    starts.append(i)
                    visited.add(i)
        return False
```

### 実装3

- bottom-up DP
- 計算量
  - Time: O(n * m)
  - Space: O(n + m)

```python3
class Solution:
    def wordBreak(self, s: str, wordDict: list[str]) -> bool:
        if not s:
            return True
        if not wordDict:
            return False
        
        can_segment = [True] + [False] * len(s)
        for i in range(1, len(s) + 1):
            for word in wordDict:
                if i < len(word) or not can_segment[i - len(word)]:
                    continue
                if s.startswith(word, i - len(word)):
                    can_segment[i] = True
                    break
        return can_segment[-1]
```

## Step 3

[実装3](#実装3)
