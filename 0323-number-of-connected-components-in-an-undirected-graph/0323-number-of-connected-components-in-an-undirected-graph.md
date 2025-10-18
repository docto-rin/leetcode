## Step 1

- 問題文
  - `n`ノードの無向グラフが与えられる。
  - 辺は配列`edges`で与えられる。`edges[i] = [a, b]`はノードaとノードbの接続を表す。
  - 各ノードは`0`~`n-1`でナンバリングされている。
  - 連結成分の個数を返す。
  - 制約：
    - 1 <= n <= 100
    - 0 <= edges.length <= n * (n - 1) / 2

### 実装1

- アルゴリズムの選択
  - DFS/BFSで良いだろう。
  - 辺の個数の制約を見て、情報量ゼロだな...と思った。
- 実装
  - edgesはアクセス性が悪いので先に走査して隣接リスト形式に直しておく。
  - 訪問済み管理の変数を、traverseする関数からどうアクセスするか毎回迷う。
    - ナイーブだがnonlocal変数（Enclosingレベル）へのmutationをすることにした。
    - visited変数を引数で渡したところで、deepcopyしない限り実態は同じなので、引数には加えない。
    - また、いちいち内部でdeepcopyする意味もないと思った。（重くなるだけ）
- 計算量
  - Time: O(V + E)
  - Space: O(V + E)

```python3
from typing import List
from collections import deque


class Solution:
    def countComponents(self, n: int, edges: List[List[int]]) -> int:
        adjacency_list = [[] for _ in range(n)]
        for node1, node2 in edges:
            adjacency_list[node1].append(node2)
            adjacency_list[node2].append(node1)
        
        visited = [False] * n
        def traverse(start):
            frontiers = deque([start])
            visited[start] = True
            while frontiers:
                node = frontiers.popleft()
                for neighbor in adjacency_list[node]:
                    if visited[neighbor]:
                        continue
                    frontiers.append(neighbor)
                    visited[neighbor] = True
        
        num_components = 0
        for node in range(n):
            if visited[node]:
                continue
            traverse(node)
            num_components += 1
        
        return num_components
```

- 特に迷いなく書けた。ここまで7分。
- ところで、そろそろUnionFindを勉強しなければならない気がしています。

## Step 2

- https://discord.com/channels/1084280443945353267/1183683738635346001/1197738650998415500
  - > union-find  は、微妙に常識から外れるかな(多くの人が知っているだろうが知らなくてもドン引きはされない)、くらいの感覚です。DFS による解法のほうは常識でしょう。
    >
    > union-find は、私はこういうイメージです。...
    >
    > 高速化の技法がpath compression, splitting, halving, union by size, union by rankとあるようで、...
- https://youtu.be/nclfErc9pbE?si=QP-DUn3SNRnTabkK
  - 以前コーディングテスト対策でUnionFindを勉強したときに、これを見て理解しました。
  - unionルーチンをメソッド名uniteで実装する理由など、豆知識を知れます。

### 実装2

- UnionFind (path compression & union by size)
- 計算量
  - 操作の償却（均し）計算量: O(α(n)) （逆アッカーマン関数、ほぼO(1)とみなせる。）
  - Time: O(V + E * α(V))
  - Space: O(V)

```python3
from typing import List


class DisjointSetUnion:
    def __init__(self, size: int) -> None:
        self.parent: List[int] = list(range(size))
        self.group_size: List[int] = [1] * size
        self.num_groups: int = size

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            # path compression
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x: int, y: int) -> bool:
        """Return True if merged, False if already same."""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        # union by size
        if self.group_size[rx] < self.group_size[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        self.group_size[rx] += self.group_size[ry]
        self.num_groups -= 1
        return True


class Solution:
    def countComponents(self, n: int, edges: List[List[int]]) -> int:
        disjoint_set_union = DisjointSetUnion(n)
        for u, v in edges:
            disjoint_set_union.union(u, v)
        return disjoint_set_union.num_groups
```

<details>
  <summary>UnionFind高速化まとめ by GPT-5</summary>

  
## 1. Path Compression（経路圧縮）

### 概念

`find(x)`を呼び出したとき、探索経路上の全ノードを最終的な代表（root）に**直接つなぎ直す**最適化です。

### 目的

* 木の高さを極端に低くする（平均ほぼ2程度になる）
* 次回以降の`find`がほぼO(1)になる

### 図示

```
Before:
1 → 2 → 3 → 4  (root)

After find(1):
1 → 4
2 → 4
3 → 4
4 → 4
```

### 実装（典型）

```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # 経路圧縮
    return parent[x]
```

## 2. Path Splitting（経路分割）

### 概念

`find`の探索中に、**各ノードを親の親につなぎ直す**。
最終的なrootまでたどる必要はなく、走査中に構造を平坦化していく。

### 図示

```
Before:
1 → 2 → 3 → 4

Step-by-step in find(1):
1 → 3
2 → 4
3 → 4
```

### 実装例

```python
def find(x):
    while parent[x] != x:
        parent[x], x = parent[parent[x]], parent[x]  # 親を1段スキップ
    return x
```

### 特徴

* `find`の再帰を使わず反復的に書ける。
* 経路圧縮とほぼ同等の理論的性能（O(α(n))）。

## 3. Path Halving（経路半減）

### 概念

探索経路上で、**ノードの親を祖父ノード（親の親）に更新する**。
「半分の高さにする」ので *halving*。

### 図示

```
Before:
1 → 2 → 3 → 4 → 5

After one halving pass:
1 → 3
2 → 4
3 → 5
```

### 実装例

```python
def find(x):
    while parent[x] != x:
        parent[x] = parent[parent[x]]  # 親を祖父に更新
        x = parent[x]
    return x
```

### 特徴

* path splittingよりやや簡単。
* 両者ともpath compressionと同等のオーダー（実用上ほぼ同速）。

## 4. Union by Size（サイズによる併合）

### 概念

`union(x, y)`の際に、

> 小さい集合を大きい集合のrootの下にぶら下げる
> というルールを設ける。

### 目的

* 木が極端に偏るのを防ぐ（高さ増加を抑える）

### 実装例

```python
def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if size[rx] < size[ry]:
        rx, ry = ry, rx
    parent[ry] = rx
    size[rx] += size[ry]
```

### 効果

* 経路圧縮と組み合わせると、理論上O(α(n))。
* サイズを追跡できるので、成分サイズ計算にも使える。

## 5. Union by Rank（ランクによる併合）

### 概念

各rootに「ランク」（木の高さの近似）を記録し、

> ランクの小さい方を大きい方にぶら下げる
> という戦略をとる。

### 実装例

```python
def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if rank[rx] < rank[ry]:
        parent[rx] = ry
    elif rank[rx] > rank[ry]:
        parent[ry] = rx
    else:
        parent[ry] = rx
        rank[rx] += 1
```

### ランクとサイズの違い

* **rank**: 木の高さを近似。理論的にはサイズより厳密な高さ制御。
* **size**: 実装が単純で、実務ではrankよりも扱いやすい。

### 性能差

* 両者とも理論的には同等。ほぼ定数時間（O(α(n))）で動作する。

## まとめ表

| 手法名                  | 対象操作    | 内容                 | 効果      |
| -------------------- | ------- | ------------------ | ------- |
| **Path Compression** | `find`  | 経路上のノードをすべてroot直下に | 木をほぼ平坦化 |
| **Path Splitting**   | `find`  | 各ノードを親の親につなぐ       | 同上（非再帰） |
| **Path Halving**     | `find`  | 各ノードを祖父ノードにつなぐ     | 同上（半減）  |
| **Union by Size**    | `union` | 小集合を大集合に併合         | 木の高さ抑制  |
| **Union by Rank**    | `union` | 小ランクを大ランクに併合       | 木の高さ抑制  |

## 最適な組み合わせ

理論・実装ともに最も使われるのは：

> **Union by Size (or Rank) + Path Compression**

この組み合わせで償却計算量は
`O(α(n))`（逆アッカーマン関数）になり、
実質的にすべての操作がほぼO(1)で動作します。
</details>

## Step 3

[実装1](#実装1)
