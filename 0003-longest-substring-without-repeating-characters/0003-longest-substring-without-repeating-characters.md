## Step 1

- 問題文
  - 文字列`s`が与えられる。重複した文字を使わない最長の部分文字列（substring）の長さを返せ。
  - 制約：
    - 0 <= s.length <= 5 * 10^4
    - s consists of English letters, digits, symbols and spaces.

### 実装1

- アルゴリズムの選択
  - 部分文字列は連続している必要があるので、greedyに解決する可能性がある。
    - 左から走査し、これまでの文字と重複がないかを見るにはstr.findを使えば良いだろう。
    - 重複があった（登場が2回目の文字だった）場合、1回目の文字とそれより左は今後関心がなくなる。
    - 逆に1回目の登場位置より右はまだsubstringの長さが伸びる可能性を秘めている。
    - 自然とダブルポインタ（sliding window）に行き着く。
- 実装
  - loopで手軽に実装
- 計算量
  - Time: O(n^2)
    - 実際にはstr.findの恩恵で速いはず。
  - Space: O(1)

```python3
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        first = 0
        max_length = 0
        for last in range(1, len(s)):
            found = s.find(s[last], first, last)
            if found == -1:
                continue
            # substring is from first to last - 1
            max_length = max(max_length, last - first)
            first = found + 1
        return max(max_length, len(s) - first)
```

- ここまで5分
- 最初、return max(max_length, len(s) - first)のlen(s)をlastと書いてしまい、s=""のエッジケースでUnboundLocalErrorになってしまった。
  - 解決策として、returnの部分をlen(s)にする他、最初にlast = 0と初期化しておくなど。
  - または、max_lengthを常に更新するようにする。
  - 1回目にはやく書けたのが嬉しくて慌ててsubmitしたことを非常に後悔しました。

## Step 2

- [コメント集](https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0)
  - https://discord.com/channels/1084280443945353267/1322513618217996338/1352231179083841536
    - > -1 がでてくればいいので、seen_char_to_index.get(s[right], -1)と使えば、条件分岐を回避できますね。
    - dict.getはキーがないとNone、もしくは引数で渡したデフォルト値にフォールバックする。
    - 強化学習により極度にエラーを恐れているCoding Agentがエラーを握り潰すためによく書いてくるやつ...
- https://github.com/olsen-blue/Arai60/pull/49
  - setや辞書などのhashtableを使う方法は最初に思いついたが、in判定するなら入力`s`のスライスをそのまま活用すればいいと思い、findに移行した。
  - 前者は一応時間計算量はO(n)だが、ハッシュの計算が定数倍でのしかかるのと、str.find()がネイティブコードで高速なので、いい勝負しそう。
    <details>
      <summary>find vs dict ベンチマーク by GPT-5</summary>
    
      <img width="990" src="https://github.com/user-attachments/assets/0df0dbf4-8940-4ff5-bb04-b9744e58bbc2" />
      <img width="990" src="https://github.com/user-attachments/assets/36ecf4eb-7681-45f0-b478-97de3a133e6c" />
    </details>
  - all-uniqueだと倍くらいにはなっているが、findもだいぶ善戦している。
  - まぁけれども、汎用的にはdictのが良さそうか。

### 実装2

- 辞書で各文字が最後に現れたインデックスを管理。
- 計算量
  - Time: O(n)
  - Space: O(n)

```python3
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last_seen = {}
        left = 0
        max_length = 0

        for right, character in enumerate(s):
            left = max(left, last_seen.get(character, -1) + 1)
            last_seen[character] = right
            max_length = max(max_length, right - left + 1)

        return max_length
```

## Step 3

[実装2](#実装2)
