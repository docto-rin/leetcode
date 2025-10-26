## Step 1

- 問題文
  - `n`本の支柱を持つフェンスがあり、それぞれの支柱は`k`色のいずれかで塗ることができる。
  - 連続する3本以上の支柱が同じ色にならないように、すべての支柱を塗る方法の総数を返せ。

### 実装1

- アルゴリズムの選択
  - 支柱の個数nを1から増やしていくような動的計画法（bottom-up）で解く。
  - 状態はk+1個あれば十分そうだが、これだとkが大きい時にかさむのと、本質的に異なる色を区別する意味がない（対等性が成り立つ）ので、削減したい。
  - 少し考えて、次にどの色も塗っていい状態と、（すでにある色が2本連続していて）特定の1色だけNGな状態の2つを考えれば良さそう。
- 実装
  - 長さ2のリストを使い、forループを使って状態（方法）の総数を更新していく。
- 計算量
  - Time: O(n)
  - Space: O(1)
- 所要時間

```python3
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        counter = [k, 0]
        for _ in range(1, n):
            any_color_is_ok = counter[0] * (k - 1) + counter[1] * (k - 1)
            one_color_is_ng = counter[0]
            counter = [any_color_is_ok, one_color_is_ng]
        return sum(counter)
```

- ここまで7分。
- 状態数が常に2なので、リストを使わなくてよかったかも。

## Step 2

- https://github.com/garunitule/coding_practice/pull/30/
  - 状態推移をコンパクトにして、3項間漸化式 $T_i = (k-1)(T_{i-1} + T_{i-2})$ で表せるとのこと。
    - $(k-1)T_{i-2}$ のイメージは、状態any_color_is_okを数えるために、2つ前に戻って直前と異なるk-1色で塗り直しているイメージ。
  - 再帰関数を使っての実装も可能。
    - @cacheをつけない場合、計算回数が指数爆発する。その指数の底は黄金数。
    - @cacheをつければO(n)。
    - https://docs.python.org/ja/3/library/functools.html#functools.cache
- https://github.com/shintaro1993/arai60/pull/34/
  - lrt_cacheの自前実装
- https://github.com/shining-ai/leetcode/pull/30/
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.leqr94ydg2be)
  - https://discord.com/channels/1084280443945353267/1201211204547383386/1220666008881336331
    - LRU Cache
      - https://docs.python.org/ja/3/library/functools.html#functools.lru_cache
      - 最も使われていない要素から削除したい -> Doubly Linked List: O(1)
      - すぐ順序を更新できる必要がある -> HashMap: O(1)
      - なので、Linked HashMapを使って実装するらしい。
      - https://github.com/python/cpython/blob/3.14/Lib/functools.py
  - https://discord.com/channels/1084280443945353267/1337642831824814192/1360598387388580055
    - > これ、OrderedDict の中身は Doubly-Linked List なので、まあ、練習としては、Doubly-Linked List 自体を書いて欲しいところではありますね。

### 実装2

- 3項間漸化式 $T_i = (k-1)(T_{i-1} + T_{i-2})$ をbottom-upに計算していく。

```python3
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n < 0 or k < 0:
            raise ValueError(f"n '{n}' and k '{k}' must be non-negative integers.")
        if k == 0:
            return 0
        if n == 0:
            return 1
        if n == 1:
            return k
        
        first_previous = k
        current = k * k
        for _ in range(3, n + 1):
            second_previous = first_previous
            first_previous = current
            current = (k - 1) * (first_previous + second_previous)
        return current
```

### 実装3

- 再帰関数 + @cache で実装。

```python3
from functools import cache


class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n < 0 or k < 0:
            raise ValueError(f"n '{n}' and k '{k}' must be non-negative integers.")
        if k == 0:
            return 0
        if n == 0:
            return 1
        
        @cache
        def helper(num_posts):
            if num_posts == 1:
                return k
            if num_posts == 2:
                return k * k
            
            return (k - 1) * (helper(num_posts - 1) + helper(num_posts - 2))
        
        return helper(n)
```

※一部のテストケースで再帰上限10**6にかかる。

### 実装4

- 漸化式を線形変換による行列表現に直し、冪乗法を適用すると、時間計算量がO(n) -> O(log n)になるとのこと。
- 実装が少々重いため、GPT-5の手助けを借りた。

総数を $T_i$ とすると、Paint Fence（同色3連続禁止）の漸化式は、

$$ T_i = (k-1)(T_{i-1} + T_{i-2}) $$

初期値は、

$$ T_1 = k,\quad T_2 = k^2 $$

これを2次の線形漸化式として行列表現に直すと、

```math
\begin{bmatrix} T_i \
T_{i-1} \end{bmatrix}
= \underbrace{\begin{bmatrix}
k-1 & k-1 \
1   & 0
\end{bmatrix}}_{M}
\begin{bmatrix} T_{i-1} \
T_{i-2} \end{bmatrix}
```

これを繰り返して、

```math
\begin{bmatrix} T_n \
T_{n-1} \end{bmatrix}
= M^{n-2}
\begin{bmatrix} T_2 \
T_1 \end{bmatrix}
= M^{n-2}
\begin{bmatrix} k^2 \
k \end{bmatrix}
```

で、答えはベクトルの1要素目 $T_n$ となる。

例えば n=15 の場合を考える。 $M^{13}$ を求めるとき、

- 普通なら、 M × M × M × ... × M と13回計算するが、
- 冪乗法では 以下のように指数を半分にしていく。

```
M^13 = M × (M^6)^2
M^6  = (M^3)^2
M^3  = M × (M^1)^2
M^1  = M
```

掛け算回数は約 log2(13) ≒ 4 回で済み、時間計算量が O(n) → O(log n) に改善する。

- 計算量
  - 行列積は定数時間（2x2）なので、べき乗は O(log n)。
  - 追加メモリは O(1)。

```python
from typing import List


class Solution:
    def num_ways_log(self, n: int, k: int) -> int:
        """Count colorings in O(log n) using matrix exponentiation.

        Recurrence:
            T_n = (k - 1) * (T_{n-1} + T_{n-2})
        Matrix form:
            [T_n, T_{n-1}]^T = M^{n-2} * [T_2, T_1]^T,
            where M = [[k-1, k-1], [1, 0]]

        Args:
            n: Number of fence posts (n >= 0).
            k: Number of colors (k >= 0).

        Returns:
            The number of valid colorings with no three adjacent posts
            painted the same color.

        Raises:
            ValueError: If n or k is negative.
        """
        if n < 0 or k < 0:
            raise ValueError(f"n '{n}' and k '{k}' must be non-negative integers.")
        if k == 0:
            return 0
        if n == 0:
            return 1
        if n == 1:
            return k
        if n == 2:
            return k * k

        m = [[k - 1, k - 1],
             [1, 0]]

        vec = [k * k, k]  # [T_2, T_1]
        m_pow = self._mat_pow(m, n - 2)
        t_n = self._mat_vec(m_pow, vec)[0]
        return t_n

    @staticmethod
    def _mat_mul(a: List[List[int]], b: List[List[int]]) -> List[List[int]]:
        """Multiply 2x2 matrices."""
        return [
            [a[0][0] * b[0][0] + a[0][1] * b[1][0],
             a[0][0] * b[0][1] + a[0][1] * b[1][1]],
            [a[1][0] * b[0][0] + a[1][1] * b[1][0],
             a[1][0] * b[0][1] + a[1][1] * b[1][1]],
        ]

    @staticmethod
    def _mat_vec(a: List[List[int]], v: List[int]) -> List[int]:
        """Multiply 2x2 matrix and 2x1 vector."""
        return [
            a[0][0] * v[0] + a[0][1] * v[1],
            a[1][0] * v[0] + a[1][1] * v[1],
        ]

    def _mat_pow(self, m: List[List[int]], p: int) -> List[List[int]]:
        """Fast exponentiation for 2x2 matrix."""
        # Identity matrix
        res = [[1, 0],
               [0, 1]]
        base = [row[:] for row in m]
        e = p
        while e > 0:
            if e & 1:
                res = self._mat_mul(res, base)
            base = self._mat_mul(base, base)
            e >>= 1
        return res
```

### 実装5

- LRU Cacheの自前実装。

```python3
from dataclasses import dataclass
from typing import Optional, Any, Dict, Callable
from functools import wraps


@dataclass
class ListNode:
    """Doubly linked list node."""

    key: Any
    val: Any
    prev: Optional["ListNode"] = None  # String quote for forward reference
    next: Optional["ListNode"] = None


class OrderedMap:
    """Order-preserving map with O(1) operations."""

    def __init__(self):
        self._map: Dict[Any, ListNode] = {}
        # Sentinel nodes: head <-> ... <-> tail
        self._head = ListNode(key=None, val=None)
        self._tail = ListNode(key=None, val=None)
        self._head.next = self._tail
        self._tail.prev = self._head

    def __getitem__(self, key: Any) -> Any:
        """Get value for key. Raises KeyError if not found."""
        if key not in self._map:
            raise KeyError(key)
        return self._map[key].val

    def __setitem__(self, key: Any, val: Any) -> None:
        """Set key-value pair, moving it to end."""
        if key in self._map:
            node = self._map[key]
            node.val = val
            self.move_to_tail(key)
        else:
            new_node = ListNode(key=key, val=val)
            self._add_to_tail(new_node)
            self._map[key] = new_node

    def __contains__(self, key: Any) -> bool:
        """Check if key exists."""
        return key in self._map

    def size(self) -> int:
        return len(self._map)

    def move_to_tail(self, key: Any) -> None:
        """Move key to tail (most recent position)."""
        if key in self._map:
            node = self._map[key]
            self._remove(node)
            self._add_to_tail(node)

    def pop_first(self) -> Optional[tuple]:
        """Remove and return (key, val) of first entry, or None."""
        first = self._head.next
        if first is None or first is self._tail:
            return None
        self._remove(first)
        del self._map[first.key]
        return (first.key, first.val)

    def _add_to_tail(self, node: ListNode) -> None:
        last = self._tail.prev
        assert last is not None
        node.prev = last
        node.next = self._tail
        last.next = node
        self._tail.prev = node

    def _remove(self, node: ListNode) -> None:
        prev_node = node.prev
        next_node = node.next
        assert prev_node is not None and next_node is not None
        prev_node.next = next_node
        next_node.prev = prev_node
        node.prev = None
        node.next = None


class LRUCache:
    """LRU cache with fixed capacity. All operations are O(1)."""

    def __init__(self, capacity: int):
        if capacity < 0:
            raise ValueError("capacity must be non-negative")
        self._capacity = capacity
        self._map = OrderedMap()

    def get(self, key: Any) -> Any:
        """Get value and mark as recently used. Raises KeyError if not found."""
        if key not in self._map:
            raise KeyError(key)
        self._map.move_to_tail(key)
        return self._map[key]

    def put(self, key: Any, value: Any) -> None:
        """Put key-value pair. Evicts LRU item if capacity exceeded."""
        if self._capacity == 0:
            return

        self._map[key] = value

        if self._map.size() > self._capacity:
            self._map.pop_first()

    def __contains__(self, key: Any) -> bool:
        """Check if key exists in cache."""
        return key in self._map


def lru_cache(capacity: int = 128) -> Callable:
    """Decorator to cache function results with LRU eviction."""

    def decorator(func: Callable) -> Callable:
        cache = LRUCache(capacity=capacity)

        @wraps(func)
        def wrapper(*args):
            if args in cache:
                return cache.get(args)

            result = func(*args)
            cache.put(args, result)
            return result

        return wrapper

    return decorator


if __name__ == "__main__":
    # Test basic LRU eviction
    cache = LRUCache(capacity=2)
    cache.put("a", 1)
    cache.put("b", 2)
    cache.put("c", 3)  # Evicts "a"
    try:
        cache.get("a")
        assert False, "Expected KeyError"
    except KeyError:
        pass
    assert cache.get("b") == 2

    # Test None value caching
    cache.put("null", None)
    assert cache.get("null") is None

    # Test decorator
    @lru_cache(capacity=10)
    def fib(n):
        return n if n < 2 else fib(n - 1) + fib(n - 2)

    assert fib(10) == 55
    print("All tests passed")
```

- 限界
  - デコレータがキーワード引数に対応していない。
  - capacity=0は早期リターンしていますが、functoolsの実装ではデコレーターでは常に関数を実行し、キャッシュはしないという挙動をする。
  - デコレーターでunhashableな引数（list, dict など）を渡すとエラーになる。
  - スレッドセーフでない。

## Step 3

### 実装6

- ほぼ[実装2](#実装2)

```python3
class Solution:
    def num_ways(self, n: int, k: int) -> int:
        if n < 0:
            raise ValueError(f"n '{n}' must be non-negative ingetger")
        if k < 0:
            raise ValueError(f"k '{k}' must be non-negative ingetger")
        if k == 0:
            return 0
        if n == 0:
            return 1
        if n == 1:
            return k
        
        first_previous = k
        current = k * k
        for _ in range(3, n + 1):
            second_previous = first_previous
            first_previous = current
            current = (k - 1) * (first_previous + second_previous)
        return current
```
