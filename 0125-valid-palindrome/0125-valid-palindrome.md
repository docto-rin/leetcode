# 125. Valid Palindrome

- URL: https://leetcode.com/problems/valid-palindrome/
- Difficulty: Easy
- Tags: Two Pointers, String
- Notebook: https://share.solve.it.com/d/e80e6c41b8c79d6a8e2915069f91e325

## 問題文

<p>A phrase is a <strong>palindrome</strong> if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Alphanumeric characters include letters and numbers.</p>

<p>Given a string <code>s</code>, return <code>true</code><em> if it is a <strong>palindrome</strong>, or </em><code>false</code><em> otherwise</em>.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre>
<strong>Input:</strong> s = &quot;A man, a plan, a canal: Panama&quot;
<strong>Output:</strong> true
<strong>Explanation:</strong> &quot;amanaplanacanalpanama&quot; is a palindrome.
</pre>

<p><strong class="example">Example 2:</strong></p>

<pre>
<strong>Input:</strong> s = &quot;race a car&quot;
<strong>Output:</strong> false
<strong>Explanation:</strong> &quot;raceacar&quot; is not a palindrome.
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre>
<strong>Input:</strong> s = &quot; &quot;
<strong>Output:</strong> true
<strong>Explanation:</strong> s is an empty string &quot;&quot; after removing non-alphanumeric characters.
Since an empty string reads the same forward and backward, it is a palindrome.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= s.length &lt;= 2 * 10<sup>5</sup></code></li>
	<li><code>s</code> consists only of printable ASCII characters.</li>
</ul>


## Step 1

### 実装1

- アルゴリズムの選択
  - 大まかには入力をパース＋正規化し、両端から整合を見ていく方針。
  - パース＋正規化した中間結果を変数としておくとExtra Spaceを食うが、その方がシンプルコードになりそう。
    - 一方で、パース＋正規化を逐次的に行うことでExtra Space: O(1)で書くこともできる。
    - 関数型っぽい書き方をうまく実装すればシンプルさを維持できるかもしれないが、思いつかない。
    - map、filterなどを使うのだろうか。
  - 一旦は前者の中間結果を変数として置く書き方で書いてみる。
- 実装
  - reモジュールで必要な文字をマッチする方法が思いついた。
  - その後、alphanumericな文字に対し、逐次的にstr.lower()を使う。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
import re


class Solution:
    def isPalindrome(self, s: str) -> bool:
        alphanumerics = re.findall(r"[a-zA-Z0-9]", s)
        normalized = [ch.lower() for ch in alphanumerics]

        # read bidirectionally
        left = 0
        right = len(normalized) - 1
        while left < right:
            if normalized[left] != normalized[right]:
                return False
            left += 1
            right -= 1
        return True
```

- 10分ほどかかった。
- 振り返り
  - ch.lower()は最初に全体まとめてやった方が速そう。
    ```py
    normalized = re.findall(r"[a-zA-Z0-9]", s.lower())
    ```
    - 今の実装は明示的に内包表記でループしているのでstr.lower()のネイティブコードの恩恵を受けられない。
  - 次の実装はExtra Space: O(1)で、なるべくシンプルに書きたい。
  - re.findall()以外の方法もありそう。str型のメソッドやstringモジュールなど、探せば見つかりそう。

### 実装2

- アルゴリズムと実装
  - Extra Space: O(1)で書いてみる。
  - reモジュールに頼らず書いてみる。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        def skip_non_alphanumeric(index, step):
            while 0 <= index < len(s):
                if s[index].isalpha() or s[index].isdecimal():
                    break
                index += step
            return index
        
        # read bidirectionally
        left = skip_non_alphanumeric(0, 1)
        right = skip_non_alphanumeric(len(s) - 1, -1)
        while left < right:
            if s[left].lower() != s[right].lower():
                return False
            left = skip_non_alphanumeric(left + 1, 1)
            right = skip_non_alphanumeric(right - 1, -1)
        return True
```

- 振り返り
  - .isalpha() or .isdecimal() のところは、一応ASCII以外が来てもある程度厳密にチェックできるように.isalnum()を避けてみた。
  - ただそれをいうなら本当に厳密なのはre.match(r"[a-zA-Z0-9]", s[index])な気がする。

## Step 2

**Q:** 実装1、2のコードをレビューして、改善点を教えてください。


**A:**
両方の実装をレビューしますね！

**実装1について**

良い点：
- シンプルで読みやすい
- 正規表現の使い方は正しい

改善点：
1. 振り返りで書かれている通り、`s.lower()`を先にやる方が効率的
2. 両端比較は`reversed`やスライスで簡潔に書ける：
   ```python
   return normalized == normalized[::-1]
   ```
3. `str.isalnum()`で十分では？（後述）

**実装2について**

良い点：
- O(1) Extra Spaceを達成
- ヘルパー関数で意図が明確

改善点：
1. `isalpha() or isdecimal()` → `isalnum()`で良いと思います
   - 問題文は "alphanumeric" としか言っておらず、ASCII範囲外の文字がテストケースに含まれる可能性は低い
   - 仮にあっても、Unicodeのalphanumericを受け入れる方が自然な解釈とも言える
2. ヘルパー関数名：`skip_non_alphanumeric`は動作を表しているが、`find_next_alnum`のような「何を返すか」を示す名前も検討の余地あり
3. ループ内で毎回`len(s)`を呼んでいる（微小だがキャッシュ可能）

**その他の書き方案**

実装1の簡潔版：
```python
def isPalindrome(self, s: str) -> bool:
    cleaned = [c for c in s.lower() if c.isalnum()]
    return cleaned == cleaned[::-1]
```

実装2のジェネレータ版（関数型っぽく）：
```python
def isPalindrome(self, s: str) -> bool:
    def chars():
        for c in s.lower():
            if c.isalnum():
                yield c
    forward = list(chars())
    return forward == forward[::-1]
```

ただし、O(1) Extra Spaceを維持しつつ関数型っぽく書くのは難しいですね。実装2のアプローチが素直だと思います。

何か気になる点はありますか？

- LLMレビューの感想
  - return normalized == normalized[::-1]は簡潔だがスライスで重くなるのでなんとも言えない。
  - まぁ確かにstr.isalnum()で十分かもしれない。厳密にアラビア数字だけにしたいなら正規表現でいいので。
  - ヘルパー関数名、確かにind_next_alnumの方が直感的で情報量も多い気がする。
  - > O(1) Extra Spaceを維持しつつ関数型っぽく書くのは難しい
    - なるほど。コメント集などもみて適宜検討。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.idizyy9b2j62)
  - https://discord.com/channels/1084280443945353267/1192728121644945439/1218815342206386176
    - > 本質的にはそれほど変わりません。...これ、3つのループにもできますし、1つのループにもできます。
    - その通りで、いろいろ脳内で試して可読性、速度、メモリのトレードオフを見定める感覚。
  - https://discord.com/channels/1084280443945353267/1253694251271852095/1281522459753058357
    - > Go 言語っぽい書き方とか探してみませんか
    - Go言語のgoroutineとchannelを使って、並列処理で書く。
        1. goroutine (`go func()`) = 軽量スレッドのようなもの。`go filter(...)`で別の処理が並列に走り出す
        2. channel (`chan rune`) = スレッド間でデータをやり取りするパイプ。Pythonの`Queue`に近い
            - `c <- value` : channelに値を送る
            - `value := <- c` : channelから値を受け取る（届くまで待つ）
        3. 左から読むgoroutineと右から読むgoroutineが同時に動き、それぞれがchannelに1文字ずつ送る。メインのループで両方から1文字ずつ受け取って比較する
- PR
  - https://github.com/Kitaken0107/GrindEasy/pull/8
  - https://github.com/rihib/leetcode/pull/25
  - https://github.com/ryosuketc/leetcode_grind75/pull/5
  - https://github.com/huyfififi/coding-challenges/pull/5
  - https://github.com/kzhra/Grind41/pull/5
  - https://github.com/t-ooka/leetcode/pull/6

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの学習記録とそれについているコメントをあなたがチェックし、PRごとの内容を簡潔にまとめて欲しいです。


**A:**
各PRの内容を確認しますね。

各PRの内容を確認しました。以下にまとめます：

---

**1. Kitaken0107/GrindEasy#8** (Python)

主なコメント：
- 数字も回文判定に含めるべきだが考慮されていない指摘
- `left`/`right`ポインタで`while left < right`とする書き方の方が分かりやすい
- 長さ1以下の場合のif文は不要

---

**2. rihib/leetcode#25** (Go)

主なコメント：
- goroutineとchannelを使った並列処理の実装にラベル付け
- 具体的なコードへのコメントは少なめ

---

**3. ryosuketc/leetcode_grind75#5** (C++)

主なコメント（非常に充実）：
- C++では文字列はmutable、Pythonの`+=`での文字列コピーとは異なる
- 関数の引数は`const string&`か`string_view`で渡すべき（コピー回避）
- 戻り値のコピーを避けるには引数に参照を渡す方法もある（ただしNRVOで最適化される可能性あり）
- `reserve`で事前にcapacityを確保する選択肢
- メモリコピーの実行時間見積もり（1 clock で4-8バイト程度）
- Google Style Guideでは`public`を先に宣言
- Rope、Piece Tableなどのデータ構造の紹介

---

**4. huyfififi/coding-challenges#5** (Python)

主なコメント：
- `str.isalnum()`のドキュメントを確認すべき
- `isalnum()`は"四"や"²"などUnicodeの数字も`True`を返すため、問題の意図と異なる可能性がある → 面接では確認が必要

---

**5. kzhra/Grind41#5** (C++)

主なコメント：
- `else`を省いてインデントを浅くする
- `==`のときはループを抜けて良い
- Google Style Guide準拠の空白の入れ方（キーワード後、`{`前、`:`前後など）
- `std::tolower()`と`isalnum()`を組み合わせてif文を簡潔に
- `continue`の一貫性、不要な`continue`は削除
- early returnで条件を反転した方が読みやすい
- 変数名は`left`/`right`の方が直感的

---

**6. t-ooka/leetcode#6** (Python/C++)

主なコメント：
- `i`/`j`より`left`/`right`の方が分かりやすい
- `!= True`より`not X`の方が自然
- whileが連続すると読みにくい → `if not ... : continue`パターンの提案
- PEP8ではwhileと同じ行に実行文を置くことは非推奨

---

**共通して多かった指摘：**
1. 変数名は`left`/`right`が直感的
2. early returnでインデントを浅く
3. `isalnum()`の挙動（Unicode対応）に注意
4. 不要な`continue`や`else`は省略

何か気になるPRがあれば、詳しく見ていきましょうか？

### 実装3

- リファクタ内容（[実装2](#実装2)をベースに改良）
  - isalpha() or isdecimal()の代わりに isalnum() を使う。
  - 二重ループだったのを一重ループに変更。
    - 手順の都合上、初期化が[実装2](#実装2)よりシンプルになった。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        left = 0
        right = len(s) - 1
        while left < right:
            if not s[left].isalnum():
                left += 1
                continue
            
            if not s[right].isalnum():
                right -= 1
                continue
            
            if s[left].lower() != s[right].lower():
                return False
            left += 1
            right -= 1
        
        return True
```

## Step 3

### 実装4

- 何も見ずに書き直し
- 可読性の高さから[実装1](#実装1)ベースの実装を選択。
- alphanumericの判定には正規表現が必要十分で最適と判断。

```python
import re


class Solution:
    def isPalindrome(self, s: str) -> bool:
        normalized = re.findall(r"[a-zA-Z0-9]", s.lower())
        left = 0
        right = len(normalized) - 1
        while left < right:
            if normalized[left] != normalized[right]:
                return False
            left += 1
            right -= 1
        return True
```

## Step 4

自分のPR：（自分のPRへのリンク）

**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装4

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```