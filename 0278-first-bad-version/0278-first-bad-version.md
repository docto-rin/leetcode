# 278. First Bad Version

- URL: https://leetcode.com/problems/first-bad-version/
- Difficulty: Easy
- Tags: Binary Search, Interactive
- Notebook: https://share.solve.it.com/d/e2139310cbf63468f316a82d8ead06cb

## Step 1

### 実装1

- アルゴリズムの選択
  - [1, 2, ..., n] --(API)--> [False, ..., False, True, ..., True]となる。
  - First Trueを見つける方法だが、単調述語なので二分探索（lower_bound, bisect_left）でいける。
- 実装
  - range(1, n + 1)に対し、key=isBadversionで単調述語を作れる。
  - 返り値がインデックスであり、1-indexedなversionsより1小さいことに注意。
  - すべてのversionがfineなときは分かりやすく-1を返すことにする。（未来の次のバージョンを返すのは違和感がある。）
- 計算量
  - Time: O(log n)
  - Extra Space: O(1)

```python
import bisect


# The isBadVersion API is already defined for you.
# def isBadVersion(version: int) -> bool:


class Solution:
    def firstBadVersion(self, n: int) -> int:
        first_bad_index = bisect.bisect_left(
            range(1, n + 1), True, key=isBadVersion
            )
        first_bad_version = first_bad_index + 1
        if first_bad_version == n + 1:
            # all versions are fine
            return -1
        return first_bad_version
        
```

- 振り返り
  - 特にない。

## Step 2

- LLMレビューの感想
  - 久しぶりにbisectに頼らず書いてみてもいいかも。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.48l4rpfkee05)
  - https://discord.com/channels/1084280443945353267/1351704260793995355/1370664386246606899
    - > left, right もよく使われるのですが、本当はいい名前ではあまりないです。二分探索をする仕事をシフト制でやるとして、職場に行ってみたら「左が100で右が200」とだけ書いてある業務日誌があったら昨日働いていた人に電話しませんか。「左と右ってどういう意味なのか。」
      - docstringにループ不変条件を書いておくと良いかも

### 実装2

- 実装の検討
  - bisectを使わずに書く。
  - 不変条件の設定の仕方は等号を含めるかどうかで2x2=4パターンあるが、そのうち2つ書いた。
  - 1つ目に書いた方は4パターンの中で最もよくみる気がする。
  - 2つ目に書いた方は、4パターンの中で最も不人気と思われる。
    - 4パターンの中で唯一 mid = (left + right + 1) // 2 の+1が必要だったりする。

```python
# The isBadVersion API is already defined for you.
# def isBadVersion(version: int) -> bool:


class Solution:
    def firstBadVersion(self, n: int) -> int:
        """Invariant:
        - for all i < left: isBadVersion(i) is False
        - for all i > right: isBadVersion(i) is True
        """
        left = 1
        right = n
        while left <= right:
            mid = (left + right) // 2
            if isBadVersion(mid):
                right = mid - 1
            else:
                left = mid + 1
        # right == left - 1
        if left == n + 1:
            return -1
        return left


class Solution:
    def firstBadVersion(self, n: int) -> int:
        """Invariant:
        - for all i <= left: isBadVersion(i) is False
        - for all i > right: isBadVersion(i) is True
        """
        left = 0
        right = n
        while left < right:
            mid = (left + right + 1) // 2
            if isBadVersion(mid):
                right = mid - 1
            else:
                left = mid
        # left == right
        if left + 1 == n + 1:
            # all versions are fine
            return -1
        return left + 1

```

- PR
  - https://github.com/huyfififi/coding-challenges/pull/14
  - https://github.com/rihib/leetcode/pull/33
  - https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/25
  - 

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
各PRの内容をまとめます。

---

**PR 1: huyfififi/coding-challenges#14** (Python)

*レビューコメントのポイント:*
- **変数名 `left_i` について** (colorbox): `i`がindexの略なら省略せず`index`と書くか、単に`left`にした方が良い
- **`left`/`right`の意味** (oda): 「左が100で右が200」だけでは意味が分からない。本当は「99以前はgood確認済み、100以降は未確認」という情報を表現したい。変数名を長くするか、ループ不変条件をコメントで明記すべき
- **オーバーフロー対策**: `mid = left + (right - left) / 2` の書き方を覚えておくと良い（Pythonでは不要だが他言語で有用）
- **エッジケース** (oda): `n=1`でwhileに入らないケースへの注意

---

**PR 2: rihib/leetcode#33** (Go)

*レビューコメントのポイント:*
- **「半閉区画」→「半開区間」** (nodchip): 用語の訂正
- **ループ不変条件の説明が不十分** (nodchip): 「rightは常にtrueの範囲に収まる」「leftはtrueに移動する」などの説明に対し、「なぜそうなるのか」「right=midにしてよい理由は？」など根本的な理由を問う質問が多数
- **isBadVersion()の呼び出し回数** (oda): ループ1回で最大3回呼ばれる実装があり、APIが重い処理の場合に問題になる可能性

*特徴:* nodchipさんから二分探索の理解を深めるための鋭い質問が多く、ループ不変条件や境界の意味を正確に説明できるかが問われている

---

**PR 3: NobukiFukui/Grind75-ProgrammingTraining#25** (Python)

*レビューコメントのポイント:*
- **整数除算** (liquo-rice): `int(left + (right - left) / 2)` より `//` を使う方が分かりやすい
- colorbox: 「良さそうに見えます」

*特徴:* シンプルなPRで、軽微なPythonの書き方の指摘のみ

---

**共通して学べるポイント:**
1. **変数名とループ不変条件の明示** - `left`/`right`だけでは意味が曖昧なので、不変条件をコメントで書く
2. **APIコールの回数を意識** - `isBadVersion()`が重い可能性を考慮
3. **Pythonでは `//` を使う** - `int(... / 2)` より明確

## Step 3

### 実装3

- 何も見ずに書き直し

```python
# The isBadVersion API is already defined for you.
# def isBadVersion(version: int) -> bool:

class Solution:
    def firstBadVersion(self, n: int) -> int:
        """Invariant:
        - all versions in [1, left) are fine
        - all versions in [right, n] are bad
        """
        left = 1
        right = n + 1
        while left < right:
            mid = left + (right - left) // 2
            if isBadVersion(mid):
                right = mid
            else:
                left = mid + 1
        # left == right
        if right == n + 1:
            return -1
        return right
```

- 振り返り
  - first Trueを見つけるとき、右側に対する不変条件がi > rightよりi >= rightとなっていた方が、返り値が考えやすいと感じた。
  - [#実装3](#実装3)は、i < leftとi >= rightに対し不変条件を考えているため返り値がrightと分かりやすい。
  - 停止条件もleft == rightと重なるため、分かりやすいと感じる。

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