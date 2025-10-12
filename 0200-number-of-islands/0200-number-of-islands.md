## Step 1

- 問題文
  - `m x n`の2Dバイナリグリッド`grid`を受け取る。'1'が陸地、'0'が水を表す。このとき島の個数を答えよ。
  - 周囲が水で覆われており、水平または垂直方向に接続した陸地全体を島と定義する。なおgridの4辺は水に囲まれているとする。
  - 制約：
    - m == grid.length
    - n == grid[i].length
    - 1 <= m, n <= 300
    - grid[i][j] is '0' or '1'.
- アルゴリズムの選択
  - gridは無向グラフと解釈でき、無向グラフの連結成分の個数を答える問題である。
  - BFSかDFSでいいだろう。どちらが好ましいかはあまり想像つかない。
    - 1 <= m, n <= 300の長方形であり、グラフの幅が極端に広いわけでも、深いわけでもない（と思う）。
    - もし極端に幅が広いならOut of Memoryを考慮しDFSにする、など？
  - ひとまず根拠に乏しいがBFSでやる。
- 実装
  - BFSはキューで実装できる。pythonならdeque()を使うのがいいだろう。

### 実装1

BFS

- 時間計算量: O(V + E) = O(m * n)
- 空間計算量: O(m * n)

```python3
from typing import List, Tuple
from collections import deque


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def bfs_in_island(start: Tuple[int, int], visited: set):
            land_queue = deque()
            land_queue.append(start)
            while land_queue:
                land = land_queue.popleft()
                visited.add(land)
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if neighbor not in visited \
                        and 0 <= neighbor[0] < len(grid) and 0 <= neighbor[1] < len(grid[0]) \
                        and grid[neighbor[0]][neighbor[1]] == "1":
                        land_queue.append(neighbor)
            return visited

        visited = set()
        count = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if (i, j) not in visited and grid[i][j] == "1":
                    count += 1
                    visited = bfs_in_island((i, j), visited)
        return count
```

しかし、Time Limit Exceeded (39/49 testcases passed)だった。

非効率な部分があるのだろうか、一旦DFSにしてみる。

DFSは再帰か、ループ+Stackで実装できるが、stack overflowやデバッグのしやすさを考慮して後者を選択。

[実装1](#実装1)のland_queueをland_stackに置き換え、'bfs'->'dfs'に書き換えればそのまま動くはず。

### 実装2

DFS

```python3
from typing import List, Tuple


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def dfs_in_island(start: Tuple[int, int], visited: set):
            land_stack = []
            land_stack.append(start)
            while land_stack:
                land = land_stack.pop()
                visited.add(land)
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if neighbor not in visited \
                        and 0 <= neighbor[0] < len(grid) and 0 <= neighbor[1] < len(grid[0]) \
                        and grid[neighbor[0]][neighbor[1]] == "1":
                        land_stack.append(neighbor)
            return visited

        visited = set()
        count = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if (i, j) not in visited and grid[i][j] == "1":
                    count += 1
                    visited = dfs_in_island((i, j), visited)
        return count
```

Acceptedだった。
- Runtime: 283 ms
- Memory: 25.54 MB

実装1, 2をGPT-5にレビューしてもらったところ、TLEの原因そうなバグを教えてもらった。

> BFSで「キューに入れる時点で訪問済みにしない」実装になっているため、同じセルが複数回キューに積まれるケースが多発します。
> 格子では同一セルに対して最大4方向から到達し得るので、visited を pop 時に追加すると重複 enqueue が増え、BFSは特にそれが顕在化してTLEになりやすいです。
> 対策は「push 時に visited に入れる」か「push 時に grid を '0' などで潰す」ことです。

その通り。実装1を修正する。

### 実装3

BFSの重複エンキューするバグを修正。

```python3
from typing import List, Tuple
from collections import deque


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def bfs_in_island(start: Tuple[int, int], visited: set):
            land_queue = deque()
            land_queue.append(start)
            visited.add(start)
            while land_queue:
                land = land_queue.popleft()
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if neighbor not in visited \
                        and 0 <= neighbor[0] < len(grid) and 0 <= neighbor[1] < len(grid[0]) \
                        and grid[neighbor[0]][neighbor[1]] == "1":
                        land_queue.append(neighbor)
                        visited.add(neighbor)
            return visited

        visited = set()
        count = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if (i, j) not in visited and grid[i][j] == "1":
                    count += 1
                    visited = bfs_in_island((i, j), visited)
        return count
```

Acceptedになった。

Runtime: 280 ms
Memory: 25.27 MB

また、GPT-5の言うように、gridがすでに実質的にvisitedの機能を内包しているので、visitedは使わない方が高速かつ省メモリだろう。

### 実装4

BFS、DFSともに、visitedを廃止して、gridを"0"で潰す方法。

```python3
from typing import List, Tuple
from collections import deque


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def bfs_in_island(start: Tuple[int, int]):
            land_queue = deque()
            land_queue.append(start)
            grid[start[0]][start[1]] = "0"
            while land_queue:
                land = land_queue.popleft()
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if 0 <= neighbor[0] < len(grid) and 0 <= neighbor[1] < len(grid[0]) \
                        and grid[neighbor[0]][neighbor[1]] == "1":
                        land_queue.append(neighbor)
                        grid[neighbor[0]][neighbor[1]] = "0"
        
        def dfs_in_island(start: Tuple[int, int]):
            land_stack = []
            land_stack.append(start)
            grid[start[0]][start[1]] = "0"
            while land_stack:
                land = land_stack.pop()
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if 0 <= neighbor[0] < len(grid) and 0 <= neighbor[1] < len(grid[0]) \
                        and grid[neighbor[0]][neighbor[1]] == "1":
                        land_stack.append(neighbor)
                        grid[neighbor[0]][neighbor[1]] = "0"

        count = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if grid[i][j] == "1":
                    count += 1
                    bfs_in_island((i, j))
        return count
```

- BFS
  - Runtime: 266 ms
  - Memory: 20.4 MB
- DFS
  - Runtime: 272 ms
  - Memory: 20.4 MB

どちらの方法でも、Runtime、Memory両方改善している。

またここで、ローカルスコープ外のgridを書き換えているが、UnboundLocalErrorとかは出ないのだろうかと疑問に思い、調べた。

<details>
  <summary>スコープと束縛について</summary>

  - The global statement: https://docs.python.org/3/reference/simple_stmts.html#the-global-statement
- The nonlocal statement: https://docs.python.org/3/reference/simple_stmts.html#the-nonlocal-statement
- Naming and binding: https://docs.python.org/3/reference/executionmodel.html#naming-and-binding

読み取りとミューテーション（要素代入・属性代入・メソッド呼び出し）はOK、再束縛（そのものへ代入）は宣言が必要とのこと。

#### 外側へのミューテーション
```python3
def outer():
    x = [1, 2, 3]
    def inner():
        x[0] = 99
        # x = [101, 2, 3]
    inner()
    print(x)

outer()  # => [99, 2, 3]
```
#### ローカル変数の再束縛（外側は不変）
```python3
def outer():
    x = [1, 2, 3]
    def inner():
        # x[0] = 99
        x = [101, 2, 3]
    inner()
    print(x)

outer()  # => [1, 2, 3]
```
#### 外側へのミューテーションが、未定義なローカル変数にかかる
```python3
def outer():
    x = [1, 2, 3]
    def inner():
        x[0] = 99
        x = [101, 2, 3]
    inner()
    print(x)

outer()  # => UnboundLocalError: cannot access local variable 'x' where it is not associated with a value

# コンパイラはまずinner()の中にあるx = [99, 2, 3]をみて、変数xはinnerのローカル変数と決めてしまう
# その上でx[0] = 99を実行しようとして、まだ値が束縛されていないローカル変数にアクセスしようとするから、UnboundLocalError
```
#### 外側の変数の再束縛
```python3
def outer():
    x = [1, 2, 3]
    def inner():
        nonlocal x
        x[0] = 99
        x = [101, 2, 3]
    inner()
    print(x)

outer()  # => [101, 2, 3]
```

なお、Pythonのスコープ階層は LEGBルールとなっている。
| スコープ階層        | 例                      |
| ------------- | ---------------------- |
| L (Local)     | 現在の関数内                 |
| E (Enclosing) | 外側の関数（= 関数クロージャ）       |
| G (Global)    | モジュール全体                |
| B (Built-in)  | `len` や `list` などの組み込み |
nonlocalはEnclosing階層の変数のこと。
</details>

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.6z8kd1z4d8gp)
  - https://discord.com/channels/1084280443945353267/1313503038358552607/1322228897093521458
    - neighborごとの処理を関数化する。
  - https://discord.com/channels/1084280443945353267/1295324533963751465/1339578918994444329
    - > このコードのざっくりとした感想として、UnionFind の設計が好ましくないですね。
      - 1D想定での実装がナンセンスということでしょうか。
      - UnionFindの存在をすっかり忘れていたので、思い出していきたい。
- https://github.com/tarinaihitori/leetcode/pull/17
  - > 与えられた grid に破壊的な変更を加えていることは認識できていますかね？破壊的な変更は望ましくないケースが多いと思うので、基本的に避けたほうがよいと思います。grid とは別で訪問済みかどうかを管理すると破壊的な変更が不要になります。
    - この観点で行くと、非破壊的にやるならvisited = set()を使ってもいいし、numsをdeepcopyしておいても良さそう。hashtableの管理は重いのと、ロジックの再利用性からやるなら後者の方でやりそう。

## Step 3

### 実装5

BFSとDFSを切り替え可能にするためのwrapperクラスを作ってみたが、Runtimeがやや悪化し、イマイチだった。

```python3
from typing import List, Tuple
from collections import deque


class LandQueue:
    def __init__(self, method: str = "bfs"):
        self.queue = deque()
        self._append = self.queue.append

        if method == "bfs":
            self._pop = self.queue.popleft
        elif method == "dfs":
            self._pop = self.queue.pop
        else:
            raise ValueError(f"Unknown method: {method}")
    
    def __bool__(self) -> bool:
        return bool(self.queue)

    def append(self, land):
        self._append(land)

    def pop(self):
        return self._pop()


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def is_land(row: int, col: int) -> bool:
            return 0 <= row < len(grid) \
                and 0 <= col < len(grid[0]) \
                and grid[row][col] == "1"

        def walk_in_island(start: Tuple[int, int], method: str = "bfs"):
            land_queue = LandQueue(method)
            land_queue.append(start)
            grid[start[0]][start[1]] = "0"

            while land_queue:
                land = land_queue.pop()
                for direction in [[1, 0], [-1, 0], [0, 1], [0, -1]]:
                    neighbor = (land[0] + direction[0], land[1] + direction[1])
                    if is_land(neighbor[0], neighbor[1]):
                        land_queue.append(neighbor)
                        grid[neighbor[0]][neighbor[1]] = "0"

        count = 0
        for row in range(len(grid)):
            for col in range(len(grid[0])):
                if is_land(row, col):
                    count += 1
                    walk_in_island((row, col), "dfs")
        return count
```

こうみると、BFSとDFSの違いはpopを左から取るか右から取るかしか違いがないように見えて面白い。

### 実装6

DFSを再帰でも実装しておく。stackベースよりも直感的で実装しやすいが、積極的には選びたくない。

```python3
from typing import List, Tuple


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def is_land(point: Tuple[int, int]) -> bool:
            return 0 <= point[0] < len(grid) \
                and 0 <= point[1] < len(grid[0]) \
                and grid[point[0]][point[1]] == "1"

        def walk_in_island(start: Tuple[int, int]):
            grid[start[0]][start[1]] = "0"
            directions = ((1, 0), (-1, 0), (0, 1), (0, -1))
            for direction in directions:
                neighbor = (start[0] + direction[0], start[1] + direction[1])
                if is_land(neighbor):
                    walk_in_island(neighbor)
        
        count = 0
        for row in range(len(grid)):
            for col in range(len(grid[0])):
                if is_land((row, col)):
                    count += 1
                    walk_in_island((row, col))
        return count
```
