## Step 1

- 問題文
  - 有効なメールアドレスのリストemailsを受け取り、実質的に異なるメールアドレスの数を返す問題。
  - 有効なメールアドレスemails[i]は {local_name}@{domain_name} というフォーマットである。
  - local_name内に"."が含まれる場合、"."を除いたアドレスにメールは送信される。
  - local_name内に"+"が含まれる場合、最初の"+"より後ろは無視される。
- 入力の制約
  - emails[i]はただ1つの@を持ち、そのほか英語、"+"、"."からなる。
  - local_nameもdomain_nameも空にはならない。
  - local_nameは"+"で始まらない。
  - domain_nameは常に.com接尾辞で終わり、@と.comの間に最低でも1文字を持つ。
- アルゴリズムの選択
  - 大枠は、正規化したメールアドレスをもとにした固有なキーをsetを管理し、そのサイズを返す方法しかないと思う。
  - メールアドレスの正規化（またはパース）はやり方は複数考えられる。ここでは2つ考えた。
    - [実装1](#実装1): emails[i]をstr型の組み込みメソッド（split, replace）を使って正規化し、(local_name, domain_name)のタプルでキーとする。
    - [実装2](#実装2): emails[i]をイテラブルみなし、forループで1文字ずつ前から処理していく方法。
      - メールアドレスは構文解析的には文字単位のLL(1)で解析可能。（最近コンパイラを勉強した。）
      - それでも組み込みメソッドを使うより実装は複雑になり、かつ遅そう。
- 実装の方針
  - 実用的な問題設定なので、最低限異常な入力を考慮したいところ。
    - @が0個 or 2個以上の場合はValueErrorを送出する。
    - 実用を考えればもっと凝るべきだが、一旦他は省略。

### 実装1

- emails[i]をstr型の組み込みメソッドを使って正規化し、(local_name, domain_name)のタプルでキーとする。
- n = emails.length, l = emails[i].length として、
- 時間計算量: O(n*l)
- 空間計算量: O(n*l)

この問題は計算量オーダー自体はさほど重要ではなさそう。

```python3
from typing import List


class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize_address(email):
            atmark_split = email.split("@")
            if len(atmark_split) != 2:
                raise ValueError("Include exactly one @ in your address")
            local_name = atmark_split[0].split("+")[0].replace(".", "")
            domain_name = atmark_split[1]
            return local_name, domain_name
        
        different_addresses = set()
        for email in emails:
            different_addresses.add(normalize_address(email))
        return len(different_addresses)
```

- Runtime: 3 ms
- Memroy: 18 MB

せっかくなら、normalize_address関数の返り値を、
```python3
return local_name + "@" + domain_name
```
にしてもいいかもしれない。

### 実装2

- emails[i]をイテラブルみなし、forループで1文字ずつ前から処理していく方法。
- 時間計算量: O(n*l)
- 空間計算量: O(n*l)

```python3
from typing import List
from itertools import islice


class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize_address(email):
            between_plus_and_atmark = False
            before_atmark = True
            stored = []
            for character in email:
                if before_atmark and character == ".":
                    continue
                if before_atmark and character == "+":
                    between_plus_and_atmark = True
                    continue
                if character == "@":
                    yield tuple(stored)
                    stored = []
                    between_plus_and_atmark = False
                    before_atmark = False
                    continue
                if not between_plus_and_atmark:
                    stored.append(character)
            yield tuple(stored)
        
        different_addresses = set()
        for email in emails:
            different_addresses.add(tuple(islice(normalize_address(email), 2)))
        return len(different_addresses)
```

- @の個数に関する異常検知を含められなかった。
- generatorを使う意味はあまりないが、条件分岐の削減のため。かえってわかりにくいか？
- Runtime: 27 ms
- Memroy: 18.2 MB

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.h73uwlfq793n)
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1207996784211918899
    > この方法、ステートマシンとかいいますね。
    - [実装2](#実装2)
  - https://github.com/hayashi-ay/leetcode/pull/25/
    - 正規表現を書いて要素ごとのキャプチャーも確かにありえた。発想できなかった。
    - 本質的には[実装1](#実装1)と同じはず。
    - パターンを書き慣れておらず、手を出しにくい。
      - 普段コードを正規表現で修正するときはパターンをLLMに生成させているため...
  - コピーの仕方（破壊的か否か）
  - ユースケース: https://discord.com/channels/1084280443945353267/1251052599294296114/1254245440690589787
  - https://discord.com/channels/1084280443945353267/1307605446538039337/1333100090054934628
    > 内包表記も可能ですがループを回す方が見やすいかもしれませんね。

### 実装3

- reモジュールによる方法
  - https://docs.python.org/ja/3.13/library/re.html
  - 以前にも何回かみたことあるが、どうしても自力でパターンをかけるほどシンタックスを覚えられない。
- パターンマッチ用メソッドはどれを使えばいい？
  - > [search() vs. match()](https://docs.python.org/ja/3.13/library/re.html#search-vs-match)
    > 
    > Python offers different primitive operations based on regular expressions:
    > - re.match() checks for a match only at the beginning of the string
    > - re.search() checks for a match anywhere in the string (this is what Perl does by default)
    > - re.fullmatch() checks for entire string to be a match
  - re.fullmatch(pattern, string)を使う。
  - ()でキャプチャしたものをmatch objectから取り出せる。
- 置換はどうやる？
  - https://docs.python.org/ja/3.13/library/re.html#re.sub
  - re.sub(pattern, repl, string, count=0, flags=0)
    | 引数        | 意味                    |
    | --------- | --------------------- |
    | `pattern` | 置換対象を指定する正規表現         |
    | `repl`    | 置換後の文字列（または関数）        |
    | `string`  | 処理する入力文字列             |
    | `count`   | 置換回数の上限（デフォルト0は「すべて」） |
    | `flags`   | オプション（大文字小文字無視など）     |
  - re.subは、re.findall + 置換
  - re.findallは、re.searchの全検索版

```python3
from typing import List
import re


class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize_address(email):
            match_object = re.fullmatch(r'([^@+][^@]*)@([^@]+\.com)', email)
            if not match_object:
                raise ValueError("Invalid email address")
            local_name, domain_name = match_object.groups()
            local_name = re.sub(r'\+.*', '', local_name)
            local_name = re.sub(r'\.', '', local_name)
            return f"{local_name}@{domain_name}"
        
        return len({normalize_address(email) for email in emails})
```

- 入力制約をなるべく盛り込み、それにマッチしないものはValueErrorとした。
  - "@"の個数が1個でない。
  - local_nameが"+"から始まる。
  - domain_nameが".com"で終わらない。
- こうした細かい制約を、コードの行数を増やさず簡潔に実装できるのが良さに感じた。

## Step 3

- [実装1](#実装1)の方法を書く。
- 例外処理を丁寧に書いてみる
  - https://google.github.io/styleguide/pyguide.html#24-exceptions
  - 以前のレビューでも指摘されたことがあるが、f-stringで変数を含めると良さそう。

```python3
from typing import List


class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize_address(email: str) -> str:
            atmark_split = email.split("@")
            if len(atmark_split) != 2:
                raise ValueError(f"Email address '{email}' must include exactly one '@'")
            local_name, domain_name = atmark_split
            if local_name.startswith("+"):
                raise ValueError(f"Local name '{local_name}' must not start with '+'")
            if not domain_name.endswith(".com"):
                raise ValueError(f"Domain name '{domain_name}' must end with '.com'")
            local_name = local_name.split("+", maxsplit=1)[0]
            local_name = local_name.replace(".", "")
            return f"{local_name}@{domain_name}"
        
        return len({normalize_address(email) for email in emails})
```

使ってみると、

```
>>> solution.numUniqueEmails(["example@example.jp"])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 16, in numUniqueEmails
  File "<stdin>", line 16, in <setcomp>
  File "<stdin>", line 11, in normalize_address
ValueError: Domain name 'example.jp' must end with '.com'
```

いいんじゃないでしょうか。
