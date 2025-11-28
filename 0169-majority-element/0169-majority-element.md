# 169. Majority Element

- URL: https://leetcode.com/problems/majority-element/
- Difficulty: Easy
- Tags: Array, Hash Table, Divide and Conquer, Sorting, Counting
- Notebook: https://share.solve.it.com/d/b36c621fe2f8f83ab1d3a0c69e6ce31d

## Step 1

### 実装1

- アルゴリズムの選択
  - 線形走査で登場回数を数え、最大なものを答える
  - 過半数という性質をうまく活かす方法が思いつかない。
- 実装
  - collections.Counterなどは使わないことにする。
- 計算量
  - Time: O(n)
  - Extra Space: O(n)

```python
class Solution:
    def majorityElement(self, nums: list[int]) -> int:
        num_to_count = {}
        for num in nums:
            if num not in num_to_count:
                num_to_count[num] = 0
            num_to_count[num] += 1
        return max(num_to_count, key=lambda x: num_to_count[x])
```

- 振り返り
  - O(1) spaceにする方法がわからない。
  - max()で、key=num_to_count.getでよかった。（思い出せなかった。）

## Step 2

### 実装2

- アルゴリズムの再考。
  - O(1) Spaceの方法をLLMに教えてもらった。
  - Boyer-Moore Voting Algorithm
    - 暫定のmajorityとそれ以外に分けて考える。
    - scoreを、majorityが来たら+1、それ以外が来たら-1、とする。
    - scoreが0になったら次に来た数字をmajorityと設定する。
    - 最後にmajorityになっていた数字をreturnする。
    - 過半数の性質を活用できている。
- 計算量
  - Time: O(n)
  - Extra Space: O(1)

```python
class Solution:
    def majorityElement(self, nums: list[int]) -> int:
        if not nums:
            raise ValueError("nums must not be empty")
        
        count = 0
        for num in nums:
            if count == 0:
                majority = num
                count = 1
                continue
            
            if num == majority:
                count += 1
            else:
                count -= 1
        
        return majority
```

- コメント集：空
- PR
  - https://github.com/ryosuketc/leetcode_grind75/pull/19
  - https://github.com/huyfififi/coding-challenges/pull/19
  - https://github.com/rihib/leetcode/pull/37
  - https://github.com/erutako/leetcode/pull/4
  - https://github.com/Kitaken0107/GrindEasy/pull/19

**Q:** 上記に他の人が解いたPRをまとめています。それぞれのPRの代表的なソリューションとレビューコメントをチェックし、PRごとの内容を簡潔にまとめてください。


**A:**
各PRの内容をまとめます：

---

**1. ryosuketc/leetcode_grind75#19 (C++)**
- **解法**: HashMapでカウント → 過半数を超えたら即return
- **レビューコメント**: 
  - 「最大得票数を探すより、過半数を超えたら即返す方がシンプル」(nodchip)

---

**2. huyfififi/coding-challenges#19 (Python)**
- **解法**: 複数アプローチ（Counter, Heap, Sort, QuickSelect, Boyer-Moore）
- **レビューコメント**: なし（セルフマージ）
- **特徴**: Boyer-Mooreの説明やQuickSelectのバリエーションも含む

---

**3. rihib/leetcode#37 (Go)**
- **解法**: HashMap + Boyer-Moore Voting
- **レビューコメント**:
  - `candidate, count := nums[0], 0` の1行2変数初期化について「目を動かす距離が長くなるので好まない」(oda) ※趣味の範囲
- **補足**: `len()`を複数回呼ぶオーバーヘッドは1ns程度で無視できる

---

**4. erutako/leetcode#4 (Python)**
- **解法**: HashMapでカウント
- **レビューコメント**:
  - ループ末尾が `return` で終わるより、`continue` と順序を入れ替える方が良い(oda)
  - 著者の返信: 「過半数発見時に即returnしたい意図を強調するなら現状でOKかも」

---

**5. Kitaken0107/GrindEasy#19 (Python)**
- **解法**: defaultdict + カウント
- **レビューコメント**:
  - 変数名 `cnt_nums` → `num_to_count` の方が良い(nodchip)
  - `nums_len = len(nums)` は不要、直接 `len(nums)` で十分(nodchip, oda)
  - `//` 演算子の前後にスペースを入れる（PEP8準拠）(nodchip)

---

気になるポイントや、さらに深掘りしたい部分はありますか？

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