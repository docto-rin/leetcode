## Step 1

- 問題文
  - 整数の配列`nums`が与えられる。狭義単調増加する部分列で最長なもの（Longest Increasing Subsequence, LIS）の長さを返す。
  - 制約：
    - 1 <= nums.length <= 2500
    - -10^4 <= nums[i] <= 10^4

### 実装1

- アルゴリズムの選択
  - DPによる方法を思いつく。
    - `nums`を線形に走査し、次の要素`num`をこれまで作ってきた部分列のうち、一番長くなるものにくっつける。
    - なので、部分列の情報としては一番最後の要素と長さだけ覚えていればいい。
    - くっつけられるものがない場合は`num`から始まる新しい部分列を作る。
    - `nums`の走査が終わったら、一番長い部分列の長さを返す。
- 実装
  - 二重forループで良いだろう。
- 計算量
  - Time: O(n^2)
  - Space: O(n)

```python3
from typing import List
from operator import itemgetter


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        subsequences = []  # [(tail, length)]
        for num in nums:
            next_length = 1
            for tail, length in subsequences:
                if num > tail:
                    next_length = max(next_length, length + 1)
            subsequences.append((num, next_length))
        
        return max(subsequences, key=itemgetter(1))[1]
```

- ここまで15分ほど。
- > Follow up: Can you come up with an algorithm that runs in O(n log(n)) time complexity?
  - 内側のループに無駄がありそうだが思いつかない。GPT-5にヒントを乞う。
- > tails配列（“長さLの増加列の末尾最小値”を保持）＋二分探索への置換で O(n log n) を狙える。
  - あーなるほど。理解しました。

### 実装2

- 各長さの部分列の中で、最小な末尾のみを記録する。
- numsを線形走査し、それぞれで二分探索で一発で最も長さを確保できる接続先を見つけていく。
- 計算量
  - Time: O(nlogn)
  - Space: O(n)

```python3
from typing import List
from bisect import bisect_left


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        tails = []
        for num in nums:
            next_length = bisect_left(tails, num) + 1
            if next_length > len(tails):
                tails.append(num)
                continue
            tails[next_length - 1] = num
        
        return len(tails)
```

- 自分でfollow-up
  - LISそのものを返したい場合、tailsがそのままになるかと思ったが、誤り。
  - tailsはvalueの狭義単調増加列にはなるが、インデックスはそうとは限らないため、tailsを部分列とみなしたとき実現できるとは限らない。
  - 追加の情報を記録する必要がありそう。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.92aluhxkunm1)
  - [実装1](#実装1)でtailの値を保存する必要がなかったことに気づいた。
  - セグメント木なるものが使えるらしい。

### 実装3

- セグメント木を使ってみる。
  - 区間に対するクエリを高速で処理できるとのこと。

```python3
from typing import List
from bisect import bisect_left


class SegmentTreeMax:
    def __init__(self, size):
        n = 1
        while n < size:
            n *= 2
        self.base = n
        self.data = [0] * 2 * n
    
    def _to_leaf_index(self, index):
        return index + self.base - 1

    def update(self, index, value):
        """change point max"""
        k = self._to_leaf_index(index)
        self.data[k] = max(self.data[k], value)
        k //= 2
        while k >= 1:
            self.data[k] = max(self.data[2 * k], self.data[2 * k + 1])
            k //= 2

    def query(self, first, last):
        """get range max"""
        if first > last:
            return 0
        l = self._to_leaf_index(first)
        r = self._to_leaf_index(last)
        result = 0
        while l <= r:
            if l % 2 == 1:
                result = max(result, self.data[l])
                l += 1
            if r % 2 == 0:
                result = max(result, self.data[r])
                r -= 1
            l //= 2
            r //= 2
        return result


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        ascending = sorted(set(nums))
        def to_index(number):
            # 1-indexed
            return bisect_left(ascending, number) + 1
        
        seg_tree = SegmentTreeMax(size=len(ascending))

        max_length = 0
        for x in nums:
            index = to_index(x)
            # strictly increasing
            longest = seg_tree.query(1, index - 1)
            seg_tree.update(index, longest + 1)
            max_length = max(max_length, longest + 1)

        return max_length
```

### 実装4

- LISそのものも復元して返す。

```python3
from typing import List
from bisect import bisect_left


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            # return []
            return 0

        min_tails = []
        # min_tails[i] is minimum of tail of length i subsequences
        min_tails_index = []
        # min_tails_index[i] is the index of min_tails[i] in nums
        previous_index = [-1] * len(nums)
        # previous_index[i] is the index of previous of min_tails[i]

        for i, num in enumerate(nums):
            next_length = bisect_left(min_tails, num) + 1

            if next_length > len(min_tails):
                min_tails.append(num)
                min_tails_index.append(i)
            else:
                min_tails[next_length - 1] = num
                min_tails_index[next_length - 1] = i

            if next_length - 1 > 0:
                previous_index[i] = min_tails_index[next_length - 2]

        lis = []
        backing = min_tails_index[-1]
        while backing != -1:
            lis.append(nums[backing])
            backing = previous_index[backing]
        lis.reverse()
        # return lis
        return len(lis)
```

## Step 3

### 実装5

- ほぼ[実装2](#実装2)

```python3
from typing import List
from bisect import bisect_left


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        min_tails = []
        # min_tails[i] is minimum of tail of length i + 1 subsequences
        for num in nums:
            next_length = bisect_left(min_tails, num) + 1
            if next_length > len(min_tails):
                min_tails.append(num)
                continue
            min_tails[next_length - 1] = num
        return len(min_tails)
```
