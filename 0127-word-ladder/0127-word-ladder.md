## Step 1

- 問題文
  - 単語`beginWord`から単語`endWord`へ、辞書`wordList`を用いて変換していくときのシーケンスの最小の長さを返す。
- 制約
  - 1 <= beginWord.length <= 10
  - endWord.length == beginWord.length
  - 1 <= wordList.length <= 5000
  - wordList[i].length == beginWord.length
  - beginWord, endWord, and wordList[i] consist of lowercase English letters.
  - beginWord != endWord
  - All the words in wordList are unique.
  - beginWordはwordListになくても問題ない。endWordがwordListになければ到達不可能とみなし0を返すべき。
  - 今後の説明のため、N = wordList.length, L = wordList[i].length とおく。
- アルゴリズムの選択
  - "adjacent pair of words differs by a single letter."となる2つの単語を辺で結んで無向グラフが作れる。
  - beginWordからendWordへの無向グラフ上での最短距離を求める問題とみなせる。
  - 後者の無向グラフ上での最短距離を求めるのはBFSで簡単にでき、計算量もO(E + V) = O(N * L + N) = O(N * L)で済む。
  - 前者が問題で、二重ループを回した上で、1文字ずつ比較する必要があるので計算量がO(N^2 * L)になってしまう。ボトルネックはこちら。
- 実装
  - BFSは辞書のインデックス（int）をkeyとして実装することで省メモリを狙う。

### 実装1

- 時間計算量: O(N^2 * L)
- 空間計算量: O(N^2)

```python3
from collections import defaultdict, deque
from copy import deepcopy


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_linked(word1, word2):
            num_different = 0
            for char1, char2 in zip(word1, word2):
                if char1 != char2:
                    num_different += 1
            return num_different == 1

        all_words = deepcopy(wordList)
        if beginWord in all_words:
            all_words.remove(beginWord)
        if endWord in all_words:
            all_words.remove(endWord)
        else:
            return 0
        all_words.append(beginWord)
        all_words.append(endWord)
        num_words = len(all_words)

        adjacent_list = defaultdict(list)  # index: [index1, index2, ...]
        for i in range(num_words):
            for j in range(i):
                if is_linked(all_words[i], all_words[j]):
                    adjacent_list[i].append(j)
                    adjacent_list[j].append(i)

        frontiers = deque()
        visited = set()
        frontiers.append((num_words - 2, 1))  # (index, sequence_count)
        visited.add(num_words - 2)
        while frontiers:
            frontier, count = frontiers.popleft()
            if frontier == num_words - 1:
                return count
            for neighbor in adjacent_list[frontier]:
                if neighbor not in visited:
                    frontiers.append((neighbor, count + 1))
                    visited.add(neighbor)
        return 0
```

Runtime: 9255 ms
Memroy: 19.60 MB

やはり非常に遅い。

is_linked はnum_differentが2以上になったらreturn Falseした方が良さそう。

GPT-5に改善案を尋ねた。

> 1. **パターン索引法**（例：`h*t` みたいに 1 文字を `*` に置換した L 個のキーでハッシュ→平均 **O(N·L)** で辺を引ける）
> 2. **双方向 BFS**（begin 側と end 側から同時に広げて分岐を半減）
> 
> まずは (1) だけでも十分速くなります。さらに (1)+(2) で体感がっつり速くなります。

#### 1. パターン索引法

```python3
# written by GPT-5
def build_pattern_to_indices(all_words):
    pattern_to_indices = defaultdict(list)

    for index, word in enumerate(all_words):
        for i in range(len(all_words[0])):
            pattern = word[:i] + "*" + word[i + 1 :]
            pattern_to_indices[pattern].append(index)

    return pattern_to_indices
```

このようにして、各パターンごとに、value内でdenseに辺を引いていけば、O(N * L)でadjacent_listも構築できる。

また、実装を少し変えればpattern_mapから直接neighborsを取得できる。

#### 2. 双方向BFS

関数build_pattern_to_indicesは上記のとおり定義済みとする。

```python3
# written by GPT-5
def build_ladder(all_words, start_index, end_index):
    if start_index == end_index:
        return 1

    word_length = len(all_words[0])
    pattern_to_indices = build_pattern_to_indices(all_words)

    distance_from_start = {start_index: 1}
    distance_from_end = {end_index: 1}
    frontier_from_start = {start_index}
    frontier_from_end = {end_index}

    while frontier_from_start and frontier_from_end:
        # 常に小さい方のfrontierを拡張する
        if len(frontier_from_start) > len(frontier_from_end):
            frontier_from_start, frontier_from_end = (
                frontier_from_end,
                frontier_from_start,
            )
            distance_from_start, distance_from_end = (
                distance_from_end,
                distance_from_start,
            )

        next_frontier = set()

        for current_index in frontier_from_start:
            current_word = all_words[current_index]

            for i in range(word_length):
                pattern = current_word[:i] + "*" + current_word[i + 1 :]
                neighbor_indices = pattern_to_indices.get(pattern, [])

                for neighbor_index in neighbor_indices:
                    if neighbor_index in distance_from_end:
                        return (
                            distance_from_start[current_index]
                            + distance_from_end[neighbor_index]
                        )

                    if neighbor_index not in distance_from_start:
                        distance_from_start[neighbor_index] = (
                            distance_from_start[current_index] + 1
                        )
                        next_frontier.add(neighbor_index)

                # 枝刈り
                pattern_to_indices[pattern] = []

        frontier_from_start = next_frontier

    return 0
```

感想
- while frontier_from_start and frontier_from_end:の内側に、もう一つforループを追加することで、layer-wiseに探索できる。
- なので、deque()のようなFIFO構造は不要になる。むしろsetを使っているのでwordの重複探索を防ぐことすらできる。
- visited的な長期記憶はdistance_from_startなどのsetが一緒に行っている。
- 一度使ったpattern経由の辺を枝刈りしている。visitedとは別にコミュニティ単位で潰していくイメージ。（コミュニティ内はdenseに接続しているので、1回使用したら不要になる）

### 実装2

- 時間計算量: O(N * L)
- 空間計算量: O(N * L)

関数build_ladderは上記のとおり定義済みとする。

```python3
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        all_words = deepcopy(wordList)
        if beginWord in all_words:
            all_words.remove(beginWord)
        if endWord in all_words:
            all_words.remove(endWord)
        else:
            return 0
        all_words.append(beginWord)
        all_words.append(endWord)

        return build_ladder(all_words, len(all_words) - 2, len(all_words) - 1)
```

次の実装に向けた改善案：
- beginWordがwordListに最初から含まれている可能性はあるが、もし重複してしまってキューに2回入ってもsetにより探索の重複は起きない。
- int参照の方が多少軽量かもしれないが、ボトルネックにはなってないのであまり意味はなく、可読性のためにそのまま文字列でいいだろう。
- 早期returnはなるべく最初にやっておく。

## Step 2

- コメント集
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1216123084889788486
    - > https://cs.stackexchange.com/questions/93467/data-structure-or-algorithm-for-quickly-finding-differences-between-strings
      >
      > 頭から半分または尻尾から半分が一致しているはずなので、それでバケットを作ってバケット内でのみ比較すればいいというやりかたもありますね。(編集距離が1であるかの確認に、頭から何文字一致していて、尻尾から何文字一致しているかを足してやればいいという方法をどっかで使ったことあります。)
    - 最悪時間計算量がO(nklogk)な簡易的な方法とのこと。
  - https://discord.com/channels/1084280443945353267/1303605021597761649/1306631474065309728
    - pattern_to_wordsをクラス化し、複雑な構築手順を低レベルなmethodに分解されていて、非常にわかりやすい。
  - https://github.com/shining-ai/leetcode/pull/20/files#r1517033572
    - 文字列を2箇所以上で結合するときは、二項演算子+よりf-stringの方がパフォーマンスが優れており、Google Style Guideで推奨されているとのこと。
    - 意識していなかったので、f-stringで習慣づけるようにしたい。
  - https://discord.com/channels/1084280443945353267/1295357747545505833/1309222881330335816
    - > "*" が来ると動かなくなるのが気になり、Python の場合は、タプルも dict の Key にできるのでそれも一つかなと思います。
    - 確かに、keyに視認性みたいなものが求められない場面では無理に結合せず、タプルで持っておくのが安全そう。（前にも[メールアドレスの問題](https://leetcode.com/problems/unique-email-addresses/)で同様の件があった）
  - https://discord.com/channels/1084280443945353267/1295324533963751465/1349338344580055102
    - pattern_to_wordsからneighborsを取得するロジックは、BFSのロジックの外に追いやりたい＋可能ならmethodを分割したい。

## Step 3

### 実装3

patter_to_words方式 + 単方向BFS を書く。双方向BFSは理想形と捉え、現実的に書きやすいコードをやる。

```python3
from typing import List
from collections import defaultdict


class WordNeighbors:
    def __init__(self):
        self.pattern_to_words = defaultdict(list)
    
    def _get_patterns(self, word):
        for i in range(len(word)):
            yield (word[:i], word[i+1:])
    
    def add(self, word):
        for pattern in self._get_patterns(word):
            self.pattern_to_words[pattern].append(word)
        
    def get_neighbors_with_pruning(self, word):
        for pattern in self._get_patterns(word):
            for neighbor in self.pattern_to_words[pattern]:
                if neighbor == word:
                    continue
                yield neighbor
            self.pattern_to_words[pattern] = []


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        if beginWord == endWord:
            return 1
        if endWord not in wordList:
            return 0
        
        word_neighbors = WordNeighbors()
        for word in [beginWord] + wordList:
            word_neighbors.add(word)
        
        frontiers = set()
        frontiers.add(beginWord)
        visited = set()
        visited.add(beginWord)
        count = 0
        while frontiers:
            next_frontiers = set()
            count += 1
            
            for frontier in frontiers:
                for neighbor in word_neighbors.get_neighbors_with_pruning(frontier):
                    if neighbor == endWord:
                        return count + 1
                    if neighbor in visited:
                        continue
                    next_frontiers.add(neighbor)
                    visited.add(neighbor)
            
            frontiers = next_frontiers
        return 0
```

## Step 4

復習として後日やり直した。

### 実装4

pattern_to_words方式 + 双方向BFS

```python3
from collections import defaultdict


class WordNeighbors:
    def __init__(self):
        self.pattern_to_words = defaultdict(list)
    
    def _get_patterns(self, word):
        for i in range(len(word)):
            yield (word[:i], word[i + 1 :])
    
    def register(self, word):
        for pattern in self._get_patterns(word):
            self.pattern_to_words[pattern].append(word)
    
    def get_neighbors(self, word):
        for pattern in self._get_patterns(word):
            for neighbor in self.pattern_to_words[pattern]:
                if neighbor == word:
                    continue
                yield neighbor


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        if endWord not in wordList:
            return 0
        
        word_neighbors = WordNeighbors()
        for word in [beginWord] + wordList:
            word_neighbors.register(word)
        
        progress = 1
        frontiers_from_start = [beginWord]
        frontiers_from_end = [endWord]
        visited_from_start = {beginWord}
        visited_from_end = {endWord}

        while frontiers_from_start and frontiers_from_end:
            if len(frontiers_from_start) > len(frontiers_from_end):
                frontiers_from_start, frontiers_from_end = (
                    frontiers_from_end,
                    frontiers_from_start
                )
                visited_from_start, visited_from_end = (
                    visited_from_end,
                    visited_from_start
                )
            
            next_frontiers = []
            for word in frontiers_from_start:
                for neighbor in word_neighbors.get_neighbors(word):
                    if neighbor in visited_from_end:
                        return progress + 1
                    if neighbor in visited_from_start:
                        continue
                    next_frontiers.append(neighbor)
                    visited_from_start.add(neighbor)
            frontiers_from_start = next_frontiers
            progress += 1

        return 0
```
