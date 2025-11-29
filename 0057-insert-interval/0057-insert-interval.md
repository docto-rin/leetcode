# 57. Insert Interval

- URL: https://leetcode.com/problems/insert-interval
- Difficulty: Medium
- Tags: Array
- Notebook: https://share.solve.it.com/d/2763f1526713e6f2cb009d5473284356

## Step 1

### 実装1

- アルゴリズムの選択
  - 説明のため、start, end = newIntervalとunpackする。
  - start, endがintervalsでどのあたりに位置するかを別々に決定したい。
  - 例えばstartの位置は各区間の開始時刻、つまりintervals[i][0]と比較して決定する。
    - 二分探索すればいい。
  - 上で求めたstartの位置をstart_indexとする。
  - もし開始時刻が他のintervalとintervalの間にある場合、start_indexは区間左端の挿入位置になる。
  - もし開始時刻が他のintervalの中にある場合、そのintervalとマージする必要がある。
    - if start >= previous_end: start = previous_start, then start_index -= 1
  - endも同様。
- 実装
  - 「in-placeである必要はない。」と書いてあるので、返り値はintervalsのスライスを利用して作る。
- 計算量
  - Time: O(n)
    - 主要なロジックはO(log n)だがスライスでO(n)
  - Extra Space: O(n)

```python
import bisect
from operator import itemgetter


class Solution:
    def insert(
        self, intervals: list[list[int]], newInterval: list[int]
    ) -> list[list[int]]:
        start, end = newInterval
        start_index = bisect.bisect_left(intervals, start, key=itemgetter(0))
        end_index = bisect.bisect_left(intervals, end, key=itemgetter(1))

        if start_index > 0:
            previous_start, previous_end = intervals[start_index - 1]
            if start <= previous_end:
                start = previous_start
                start_index -= 1

        if end_index < len(intervals):
            next_start, next_end = intervals[end_index]
            if end >= next_start:
                end = next_end
                end_index += 1

        result = []
        result += intervals[:start_index]
        result += [[start, end]]
        result += intervals[end_index:]
        return result

```

- 20分くらいかかってしまった。
- 振り返り
  - 手作業でやっていることをコードに落とし込むのに苦戦した。
  - また、本来はin-placeでやるかどうかを引数で選択可能にしたいが、in-placeをさっと思いつかない。

## Step 2

🙋**Q:** ここまでのStep 1のコードについてのやり取りをまとめてください。


🤖**A:**
**Step 1 コードレビューのまとめ**

**正確性**: 二分探索を使ったアプローチは正しく動作することを確認しました。`end_index`で`itemgetter(1)`を使い、その後`next_start`と比較してマージ判定する設計は意図通りです。

**改善点**:
1. **結果の構築**: `+=`を3回使う代わりに、`return intervals[:start_index] + [[start, end]] + intervals[end_index:]`で簡潔に書ける（ただし80文字制限を考慮して分割するのも良い判断）
2. **変数名**: `start`, `end`が途中でマージ後の値に変わるので、`merged_start`, `merged_end`のような名前で意図を明確にすると良い

**in-place実装について**:
- スライス代入 `intervals[start_index:end_index] = [[merged_start, merged_end]]` がシンプルで効率的
- 計算量は両方O(n)だが、スライス代入はCPython内部で最適化されているため実行速度で有利
- in-placeの主な利点はメモリ使用量の削減

**エッジケース**: 空の`intervals`でも正しく動作することを確認

### 実装2

- 実装
  - in_placeかどうか引数で選べるようにした。
  - 最後の部分をスライス代入を使ってシンプルにした。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
import bisect
from operator import itemgetter


class Solution:
    def insert(
        self,
        intervals: list[list[int]],
        newInterval: list[int],
        in_place: bool = False
    ) -> list[list[int]]:
        if not in_place:
            intervals = intervals.copy()

        start, end = newInterval
        start_index = bisect.bisect_left(intervals, start, key=itemgetter(0))
        end_index = bisect.bisect_left(intervals, end, key=itemgetter(1))

        if start_index > 0:
            previous_start, previous_end = intervals[start_index - 1]
            if start <= previous_end:
                start = previous_start
                start_index -= 1

        if end_index < len(intervals):
            next_start, next_end = intervals[end_index]
            if end >= next_start:
                end = next_end
                end_index += 1

        intervals[start_index:end_index] = [[start, end]]
        return intervals

```

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.i563jj9iu5hi)
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1214768828957859931
    - > あなたはビルの管理人として、受付に座っていてビルが開いていた時間を記録につけたいです。
    - 全ての部屋の使用時間帯を別々に記録したいのではなく、それを一つの直線に射影したものが欲しいだけ、ということか。
    - > vector<vector<int>> で管理しているのが不便すぎます
    - これは書いている時に感じたので、せめて関心のあるものはunpackするよう心がけた。
    - starts, endsを同じ長さのvector<int>で管理するのが自然そう。
- PR
  - https://github.com/ryosuketc/leetcode_grind75/pull/26
  - https://github.com/huyfififi/coding-challenges/pull/26

🙋**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


🤖**A:**
PRの内容を確認しますね。

両PRの内容を確認しました。以下にまとめます。

---

**PR 1: ryosuketc/leetcode_grind75 #26**

**解法**: C++で、フラグ管理を使った1ループアプローチを試みたが苦戦。最終的に3フェーズに分けるアプローチに落ち着いた。

**主なレビューコメント**:
- **oda**: 「被っているか判定する関数」と「くっつける関数」を分けて考えることを提案。低次の操作（`[0][1]`の比較）と高次の概念（overlap判定）の距離が遠いと混乱しやすい
- **oda**: フラグ管理は「部屋にホワイトボードを置いてシフト間で共有する」ようなもの。状態が一方向に遷移するなら別々のループ（マニュアル切り替え）の方が良い
- **nodchip**: 入力の`intervals`を変更しているのは問題。呼び出し側からすると驚く
- **oda**: queueのような制限されたインターフェースで考えると整理しやすい

**学び**: 1ループでフラグ管理するよりも、3フェーズ（左側→オーバーラップ処理→右側）に分けた方がシンプルになる

---

**PR 2: huyfififi/coding-challenges #26**

**解法**: Python。走査しながら新しいリストを作成するアプローチ。Step 2の解法がシンプル。

**主なレビューコメント**:
- **nodchip**: 別アプローチとして「newIntervalを追加→ソート→重複区間を併合」を提案（計算量はO(n log n)になるが、条件分岐がシンプルになる）

**学び**: 問題文に「新しいリストを作っていい」とあることに気づいてからスムーズに解けた。入力の制約をよく読むことが大事

---

あなたの二分探索アプローチは、これらとは異なる視点で、O(log n)の探索部分を活かした解法ですね。何か気になる点はありますか？

### 実装3

- 実装の検討
  - https://github.com/ryosuketc/leetcode_grind75/pull/26 で検討されたいた3フェーズのPython版。
  - メインロジックの計算量はO(log n) -> O(n)に悪化するが、意図がわかりやすいかもしれない。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
class Solution:
    def insert(
        self, intervals: list[list[int]], newInterval: list[int]
    ) -> list[list[int]]:
        result = []
        start, end = newInterval
        i = 0
        n = len(intervals)
        
        while i < n and intervals[i][1] < start:
            result.append(intervals[i])
            i += 1
        
        while i < n and intervals[i][0] <= end:
            start = min(start, intervals[i][0])
            end = max(end, intervals[i][1])
            i += 1
        result.append([start, end])
        
        result.extend(intervals[i:])
        
        return result
```

## Step 3

[実装2](#実装2)

## Step 4

自分のPR：（自分のPRへのリンク）

🙋**Q:** それぞれのレビューコメントについて、もう少し詳しく教えてください。

### 実装4

- レビューを受けて書き直し
  - （修正点）

```python
class Solution:
    pass
```