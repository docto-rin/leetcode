## Step 1

- 問題文
  - あなたはプロの強盗で、ある通り沿いの家々を襲撃して金を盗んでいこうとしている。
  - 各家の所持金を表す整数配列`nums`が与えられる。
  - 任意の連続する2つの家から同時に盗むと警察に通報される。
  - 通報されずに盗める合計金額の最大値を返せ。
  - 制約：
    - 1 <= nums.length <= 100
    - 0 <= nums[i] <= 400

### 実装1

- アルゴリズムの選択
  - 最適部分構造を見つけて動的計画法をやりたい。
    - まずは、0 ~ i - 1番目の家のみの最大値を、0 ~ i番目の家のみでの最大値に活かしたいが、2連続の家が同時に盗めない制約上、これではできない。
    - 直前の家から盗んだか、盗んでないかが分かれば漸化式を立てられそう。
  - 部分問題への分割だが、サイズは常に1ずつしか落とせなさそう。（二分探索的に小さくして、計算量をO(log n)に抑えるのは難しそう。）
- 実装
  - loopでイメージしやすいので、loopで書く。（再帰関数を使うまでもない。）
- 計算量
  - Time: O(n)
  - Space: O(1)

```python3
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
        
        without_last = nums[0]
        with_last = nums[1]
        for i in range(2, len(nums)):
            next_without_last = max(without_last, with_last)
            next_with_last = without_last + nums[i]
            without_last = next_without_last
            with_last = next_with_last
        return max(without_last, with_last)
```

- ここまで10分。
- 初期値を一歩手前からできることに気づいた。

### 実装2

- [実装1](#実装1)のリファクタ

```python3
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        without_last = 0  # max total without robbing last house
        with_last = nums[0]  # max total with robbing last house
        for num in nums[1:]:
            next_without_last = max(without_last, with_last)
            next_with_last = without_last + num
            without_last = next_without_last
            with_last = next_with_last
        return max(without_last, with_last)
```

- len(nums) == 1のとき、nums[1:]が空配列になるためforループがskipされる、というのはパズルなのだろうか。
  - 個人的には特に違和感ないが、これを初めて読む人はどう思うだろうか。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.8mp41bsmfqeb)
  - https://discord.com/channels/1084280443945353267/1196472827457589338/1295462394511757403
    - > このコードはスレッドセーフティーという意味でどうなっているでしょうか。
    - これをみて、nodchip-sanの記事の一節を思い出しました。
      - https://nodchip.hatenadiary.org/entry/2023/03/03/205125
      > 他によく見る例として、コードを書くにあたって、グローバル変数を濫用するというものがあります。競技プログラミングにおいては、様々なメリットから、グローバル変数が使用されます。理由として、以下が挙げられます。
      >
      > - 自動で初期化されるため、記述量が少なくて済む。
      > - 各種コンテナと比較し、多次元配列でも取り回しやすい。
      > - non-static なローカル変数と比較して、大きなサイズの配列を確保しても、スタック領域があふれることがない。
      > 
      > 一方、ソフトウェアエンジニアリングにおいては、グローバル変数は、様々な理由から避けるべきだとされています。理由として、以下が挙げられます。
      >
      > - external linkage となる事も含め、グローバル名前空間を汚す。
      > - 生存期間が必要以上に長い。
      > - 複数個所から参照される場合、コードの流れを追いにくくなる。
      > - 想定外の場所で値が変更される場合がある。
      > - マルチスレッドプログラミングにおいては、スレッド同期を適切に取らないと、スレッド競合を引き起こす。
      > - 複数回呼んだ時に、適切に初期化しないと、毎回計算結果が変わる。
      >
      > これらのデメリットを知らないまま、面接において競技プログラミングの要領でグローバル変数を濫用し、評価を下げてしまう人がいます。面接においては、仕事で書くコードを意識し、一般に避けるべきとされる書き方は避けるべきです。

## Step 3

### 実装3

- [実装2](#実装2)のリファクタ
  - 初期化をさらに一歩手前から始められることに気づいた。

```python3
class Solution:
    def rob(self, nums: List[int]) -> int:
        without_last = 0  # max total without robbing last house
        with_last = 0  # max total with robbing last house
        for num in nums:
            next_without_last = max(without_last, with_last)
            next_with_last = without_last + num
            without_last = next_without_last
            with_last = next_with_last
        return max(without_last, with_last)
```
