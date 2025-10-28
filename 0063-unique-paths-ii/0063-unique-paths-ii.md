## Step 1

- 問題文
  - 大きさが`m x n`のバイナリが入った配列`grid`を与えられる。
  - バイナリは障害物の有無を表している。
  - ロボットが左上から右下まで行く方法の総数を返せ。
  - ただし、経路上に障害物があってはならない。
  - 制約：
    - m == obstacleGrid.length
    - n == obstacleGrid[i].length
    - 1 <= m, n <= 100
    - obstacleGrid[i][j] is 0 or 1.

### 実装1

- アルゴリズムの選択
  - 動的計画法が最適と考えられる。
  - 障害物の個数が少ない場合は、（障害物を考慮しない経路の総数）-（障害物を通る経路の総数）とかでも可能だが、この解き方は人間向きである。
- 実装
  - 1次元テーブルで可能
  - 0行目、0列目は、[前回の問題](https://leetcode.com/problems/unique-paths/)であればskipでよかったが、今回はgridでの判定が必要。
- 計算量
  - Time: O(mn)
  - Space: O(n)

```python3
from typing import List


class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])
        count = [1] + [0] * (num_cols - 1)  # by column
        for row in range(num_rows):
            for col in range(num_cols):
                if obstacleGrid[row][col] == 1:
                    count[col] = 0
                    continue
                if col == 0:
                    continue
                count[col] += count[col - 1]
        return count[num_cols - 1]
```

### 実装2

- 折角なのでtop-down（再帰+メモ化）でも書いてみる

```python3
from typing import List
from functools import cache


class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        
        @cache
        def move_to(row, col):
            """move to point (row, col) on obstacleGrid.
            return the number of possible unique paths.
            """
            if obstacleGrid[row][col] == 1:
                return 0
            if row == 0 and col == 0:
                return 1
            if row == 0:
                return move_to(0, col - 1)
            if col == 0:
                return move_to(row - 1, 0)
            
            return move_to(row - 1, col) + move_to(row, col - 1)
        
        return move_to(len(obstacleGrid) - 1, len(obstacleGrid[0]) - 1)
```

- 特に迷いなくかけた。
- 答えが0になるあらゆるパターンでearly returnになることに気づき、top-downの優れている点だと感じた。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.qdbwc4eyd7p0)
  - https://discord.com/channels/1084280443945353267/1337642831824814192/1363934709503103137
    - > [0][0]にアクセスする前に一応チェックしてもいいかもしれませんね。問題文に制約があるにせよ。
    - その通りですね。

## Step 3

### 実装3

- [実装1](#実装1)に異常な入力のチェックを追加。

```python3
from typing import List


class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        if not obstacleGrid or not obstacleGrid[0]:
            return 0 
        
        if obstacleGrid[0][0] == 1 or obstacleGrid[-1][-1] == 1:
            return 0
        
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])
        count = [1] + [0] * (num_cols - 1)  # by column

        for row in range(num_rows):
            for col in range(num_cols):
                if obstacleGrid[row][col] == 1:
                    count[col] = 0
                    continue
                if col == 0:
                    continue
                count[col] += count[col - 1]
        return count[num_cols - 1]
```
