## Step 1

最初に、試験の点数を表す整数の配列`nums`と、自然数`k`が与えられる。
その後、試験の点数`val`が1つずつ追加されていくたびに、`k`番目に高い点数を返す問題。

- アルゴリズムの選択
  - ナイーブには、valが追加されるたびにnumsを降順にソートし直し、k番目をインデックスアクセスすればいいが、明らかに無駄が多く遅い。
    - ソートは一回ごとにO(nlogn)の時間計算量がかかる。ソートを毎回やるとaddの回数をmとして、O(mnlogn)。
  - 冷静に考えると、追加する値は1個なので、二分探索で挿入位置を求めて挿入すれば速い。
    - 二分探索はO(logn)、挿入はO(n)なので、1回あたりO(n)、全部でO(mn)。
  - しかし、これ以上に良い案が浮かばないので、heapとpriority queueについて勉強することにした。
    - アルゴリズムイントロダクションのheapの章、またpythonの[heapqモジュール](https://docs.python.org/ja/3/library/heapq.html)を参照した。
    - "ほぼ完全二分木"が本質で、max-heapなら親.val > 子.valが常に成り立つ。
  - 上位k個の点数をmin-heapで保持し、addの度にheapの根を返せば良さそう。
    - 追加されたvalがheapの根より大きい場合は、先にheapにvalをinsertし、1回popする。
    - m回の追加の時間計算量はO(mlogk)で、ここまで思いついた中では最善。
    - 初期化も含めるとO(nlogn + mlogk)

実装します。

- 時間計算量: O(nlogn + mlogk)
- 空間計算量: O(n)

```python3
import heapq

class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.k = k
        self.topk_nums = sorted(nums, reverse=True)[:k]
        heapq.heapify(self.topk_nums) # min-heap

    def add(self, val: int) -> int:
        if len(self.topk_nums) < self.k:
            heapq.heappush(self.topk_nums, val)
            if len(self.topk_nums) == self.k:
                return self.topk_nums[0]
            return None
        
        if val > self.topk_nums[0]:
            heapq.heappushpop(self.topk_nums, val)
        return self.topk_nums[0]

# Your KthLargest object will be instantiated and called as such:
# obj = KthLargest(k, nums)
# param_1 = obj.add(val)
```

## Step 2

GPT-5に先にレビューしてもらったところ、

> 参考までに、初期構築を最適化するなら
> * 先に K 個をヒープ化 **O(K)**
> * 残り N−K 個を順に `heappushpop` でふるいにかける **O((N−K) log K)** で、初期化を **O(N + (N−K) log K)** まで落とせます。スペースはどちらも **O(K)** です。

とのことだったので、これを実装してみます。

- 時間計算量: O((n+m)logk)
- 空間計算量: O(k)

```python3
class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.k = k
        topk_nums = nums[:k]
        heapq.heapify(topk_nums) # min-heap
        for num in nums[k:]:
            heapq.heappushpop(topk_nums, num)
        self.topk_nums = topk_nums
```

heapを使うなら徹底的にソートは避けるべきということだろう。

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.t2w1qof72ib2)
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1203365833099972628
    > 単に init で add を呼んでしまったほうが見通しがいいです。
    - コンストラクタで自前メソッドを使うの、アハ体験ですね。しかしよく考えると至極当然。
  - https://github.com/rinost081/LeetCode/pull/9/files#r1875304658
    > 豆知識ですが、3.11からpowersortが使われているみたいですね。
    > https://www.i-programmer.info/news/216-python/15954-python-now-uses-powersort.html
  - https://discord.com/channels/1084280443945353267/1297920116025065533/1331562782704930857
    > k 個取っておくようにすれば間に合いましたね。sorted array は意外と速いです。insort も参考にどうぞ。
    > https://docs.python.org/3/library/bisect.html#bisect.insort_right
    - ソート済み配列に対する適切な位置への挿入は標準で実装済みだそうです。

- あとは変数の命名について、self.topk_numsのtopkの意味がわかりにくいので、self.k_kargest_numsとかにします。
- 余力ができたら[cpython実装](https://github.com/python/cpython/blob/3.13/Lib/heapq.py)をみたり、簡単な再現実装をしたい。
  - https://github.com/nanae772/leetcode-arai60/pull/9
    - 取り組まれていてすごい。

## Step 3

```python3
import heapq

class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.k = k
        self.k_largest_nums = []

        for num in nums:
            self.add(num)
        
    def add(self, val: int) -> int:
        if len(self.k_largest_nums) < self.k:
            heapq.heappush(self.k_largest_nums, val)
            if len(self.k_largest_nums) == self.k:
                return self.k_largest_nums[0]
            return None
        
        if val > self.k_largest_nums[0]:
            heapq.heappushpop(self.k_largest_nums, val)
        return self.k_largest_nums[0]
```
