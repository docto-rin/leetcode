## Step 1

- 問題文
  - `m x n`のバイナリ行列`grid`を受け取る。島とは、4方向（水平または垂直）に接続された1のグループ（陸地を表す）を指す。グリッドの四辺はすべて水に囲まれていると仮定できる。
  - 島の面積とは、島内の値1を持つセルの数です。
  - グリッド内の島の最大面積を返してください。島が存在しない場合は0を返してください。
  - 制約：
    - m == grid.length
    - n == grid[i].length
    - 1 <= m, n <= 50
    - grid[i][j] is either 0 or 1.
- アルゴリズムの選択
  - BFSかDFS。どちらでも変わらなさそう。
- 実装
  - collections.deque を使ってiterativeに書く。
  - [前回の問題](https://leetcode.com/problems/number-of-islands/)の復習だと思って、綺麗に書く。

### 実装1

- 時間計算量: O(E + V) = O(m * n)
- 空間計算量: O(m * n)

```python3
import copy
from collections import deque


class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        grid = copy.deepcopy(grid)
        num_rows = len(grid)
        num_cols = len(grid[0])

        def is_land(row, col):
            return (
                0 <= row < num_rows
                and 0 <= col < num_cols
                and grid[row][col] == 1
            )
        
        def traverse(row_start, col_start):
            frontiers = deque()
            frontiers.append((row_start, col_start))
            grid[row_start][col_start] = 0
            
            def push_point(row, col):
                if is_land(row, col):
                    frontiers.append((row, col))
                    grid[row][col] = 0
                    return 1
                return 0
            
            count = 1
            while frontiers:
                row, col = frontiers.popleft()
                count += push_point(row + 1, col)
                count += push_point(row - 1, col)
                count += push_point(row, col + 1)
                count += push_point(row, col - 1)
            return count
        
        max_area = 0
        for row in range(num_rows):
            for col in range(num_cols):
                if is_land(row, col):
                    area = traverse(row, col)
                    max_area = max(max_area, area)
        return max_area
```

特に問題なし。ここまで16分。

DFSにする場合はpopleft()をpop()にすれば良いだろう。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.f28i04p206ak)
  - https://discord.com/channels/1084280443945353267/1192736784354918470/1226906670697938995
    - > 個人的には nonlocal 好きではないので、
    - 同感。言語仕様としてスコープ外の再束縛は標準的じゃなさそうなので、なるべく避けた実装を心がけたい。
    - スコープ外は、読み取りか、ミューテーションだけするようにする。
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1322208909103992873
    - > const int DELTA[][] = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}}; といった配列を用意しておき、
    - 進む方向を定数として定義してforループを回す方が、進む方向が将棋の駒のように増えたとしても拡張しやすい。
  - https://discord.com/channels/1084280443945353267/1307605446538039337/1334350726683693086
    - > 同じ意味のものが繰り返しているときには [] のほうが多い気がします。Tuple も immutable という点ではいいかもしれません。
    - たしかに。[(row_diff, col_diff), ...]が自然な気がする。
  - https://discord.com/channels/1084280443945353267/1218823830743547914/1343160822268166264
    - > これも趣味の範囲として一つだと思いますが、ラムダかメソッドを使って queue を追加するという処理を分けて、+1 などはベタ書きにするかもしれません。
    - 逆に、4方向しかないから書き下す方がわかりやすいという意見。

## Step 3

[実装1](#実装1)
