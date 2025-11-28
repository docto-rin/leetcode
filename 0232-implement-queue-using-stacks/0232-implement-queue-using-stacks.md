# 232. Implement Queue using Stacks

- URL: https://leetcode.com/problems/implement-queue-using-stacks/
- Difficulty: Easy
- Tags: Stack, Design, Queue
- Notebook: https://share.solve.it.com/d/d599b9df42461594a24807e846105577

## Step 1

### 実装1

- 問題の感想
  - 今まで当たり前のようにdequeなどをqueue（FIFO）として使ってきたが、確かにstackでどう実装するんだろう。
  - 機械語レベルでは全てが1つのstackにより記憶されている。
  - push to top, peek/pop from top, size, and is empty operations are valid. とのことで、インデックスアクセスも使えず難しい。
    - 特に、peek, popは明らかに工夫が必要。
- アルゴリズムの選択
  - peek, popが必要になったとき、stackの中身を別の容器に移し替えることを思いついた。
  - この移し替えは必要に迫られて、まとめて行うことで、均し計算量を軽くする作用もあるはず。（follow-up）
- 実装
  - 移し替えの操作は内部メソッドに分離する。
  - 属性も、直接外部に公開しないような意図で書く。
  - stack（容器）が2つ登場するが、お店の商品棚と倉庫に喩えた変数名にした。
- 計算量
  - Time
    | Operation | Worst Time | Amortized Time |
    |-----------|------------|----------------|
    | push      | O(1)       | O(1)           |
    | pop       | O(n)       | O(1)           |
    | peek      | O(n)       | O(1)           |
    | empty     | O(1)       | O(1)           |
  - Extra Space: O(n)

```python
# storage: [1, 2, 3, 4]
# shelf: []
# ↓ _stock & pop
# storage: []
# shelf: [4, 3, 2, 1] -> pop: 1
# ↓
# storage: []
# shelf: [4, 3, 2]
# ↓ push x2, pop x2
# storage: [5, 6]
# shelf: [4]

class MyQueue:

    def __init__(self):
        self._storage = []  # LIFO
        self._shelf = []  # FIFO
    
    def _stock(self):
        while self._storage:
            self._shelf.append(self._storage.pop())

    def push(self, x: int) -> None:
        self._storage.append(x)

    def pop(self) -> int:
        if not self._shelf:
            self._stock()
        return self._shelf.pop()

    def peek(self) -> int:
        if not self._shelf:
            self._stock()
        return self._shelf[-1]

    def empty(self) -> bool:
        return not self._shelf and not self._storage

# Your MyQueue object will be instantiated and called as such:
# obj = MyQueue()
# obj.push(x)
# param_2 = obj.pop()
# param_3 = obj.peek()
# param_4 = obj.empty()
```

- 思いつくのに苦戦し、20分くらいかかった。
- 振り返り
  - これであっているか不安。

## Step 2

**Q:** Step 1のコードをレビューして、改善点を教えてください。


**A:**
コードをレビューしますね！

**良い点：**
- 2つのスタックを使う基本アイデアは正しいです
- `_stock`で移し替えをまとめて行う設計により、償却計算量O(1)を達成しています
- `_storage`と`_shelf`という命名も役割が分かりやすい
- 計算量の分析も正確です

**改善点：**

1. **`peek`と`pop`の重複コード**
   ```python
   if not self._shelf:
       self._stock()
   ```
   この部分が2箇所にあります。`peek`を`pop`で実装するか、共通化を検討できます：
   ```python
   def peek(self) -> int:
       if not self._shelf:
           self._stock()
       return self._shelf[-1]

   def pop(self) -> int:
       self.peek()  # _shelfに値があることを保証
       return self._shelf.pop()
   ```

2. **軽微：コメントの意図**
   ```python
   self._storage = []  # LIFO
   self._shelf = []  # FIFO
   ```
   両方ともPythonのlistをスタック（LIFO）として使っているので、「FIFOとして見える順番で格納」という意味なら問題ないですが、少し紛らわしいかも。

**正しさの確認：**
ロジックは正しいです！例のトレースも合っています。`_shelf`が空のときだけ移し替えるので、各要素は最大2回しか移動しない→償却O(1)が成立します。

動作確認してみますか？簡単なテストケースを書いて試すと安心できると思います。

- LLMレビューの感想
  - 基本方針があっているとのことで安心した。
  - popとpeekで在庫確認のロジックが被っているのはその通りだと思った。peekに一元化するのもありだと思う。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.sarm16yxqaz)
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1232739063891492895
    - > お、ちょっと私の期待とは違うアルゴリズムでした。これは関数型言語のデータ構造という背景がある問題に見えます。
      > - https://www.slideshare.net/hamukazu/ss-10114639
      > - https://www.cs.cmu.edu/~rwh/students/okasaki.pdf
    - 分かりやすく面白い資料だった。
    - 「私の期待」というのは、補充でstackを反転させる操作に遅延評価を使うことで、均さなくてもO(1)を実現できることだと推測した。
      - LLMに聞いたところ、遅延評価版の厳密な実装はPythonだと難しく、Haskellなどの純粋関数型言語が必要とのこと。
- PR
  - https://github.com/colorbox/leetcode/pull/15
  - https://github.com/ryosuketc/leetcode_grind75/pull/13
  - https://github.com/huyfififi/coding-challenges/pull/13
  - https://github.com/rihib/leetcode/pull/32

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
4つのPRを確認しますね。

4つのPRの内容をまとめますね。

---

**1. colorbox/leetcode#15** (C++)
- **実装**: 最初はpush時に毎回移し替えを行う非効率な実装だった
- **レビュー指摘**: nodchipさんから「pushは`push_stack`に入れ、pop/peek時に`pop_stack`が空の場合のみ移し替えるとamortized O(1)になる」と改善を提案された
- **学び**: push時に移し替えは不要。関数型言語のデータ構造（Okasaki）に由来する問題との紹介あり

---

**2. ryosuketc/leetcode_grind75#13** (C++)
- **実装**: `stack_in` / `stack_out` という分かりやすい命名。amortized O(1)版も実装
- **レビュー指摘**:
  - `aux`という略称についての議論（補助関数の意味で使われる）
  - `MyQueue() = default` の提案
  - `bool empty() const noexcept` の指摘
  - peek時に例外が飛ぶ可能性のキャッチについて
  - Google Style Guideの関数名規約について
- **学び**: Purely Functional Data Structures (Okasaki) の BankersQueue が言及された

---

**3. huyfififi/coding-challenges#13** (Python)
- **実装**: Step1で2パターン（毎回移し替え版 / 遅延移し替え版）を試した
- **レビュー指摘**:
  - FILOよりLIFOが一般的
  - 変数名の補足コメントがあると良い
  - `pop()`と`peek()`の共通化について議論（著者は「視線が移動するので書き下す方が好み」との意見）
  - 空コレクションにtype hintを入れる提案
- **学び**: Banker's Method（均し計算量の分析手法）についてのメモあり。「重そうな処理を毎回発生させず、その状況のまま利用する方法を探る」という教訓

---

**4. rihib/leetcode#32** (Go)
- **実装**: `pushStack` / `popStack` を使った標準的な実装
- **PR説明**: Goの構造体初期化について詳しく解説（`MyQueue{}` vs `new(MyQueue)` vs `var q MyQueue`）
- **レビュー**: colorboxさん、odaさんから細かい指摘あり（詳細はShow resolved）
- **特徴**: 言語特有の初期化パターンの学習メモが充実

---

**共通の学び**:
- 2つのスタックを使い、**pop/peek時にのみ必要に応じて移し替える**のが最適解
- Chris Okasaki の "Purely Functional Data Structures" が元ネタとして複数回言及
- 変数名は `in/out`, `push/pop`, `input/output` などが採用されている

### 実装2

- リファクタ内容
  - self._shelfの空チェック＆補充ロジックをpeek()に集約
  - 属性定義時にコメントでLIFO, FIFOと書いていたのは紛らわしいので変更。
  - voidメソッドに対し、type hint "-> None"を明示的に書く。
    - PEP484: https://peps.python.org/pep-0484/#using-none

```python
class MyQueue:

    def __init__(self):
        self._storage = []
        self._shelf = []  # reversed
    
    def _stock(self) -> None:
        while self._storage:
            self._shelf.append(self._storage.pop())

    def push(self, x: int) -> None:
        self._storage.append(x)

    def pop(self) -> int:
        self.peek()
        return self._shelf.pop()

    def peek(self) -> int:
        if not self._shelf:
            self._stock()
        return self._shelf[-1]

    def empty(self) -> bool:
        return not self._shelf and not self._storage
```

## Step 3

[実装2](#実装2)

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装3

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```