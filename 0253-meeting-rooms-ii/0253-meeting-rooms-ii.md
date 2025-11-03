## Step 1

- 問題文
  - Given an array of meeting time interval objects consisting of start and end times [[start_1,end_1],[start_2,end_2],...] (start_i < end_i), find the minimum number of days required to schedule all meetings without any conflicts.
  - 制約：
    - 0 <= intervals.length <= 500
    - 0 <= intervals[i].start < intervals[i].end <= 10^6

### 実装1

- アルゴリズムの選択
  - スケジュールの問題は終了時刻の順に考えるとgreedyに解きやすい？という前提知識があった。
    - ひとまずintervalsはendでソートし、それを前提に考える。
    - 既存の部屋だけで処理できるか、新しい部屋がないと予約不可かを、各部屋の最後のmeetingの終了時刻とキューの開始時刻から判定できる。
      - 既存の部屋に予約を追加する場合は、なるべく終了時刻の遅い部屋に入れた方が効率的。
        - なぜならソートの効果でキューは既存のどれよりも終了時刻が遅いため。
- 実装
  - forループで書く。
- 計算量
  - Time: O(n^2logn)
    - n回のループのたびにroomsをソートO(nlogn)してるため。
  - Space: O(n)

```python3
from lintcode import Interval

"""
Definition of Interval:
class Interval(object):
    def __init__(self, start, end):
        self.start = start
        self.end = end
"""

class Solution:
    """
    @param intervals: an array of meeting time intervals
    @return: the minimum number of conference rooms required
    """
    def min_meeting_rooms(self, intervals: list[Interval]) -> int:
        ordered_intervals = sorted(intervals, key=lambda x: (x.end, x.start))
        rooms = []
        
        def reserve_available_room(interval, rooms):
            for room in rooms:
                if interval.start >= room[-1].end:
                    room.append(interval)
                    return
            rooms.append([interval])
        
        for interval in ordered_intervals:
            reserve_available_room(interval, rooms)
            rooms.sort(key=lambda x: x[-1].end, reverse=True)
        
        return len(rooms)
```

- 30分くらいかかった。
- だいぶ無駄が多いので次の[実装2](#実装2)でリファクタする。

### 実装2

- [実装1](#実装1)をリファクタした。
  - roomsは、その部屋の最後のミーティングの終了時刻だけ覚えていれば必要十分。
  - 次に予約する部屋は、なるべく終了時刻が遅いとこに入れたいが、二分探索できる。
  - roomsはソートされている必要があるが、毎回ソートし直さなくてもpopとappendを使って順序を維持できる。
  - ordered_intervals = sorted(intervals, key=lambda x: (x.end, x.start))だが、x.startは不要だった。
    - endでちゃんとソートされていれば、それぞれのstartに対し最も無駄のない部屋の割り当てはreserve_available_roomがやってくれる。
- 計算量
  - Time: O(nlogn + n^2) = O(n^2)
  - Space: O(n)

```python3
import bisect

from lintcode import Interval


class Solution:
    def min_meeting_rooms(self, intervals: list[Interval]) -> int:
        ordered_intervals = sorted(intervals, key=lambda x: x.end)
        rooms_end = []  # rooms_end[i] is room ith's last meetings's end time
        
        for interval in ordered_intervals:
            room_to_reserve = bisect.bisect_right(rooms_end, interval.start) - 1
            if room_to_reserve >= 0:
                rooms_end.pop(room_to_reserve)
            rooms_end.append(interval.end)
        
        return len(rooms_end)
```

## Step 2

- GPT-5によるレビュー。
  - > - 「必要な部屋数」≒「同時並行数の最大値」です。掃き出し法や最小ヒープが定石です。
    > - 開始時刻でソートし、最も早く空く部屋の終了時刻だけをヒープで管理します。`heap[0] <= start` なら再利用、そうでなければ新規部屋。
    - 最小ヒープ -> [実装3](#実装3)、掃き出し法（ダブルポインタ、sliding window）-> [実装4](#実装4)
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.c86y3qdg326e)
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1357410043854590157
    - > まあ、これでもいいんですが、座標圧縮と同等のことが対応表を作らなくてもできますよね。たとえば、(start, +1), (end, -1) からなる集合を作ってソートするなど。
      - 部屋の使用数の増減（開始 or 終了）する時刻と増減値を記録し、時刻でソートして、累積和を取り、maxすればいい。
      - ダブルポインタ（[実装4](#実装4)）を2-passでやっているのと同じ。
- https://github.com/nittoco/leetcode/pull/45/
  - この方も増減を追ってる。
- 割とダブルポインタまたは増減を追う方法に行き着いている人が多そうで、焦りを感じた。
  - 最初にアルゴリズムを考えるとき、もっと広く柔軟に行きたかった。
  - あるいは、予約を受付する人の気持ちになったらよかった。

### 実装3

- min-heap
- 計算量
  - Time: O(nlogn + nlogn) = O(nlogn)
  - Space: O(n)

```python3
import heapq

from lintcode import Interval


class Solution:
    def min_meeting_rooms(self, intervals: list[Interval]) -> int:
        start_order = sorted(intervals, key=lambda x: x.start)
        min_heap = []  # rooms' end time

        for interval in start_order:
            if min_heap and min_heap[0] <= interval.start:
                heapq.heapreplace(min_heap, interval.end)
            else:
                heapq.heappush(min_heap, interval.end)

        return len(min_heap)
```

- ソートはstartでやって、heapはendの小さい順、というのがミソだった。
- startが早い順にドローするので、最も早く空く部屋が無理なら破綻、と考えやすい。
- 逆に、それよりあとは開始時刻がもっと遅い予約しか来ないので、最も早く空く部屋を使っていい。
- これが真の貪欲法だった。
  - 自分の方法は、キューの順が終了時刻の速い順なので、どの部屋を使うかは自明じゃない。（可能な限り空くのが遅い部屋を使う）

### 実装4

- 掃き出し法（ダブルポインタ、sliding window）
- 計算量
  - Time: O(nlogn + n) = O(nlogn)
  - Space: O(n)

```python3
from lintcode import Interval


class Solution:
    def min_meeting_rooms(self, intervals: list[Interval]) -> int:
        starts = sorted(interval.start for interval in intervals)
        ends = sorted(interval.end for interval in intervals)

        used = max_used = 0  # num used rooms
        i = 0  # index of meeting that starts next
        j = 0  # index of meeting that ends next
        n = len(intervals)

        while i < n:
            if starts[i] < ends[j]:
                used += 1
                max_used = max(max_used, used)
                i += 1
            else:
                used -= 1
                j += 1

        return max_used
```

- 時系列に沿って進んでいる感じがとても素直。
  - 開始 or 終了のイベントが発生するまでwhileで一気に進んでいる感じ。
  - 終了条件は、すべてのmeetingが開始したらもう部屋を追加で用意することはない、ということ。

## Step 3

[実装4](#実装4)
