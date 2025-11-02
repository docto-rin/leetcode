## Step 1

- 問題文
  - 異なる整数の配列`nums`が与えられ、可能なすべての順列を返せ。
  - 解答はどんな順番で返してもよい。
  - 制約：
    - 1 <= nums.length <= 6
    - -10 <= nums[i] <= 10
    - All the integers of nums are unique.

### 実装1

- アルゴリズムの選択
  - `list(itertools.permutations(nums))`と書けば終わるが、この実装の中身を書くつもりでやる。
  - divide and conquer（top-down）による解決ができそう。
- 実装
  - top-downは再帰呼び出しの結果を使って返り値が決まるので、loopで書くと難しい。再帰関数で書く。
  - nums.lengthが比較的やさしく、配列の中身もただの整数で軽いため、ここではスライスを使うことにする。

```python3
class Solution:
    def permute(self, nums: list[int]) -> list[list[int]]:
        if len(nums) == 1:
            return [[nums[0]]]
        
        permutations = []
        for i in range(len(nums)):
            child_permutations = self.permute(nums[:i] + nums[i + 1 :])
            for child in child_permutations:
                permutations.append([nums[i]] + child)
        return permutations
```

- 15分かかった。
- スライスを多用している（毎階層で新しいリストを作る）のが明らかに良くなさそう。
  - 階層間でのデータの重複を減らしたい。
  - 計算量を正しくイメージできていなかった：
    > - Time Complexity: O(n * n!)
    > - Space Complexity: O(n * n!)
- 途中まで append() の中身を [nums[i]].extend(child) と書いて詰まっていた。
  - list.extend(), list.append(), などのクラスメソッド系は基本in-placeで、それがわかるよう意図的に返り値がNoneにされている。
  - 知識として知っていたのに、書くときに意識できておらず誤用していた。反省。
- permutations という変数名が、itertools.permutationsと被るので良くない。

## Step 2

- GPT-5
  - > スライスコピーのオーバーヘッドがのり、実測ではかなり重くなります。
    > - 改善案1: スワップによるインプレースのバックトラック
    > - 改善案2: 使用フラグとパスを使うバックトラック
    - バックトラックを知らなかった。空間計算量的に効率的な実装案を提示された。
      - 要は、経路を記録しながらやるDFSと捉えた。
    - fyi: radditにbacktrackingのおすすめの教材として以下が挙げられていた：
      - https://leetcode.com/problems/permutations/solutions/18239/a-general-approach-to-backtracking-questions-in-java-subsets-permutations-combination-sum-palindrome-partioning/
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.xe8lirtynkse)
  - スタックを使ってかける。
  - https://discord.com/channels/1084280443945353267/1237649827240742942/1347858547022495775
    - > ここで nums で回すと計算量が少し悪くなりますが、そもそも重いので状況次第ですね。
      - 各再帰階層で `nums` 全体を走査して未使用かを毎回判定しているため、残り候補が `n - depth` 個しかないのに、毎回 `n` 回ぶんループしてしまう無駄がある。「未使用集合だけ」をループすれば、その無駄を削れる。
    - > ただ、変更しながらループで回すことはできないので、コピーすることになります。それでも、最後の n はなくなります。
      - ミューテーション中の反復を避けるために反復用にコピーを取っておく。
      - 以前、「`for x in ls:`は最初に一度だけイテレータ`iter(ls)`を作る」と学習したが、反復中の`ls`へのミューテーションが反映されることは知らなかった。
      - https://docs.python.org/ja/3/tutorial/controlflow.html#for-statements
        > コレクションオブジェクトの値を反復処理をしているときに、そのコレクションオブジェクトを変更するコードは理解するのが面倒になり得ます。 そうするよりも、コレクションオブジェクトのコピーに対して反復処理をするか、新しいコレクションオブジェクトを作成する方が通常は理解しやすいです
      - GPT-5による要点まとめ
        > - `for ... in ...` の右側は最初に一度だけ評価され、イテレータが作られる。
        > - そのイテレータは元リストをライブ参照するため、ループ中のミューテーションは挙動に反映される。
        > - リストはエラーにならないが、辞書や集合は反復中にサイズ変更すると `RuntimeError: dictionary/set changed size during iteration` になります。
        > 
        > 実務ではミューテーションを伴う同一リストの反復は避け、必要ならスナップショットで回すか、`while ls: ... ls.pop()` の形にする方が安全です。

### 実装2

- Backtracking DFSを再帰関数を使って実装。
- 計算量
  - Time: O(n * n!)
    - 順列のパターンがn!で、それぞれに対しpath.copyでO(n)なので。
  - Space: O(n)

```python3
class Solution:
    def permute(self, nums: list[int]) -> list[list[int]]:
        all_patterns = []
        path = []
        remainings = set(nums)
        
        def traverse_remainings():
            if not remainings:
                all_patterns.append(path.copy())
                return
            for remain in list(remainings):
                path.append(remain)
                remainings.remove(remain)
                traverse_remainings()
                path.pop()
                remainings.add(remain)
        
        traverse_remainings()
        return all_patterns
```

- GPT-5からのコメント
  > - ただし `set` を使っているため、重複要素が含まれる入力では誤ります。
  > - `set` は順序を持たないため、出力順は実行ごとに変わり得ます。順序の再現性が必要なら、`nums` のインデックスで回して `used` フラグにする方法が無難です。

### 実装3

- スタックで書く方法
- 計算量
  - Time: O(n * n!)
  - Space: O(n)

```python3
class Solution:
    def permute(self, nums: list[int]) -> list[list[int]]:
        if not nums:
            return [[]]

        results = []
        stack = [[]]
        set_nums = set(nums)

        while stack:
            path = stack.pop()
            if len(path) == len(nums):
                results.append(path)
                continue

            for num in set_nums - set(path):
                stack.append(path + [num])

        return results
```

- 所感
  - [実装2](#実装2)でGPT-5がコメントしたように、順序の指定がある場合はforループは`nums`で回すのが安全そう。

## Step 3

[実装2](#実装2)
