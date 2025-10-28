## Step 1

- 問題文
  - 前回の問題と共通の部分：
    - あなたはプロの強盗で、ある通り沿いの家々を襲撃して金を盗んでいこうとしている。
    - 各家の所持金を表す整数配列numsが与えられる。
    - 任意の連続する2つの家から同時に盗むと警察に通報される。
    - 通報されずに盗める合計金額の最大値を返せ。
  - [前回の問題](https://leetcode.com/problems/house-robber-ii/)に、nums[0]とnums[-1]も連続（円形に並んでいる）とみなす制約を追加。
  - 制約：
    - 1 <= nums.length <= 100
    - 0 <= nums[i] <= 1000

### 実装1

- アルゴリズムの選択
  - 前回の問題では、with robbing last houseとwithoutでのmax totalを漸化式で求めていった。
  - なので第一印象としては、この2つの状態をそれぞれさらに、with robbing first houseとwithoutで分割すればいいかなと思った。
  - しかし、with robbing first houseとwithoutはそれぞれ独立した系列なので、わざわざ同時に取り扱うのは外見が過度に複雑になって嫌だなと感じた。
  - 少し考え、nums[-1]を除外して解いた結果と、nums[0]を除外して解いた結果を別々に求め、最後にmaxでまとめたらスマートだと思った。
  - 2周しないといけないが、避けられないかなと思った。
- 実装
  - （前回の問題と同じな）コアロジックは関数化する。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
from typing import List


class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        
        def rob_section(left, right):
            without_last = 0  # max total without robbing last house
            with_last = 0  # max total with robbing last house
            for i in range(left, right + 1):
                without_last, with_last = (
                    max(without_last, with_last),
                    without_last + nums[i]
                )
            return max(without_last, with_last)
        
        return max(
            rob_section(0, len(nums) - 2),
            rob_section(1, len(nums) - 1)
        )
```

- ここまで7分。

## Step 2

- https://discordapp.com/channels/1084280443945353267/1196472827457589338/1427479212331372545
  - max()関数のdefault引数を使ったまとめ方。良さそう。
  - https://docs.python.org/ja/3/library/functions.html#max
  - > default 引数は与えられたイテラブルが空の場合に返すオブジェクトを指定します。 イテラブルが空で default が与えられていない場合 ValueError が送出されます。
  - [実装1](#実装1)の場合は、特別処理が空配列と長さ1の場合なので、まとめない方が可読性が高いと思う。（良いまとめ方がないと思う。）
- 今更ながら、状態を単一で考え、三項間漸化式でやっている方も結構いる。
  - https://github.com/Mike0121/LeetCode/pull/57/
  - https://github.com/akmhmgc/arai60/pull/31/
  - しかしながら、2つの状態での二項間漸化式を式変形でまとめているので、発想としてあまり自然には感じない。（いきなり思いつくのが厳しい。）

## Step 3

[実装1](#実装1)
