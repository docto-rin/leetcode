## Step 1

### 実装1

- 時間計算量: O(n)
- 空間計算量: O(n)

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        set1 = set(nums1)
        set2 = set(nums2)
        intersections = []
        for number in set1:
            if number in set2:
                intersections.append(number)
        return intersections
```

ここまで2分。

### 実装2

定数倍の最適化になるはず。

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        set1 = set(nums1)
        set2 = set(nums2)
        if len(set1) > len(set2):
            set1, set2 = set2, set1
        intersections = []
        for number in set1:
            if number in set2:
                intersections.append(number)
        return intersections
```

## Step 2

### 実装3

- http://docs.python.org/ja/3/tutorial/datastructures.html#sets
  - 二項演算子&は引数がsetのときは積集合を返すようにオーバーロードされている。

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))
```

### 実装4

- https://docs.python.org/3/library/stdtypes.html#frozenset.intersection
  - さらに、intersectionメソッドなるものもある。

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1).intersection(nums2))
```

### 実装5

- https://discord.com/channels/1084280443945353267/1303257587742933024/1319664094235594824
  - > これは、まあ、これでいいんですが、もう少し書き方にバリエーションがあるように思います。挙げてみますか?
    > たとえば、追加質問で考えられるのは、「片方がとても大きくて、片方がとても小さいときには、大きい方を set にするのは大変じゃないでしょうか、特に大きいほうが sort 済みのときにはどうしますか。」とかです。
  - この場合、これまでの実装ではO(n1 + n2) ≒ O(n1)かかる。（n1 = nums1.length, n2 = nums2.length とおいた）
  - for number in set(nums2) とし、numberをクエリにnums1に対し逐次二分探索した方が速くなるケースは発生しうる。O(n2 log n1)。

```python
import bisect

class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1.sort()
        intersections = []
        for number in set(nums2):
            position = bisect.bisect_left(nums1, number)
            if position < len(nums1) and nums1[position] == number:
                intersections.append(number)
        return intersections
```

<details>
  <summary>ベンチマークをGPT-5にとってもらった</summary>

  - 測定条件：
    * `nums1` は昇順リスト `[0, 1, ..., n1-1]`
    * `nums2` は `[0, 2n1]` の一様乱数（重複あり）
    * 集合方式は両方の set 構築コスト込み
    * 二分探索方式は `set(nums2)` で重複排除後に `bisect_left` で探索
  - <img width="80%" height="80%" alt="benchmark1" src="https://github.com/user-attachments/assets/fc62da3c-0561-4da7-8601-86cee71deeb9" />
  - n2/n1が小さい領域は思っていた以上に差があった
</details>

### 実装6

- https://discord.com/channels/1084280443945353267/1303257587742933024/1320000574434971749
  - > 他、両方ソートされていてとても大きければ、マージソートの変形のように書くと思います。

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1.sort()
        nums2.sort()
        intersections = []

        i = j = 0
        while i < len(nums1) and j < len(nums2):
            if nums1[i] < nums2[j]:
                i += 1
                continue
            if nums1[i] > nums2[j]:
                j += 1
                continue
            if intersections and nums1[i] == intersections[-1]:
                i += 1
                j += 1
                continue
            intersections.append(nums1[i])
            i += 1
            j += 1

        return intersections
```

- sorted linked listなどにも適用可能な方法。

<details>
  <summary>ベンチマークをGPT-5にとってもらった</summary>

  - 測定条件：入力は事前にソート済み。
  - <img width="80%" height="80%" alt="benchmark2" src="https://github.com/user-attachments/assets/a5ea7e5d-fd67-4b23-afe6-496b43f5db99" />
  - <img width="80%" height="80%" alt="benchmark3" src="https://github.com/user-attachments/assets/5d8af428-5ae8-42ef-818c-062c44963934" />
  - どちらも大きい状況では、マージソート風の方法がかなり速い。
  - set（ハッシュテーブル）構築の定数倍の重さを痛感した。

</details>

## Step 3

```python3
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))
```
