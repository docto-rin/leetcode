# 242. Valid Anagram

- URL: https://leetcode.com/problems/valid-anagram/
- Difficulty: Easy
- Tags: Hash Table, String, Sorting
- Notebook: https://share.solve.it.com/d/31dccb7f2a9aa3998533a6acccf04c65

## Step 1

### 実装1

- アルゴリズムの選択
  - 文字種ごとの登場回数をs, tに対し数え、同じ分布か調べる。
- 実装
  - lowercase English lettersなので、長さ26の配列でカウントする。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python
import string


class Solution:
    NUM_CHARS = len(string.ascii_lowercase)
    BASE_ORD = ord("a")

    def count_lowers(self, s: str) -> list[int]:
        counter = [0] * self.NUM_CHARS
        for ch in s:
            counter[ord(ch) - self.BASE_ORD] += 1
        return counter

    def isAnagram(self, s: str, t: str) -> bool:
        return self.count_lowers(s) == self.count_lowers(t)
        
```

### 実装2

- > Follow up: What if the inputs contain Unicode characters? How would you adapt your solution to such a case?
  - こうなったら辞書を使う方が可読性・保守性で好ましい。
  - せっかくなのでdefaultdictやCounterは使わずに実装する。
- 計算量
  - 文字の種類数をmとして、
  - Time: O(n)
  - Extra Space: O(m)

```python
class Solution:
    def count_chars(self, s: str) -> dict[str, int]:
        counter = {}
        for ch in s:
            if ch not in counter:
                counter[ch] = 0
            counter[ch] += 1
        return counter

    def isAnagram(self, s: str, t: str) -> bool:
        return self.count_chars(s) == self.count_chars(t)
```

### 実装3

- early returnを導入できないか考え、作ってみた。
- 文字数・文字種が多く、anagramが来ることがレアな状況では輝きそう。
- 通常の用途ではこの可読性の悪化は割に合わないが、他言語で低レイヤーで実装するならこのくらいしても良いだろう。

- 振り返り
  - まぁひとまず想像力の範囲ではこんな感じ。

## Step 2

**Q:** Step 1のコードをレビューして、改善点を教えてください。



**A:**
3つの実装、どれもよく考えられていますね！いくつかコメントします。

**実装1について**

良い点：
- 固定長配列を使うのは、この制約下では最適解
- クラス変数で定数を定義しているのは良い習慣

改善案：
- `len(s) != len(t)` の早期リターンを入れると、長さが違う場合にカウント処理をスキップできる

```python
def isAnagram(self, s: str, t: str) -> bool:
    if len(s) != len(t):
        return False
    return self.count_lowers(s) == self.count_lowers(t)
```

**実装2について**

良い点：
- Unicode対応への拡張として適切な選択
- `Counter`を使わず自前実装する練習、良いと思います

細かい点：
- `counter[ch] = counter.get(ch, 0) + 1` と1行で書くスタイルもある（好みの問題）

**実装3について**

良い点：
- early returnの発想は実践的
- 実際に「ほとんど不一致」なケースでは効果的

気になる点：
- `remaining_counts[ch] -= 1` で値が0になっても辞書に残る。問題はないが、メモリ的には`del`する選択肢もある（ただし可読性との兼ね合い）

---

全体として、問題の制約をよく理解した上で複数のアプローチを試しているのが素晴らしいです。何か特定の実装について深掘りしたい点はありますか？

- LLMレビューの感想
  - `counter[ch] = counter.get(ch, 0) + 1` も悪くないが、こう書くくらいならdefaultdictに変更して+= 1を維持したい。
- コメント集：空
- PR
  - https://github.com/ryosuketc/leetcode_grind75/pull/7
  - https://github.com/eito2002/LeetCode/pull/2
  - https://github.com/huyfififi/coding-challenges/pull/7
  - https://github.com/azriel1rf/leetcode-prep/pull/2
  - https://github.com/rihib/leetcode/pull/5
  - https://github.com/kzhra/Grind41/pull/8
  - https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/21

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
7つのPRを確認しますね。順番に読み込んでいきます。

各PRの内容をまとめます。

---

**1. ryosuketc/leetcode_grind75 PR #7** (C++)
- **解法**: `unordered_map`でカウント
- **主なコメント**:
  - Structured binding (`for (const auto& [s_char, s_char_count] : ...)`) はC++17から（oda）
  - ソート後に `s == t` で比較する方法もある（Apo-Matchbox）
  - 参照渡し・RVO・ポインタに関する深い考察をメモに記載

---

**2. eito2002/LeetCode PR #2** (C++)
- **解法**: hash map、ソート、固定長配列（26要素）
- **主なコメント**:
  - ぶら下がりif（`if (...) return false;`）は事故が起きやすいので避けるべき（huyfififi）
  - 変数名を `dict` → `char_to_count` のように説明的にすべき（huyfififi）
  - `++i` と `i++` の違い（前置の方がコピーを避けられる）
  - EBCDICでは `s[i] - 'a'` が動かない可能性（nodchip）
  - CodeRabbit（AI）のレビューはこの勉強会では不要という議論

---

**3. huyfififi/coding-challenges PR #7** (Python)
- **解法**: 配列カウント、`defaultdict`、`collections.Counter`
- **主なコメント**:
  - 同じ処理を2度したら関数にすることを考える（oda）
  - 引き算で解くとSpaceがn+mではなくnかmで済む（sasanquaneuf）
  - 早期リターンできるパターンに気を配る
  - `cnt` より `count` と省略しない方が良い（Google Style Guide）
  - Step3でも同じコードでも貼った方が良い

---

**4. azriel1rf/leetcode-prep PR #2** (Python)
- **解法**: `defaultdict`でカウント、`Counter`
- **主なコメント**:
  - 変数名は `lower_snake` が標準（PEP8, Google Style Guide）
  - `cnt` ではなく `count` と略さず書く
  - `return cnt1 == cnt2` で辞書比較可能（内部でearly returnされる）
  - `Counter(s) == Counter(t)` で1行で書ける
  - 引き算して負になったらFalseで抜ける方法も（liquo-rice）
  - 1ループで2つの文字列を処理するより、2ループに分けた方が良い（oda）

---

**5. rihib/leetcode PR #5** (Go)
- **解法**: `map[rune]int`でカウント（Unicode対応）
- **主なコメント**:
  - **Unicode/サロゲートペア/結合文字/合字** についての非常に詳細な調査（本人）
  - `rune`を使っても結合文字（例：「が」=「か」+濁点）は正しく処理できない（sasanquaneuf）
  - グラフィムクラスタを使えばほぼ対応可能だが、旗シーケンスなど例外あり
  - 文字コードは魔境（oda）

---

**6. kzhra/Grind41 PR #8** (C++)
- **解法**: `map` / `unordered_map`でカウント、ソート
- **主なコメント**:
  - `return count_letter_1 == count_letter_2;` で比較可能
  - 変数名は `letter_to_count` のようにkey→valueを表す命名が良い（nodchip）
  - `unordered_map` vs `map`：要素数が少ないとハッシュ化のオーバーヘッドで差がないことも
  - `for (auto [letter, count] : ...)` で回した方が読みやすい
  - `if` の後にスペースを入れる（Google Style Guide）

---

**7. NobukiFukui/Grind75-ProgrammingTraining PR #21** (Python)
- **解法**: `sorted()` で比較、`set` + `count()`
- **主なコメント**:
  - `count()`や`sorted()`を使わずに書けるか？という問いかけ（Kitaken0107）
  - 基本的な実装力を問うレビュー

---

**全体的な傾向**:
- **変数名**: `char_to_count`、`letter_to_count` など説明的な命名が推奨
- **比較**: 辞書/mapは `==` で直接比較可能
- **早期リターン**: 長さチェックや負のカウントで早く抜ける
- **Unicode**: 結合文字やサロゲートペアの問題は深い（特にrihib PR）
- **コーディング規約**: PEP8、Google Style Guideへの言及多数

**Q:** 結局実装的には、unicodedata.normalize('NFC', text)すれば、多くの合体文字が1字になり、あとはオプションでgraphemesって感じ？


**A:**
はい、その理解で正確です！整理すると：

**NFC正規化で解決するもの**:
- 結合文字の統一: `か` + `゙` → `が`
- アクセント記号: `e` + `́` → `é`
- 同じ見た目の異なる表現を統一

**NFC正規化では解決しないもの（grapheme必要）**:
- 国旗: `🇯🇵` = Regional Indicator J + P（2コードポイント、合成されない）
- ZWJ絵文字: `👨‍👩‍👧` = 複数絵文字 + ZWJ（結合しても1コードポイントにならない）
- 肌色修飾: `👋🏽` = 👋 + 🏽（修飾子は独立したコードポイント）

```python
import unicodedata

# NFCで解決 ✓
s = "か\u3099"  # か + 濁点
unicodedata.normalize('NFC', s)  # → "が" (1文字)

# NFCでは解決しない ✗
flag = "🇯🇵"
unicodedata.normalize('NFC', flag)  # → "🇯🇵" (まだ2コードポイント)
len(unicodedata.normalize('NFC', flag))  # → 2
```

**結論**:
| ケース | NFC | grapheme |
|--------|-----|----------|
| ASCII | 不要 | 不要 |
| 日本語・アクセント | ✓ | 不要 |
| 絵文字・国旗 | 不十分 | ✓ |

実用上は「NFC正規化だけで95%のケースは対応できる」という感じですね。絵文字をユーザー入力で扱うSNS系サービスなどでは`grapheme`まで必要になります。

### 実装4

- Unicode対応をしっかりやる。
  - とはいえ、先人のやってくれた実装を使うだけ。

```python
from typing import Iterator
import unicodedata
from collections import Counter

import grapheme


class Solution:
    def iter_graphemes(self, s: str) -> Iterator[str]:
        normalized = unicodedata.normalize('NFC', s)
        return grapheme.graphemes(normalized)

    def isAnagram(self, s: str, t: str) -> bool:
        s_graphemes = self.iter_graphemes(s)
        t_graphemes = self.iter_graphemes(t)
        return Counter(s_graphemes) == Counter(t_graphemes)
        
```

## Step 3

### 実装5

- 何も見ずに書き直し
  - Unicodeの設定は忘れ、素朴な設定に取り組む。

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False
        
        s_char_to_count = {}
        for ch in s:
            if ch not in s_char_to_count:
                s_char_to_count[ch] = 0 
            s_char_to_count[ch] += 1
        
        remaining_count = s_char_to_count
        for ch in t:
            if remaining_count.get(ch, 0) == 0:
                return False
            remaining_count[ch] -= 1
        return True
```

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装6

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```