## Step 1

- 問題文
  - 異なる整数の配列`candidates`とターゲットの整数`target`が与えられる。
  - 和が`target`になるような、すべてのユニークな`candidates`から選ぶ組み合わせのリストを返せ。順序は問わない。
  - `candidates`からは同じ数字から何度も選べる。
  - 制約：
    - The test cases are generated such that the number of unique combinations that sum up to target is less than 150 combinations for the given input.
    - 1 <= candidates.length <= 30
    - 2 <= candidates[i] <= 40
    - All elements of candidates are distinct.
    - 1 <= target <= 40

### 実装1

- アルゴリズムの選択
  - candidates.lengthは控え目だが、枝刈りは前提。
  - candidates[i]は正なので、targetより大きくなったら枝刈り。
  - backtrackingでできそう。
    - 現在のcombination, sum, candidatesのindexの3つを管理すれば良さそう。
    - candidates[index]を1つ選ぶか、indexをインクリメントするか、の2択で被らない。
- 実装
  - loop + stackで書く。

```python3
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        results = []
        stack = [([], 0, 0)]  # (combination, sum, next_index)
        while stack:
            combination, total, index = stack.pop()
            
            if total == target:
                results.append(combination)
                continue
            if total > target:
                continue
            if index == len(candidates):
                continue
            
            stack.append((combination, total, index + 1))
            added = candidates[index]
            stack.append((combination + [added], total + added, index))
        
        return results
```

- ここまで10分。

### 実装2

- candidatesをあらかじめ降順にソートしておくと実行時間が2倍以上速くなった。
  - 枝刈りが早期に起き、走査数が大きく減るため。
  - targetより大きいcandidates[i]は初期に1回チェックするだけで良くなる。

```python3
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        descending_candidates = sorted(candidates, reverse=True)
        results = []
        stack = [([], 0, 0)]  # (combination, sum, next_index)
        while stack:
            combination, total, index = stack.pop()
            
            if total == target:
                results.append(combination)
                continue
            if total > target:
                continue
            if index == len(descending_candidates):
                continue
            
            stack.append((combination, total, index + 1))
            added = descending_candidates[index]
            stack.append((combination + [added], total + added, index))
        
        return results
```

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.vo5b5dbpqng)
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1265717341178822787
    - > [A, A] まで使うことが確定していて B 以降しか使ってはいけないという状況下で、
      > - B を一つ使うか、C 以降しか使ってはいけないか、に分岐する。
      > - B の使う数を列挙して分岐し、C 以降しか使ってはいけないに遷移する。
      > - 次の1個が、B, C, D, E, F... である場合に分岐する。
    - なるほど、[実装1](#実装1)、[実装2](#実装2)は1つ目のパターン。
  - https://discord.com/channels/1084280443945353267/1196472827457589338/1232733730280575017
    - > 答えの数ですが、candidates = [1..target] の場合、これは分割数というものですね。
    - https://oeis.org/A000041
    - 計算量の評価、複雑そうだったのでサボってしまった。

### 実装3

- > B の使う数を列挙して分岐し、C 以降しか使ってはいけないに遷移する。

```python3
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        descendings = sorted(candidates, reverse=True)
        results = []
        stack = [([], target, 0)]  # (combination, remain, index)
        while stack:
            combination, remain, index = stack.pop()

            if index == len(descendings):
                continue

            added = 0
            while added < remain:
                stack.append((combination, remain - added, index + 1))
                added += descendings[index]
                combination = combination + [descendings[index]]
            
            if added == remain:
                results.append(combination)
                continue
        
        return results
```

### 実装4

- > - 次の1個が、B, C, D, E, F... である場合に分岐する。

```python3
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        ascendings = sorted(candidates)
        results = []
        stack = [([], target, 0)]  # (combination, remain, index)
        while stack:
            combination, remain, index = stack.pop()

            for i in range(index, len(ascendings)):
                added = ascendings[i]
                if added >= remain:
                    break
                stack.append((combination + [added], remain - added, i))

            if added == remain:
                results.append(combination + [added])
                continue

        return results
```

### 実装5

- DP
- 外側を昇順にソートしたcandidates、内側をtotalで回す。
  - totalを自然にcandidateから始められる。
  - 小さい数字のみで作ったsum_to_combinationsを使って、大きい数字も上書きして構築できる。
    - 本当は2次元DPをやっているが、暗黙的に行なっている。

```python3
class Solution:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        ascendings = sorted(candidates)
        sum_to_combinations = [[] for _ in range(target + 1)]
        sum_to_combinations[0].append([])
        for candidate in ascendings:
            for total in range(candidate, target + 1):
                previous = total - candidate
                if not sum_to_combinations[previous]:
                    continue
                for combinations in sum_to_combinations[previous]:
                    sum_to_combinations[total].append(
                        combinations + [candidate]
                    )
        return sum_to_combinations[-1]
```

## Step 3

[実装4](#実装4)
