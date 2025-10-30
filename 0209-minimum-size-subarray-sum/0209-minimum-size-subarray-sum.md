## Step 1

- 問題文
  - 正整数の配列`nums`と正整数`target`が与えられる。
  - 和が`target`以上になるような部分配列（subarray）の長さの最小値を返せ。
  - もしそのようなsubarrayがなければ0を返せ。
  - 制約：
    - 1 <= target <= 10^9
    - 1 <= nums.length <= 10^5
    - 1 <= nums[i] <= 10^4

### 実装1

- アルゴリズムの選択
  - subarrayは連続している必要があるので、これもgreedyに解けたら嬉しい。
  - ひとまず、和がtarget以上になるまでsubarrayの右端を広げていき、target以上になったら左端を縮めていく。
- 実装
  - for-while、で書くと良さそう。
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
class Solution:
    def minSubArrayLen(self, target: int, nums: list[int]) -> int:
        INF = len(nums) + 1
        min_length = INF
        left = 0
        subarray_sum = 0

        for right in range(len(nums)):
            subarray_sum += nums[right]
            while subarray_sum >= target:
                min_length = min(min_length, right - left + 1)
                subarray_sum -= nums[left]
                left += 1

        if min_length == INF:
            return 0
        return min_length
```

- ここまで6分。
- > Follow up: If you have figured out the O(n) solution, try coding another solution of which the time complexity is O(n log(n)).
  - うーん、いきなりsliding windowでやったから再検討が必要。
  - まず原点回帰して、最もナイーブには全探索で、O(n^2)となる。
  - しかし、内側のループは無駄が多そう。
  - 前からの累積和は単調増加するので、二分探索（lower bound）で初めて和がtarget以上になる点がわかる。

### 実装2

- 左ポインタが全探索、右ポインタが二分探索
- 計算量
  - Time: O(nlogn)
  - Space: O(n)

```python3
import itertools
import bisect


class Solution:
    def minSubArrayLen(self, target: int, nums: list[int]) -> int:
        INF = len(nums) + 1
        min_length = INF
        
        prefix_sum = [0] + list(itertools.accumulate(nums))
        for left in range(len(nums)):
            right = bisect.bisect_left(prefix_sum, target + prefix_sum[left])
            if right == len(prefix_sum):
                continue
            min_length = min(min_length, right - left)
        
        if min_length == INF:
            return 0
        return min_length
```

- follow-upは8分ほど。
- 今は単元別に学習しているせいで思考が飛躍的だが、本来はこういう方法を経るのが自然な思考回路か。

## Step 2

- レビュー by GPT-5
  - > 実装1で min_length == 1min_length == 1 になったらそれ以上短縮不可なので即 return してもよい。最悪計算量は変わらないが、平均では効くことがある。
    - よく思いつくなぁ。
- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.p6d6fndbrthh)
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1253256524609097801
    - > なんか、right += 1 と prefix_sum += nums[right] が分裂しているのがちょっと気になります。
    - 別の問題で自分も同様のコメントをいただいたことがある。
    - 例えば境界条件の処理などがやりたいときに、インクリメントの遅延で実現するのは好ましくないだろう。（他の選択肢もあるはず）
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1352304493097652315
    - numsに0も許容する -> prefix_sumが広義単調増加になっても、問題ない。
      - target以上なmin_lengthならbisect_left、targetより大きいmin_lengthならbisect_rightで広義もいけてそう。

 ## Step 3

 [実装1](#実装1)
