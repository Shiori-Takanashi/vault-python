以下に、**Pythonのlist処理の総合ガイド（実用・応用を含む）**を**Obsidian用マークダウン形式**で統合・再構成しました。

---

# 🌎listの処理

---

> ## listの基本

* 作成

  ```python
  nums = [1, 2, 3]
  words = ["apple", "banana"]
  mixed = [1, "a", True]
  ```

* 空リスト

  ```python
  empty = []
  ```

* インデックス参照・スライス

  ```python
  nums[0]     # 1
  nums[-1]    # 3
  nums[1:3]   # [2, 3]
  ```

---

> ## listの基本操作

* 追加・挿入

  ```python
  nums.append(4)
  nums.insert(1, 100)
  ```

* 削除

  ```python
  nums.remove(2)
  nums.pop()        # 最後尾
  nums.pop(0)       # 先頭
  del nums[1]
  ```

* 検索・判定

  ```python
  nums.index(3)
  4 in nums
  nums.count(3)
  ```

* 並び替え・反転

  ```python
  nums.sort()
  nums.sort(reverse=True)
  nums.reverse()
  ```

* コピー

  ```python
  shallow = nums[:]
  new_list = nums.copy()
  ```

---

> ## リスト内包表記（基本～応用）

* 通常

  ```python
  squares = [x**2 for x in range(5)]
  ```

* if付き

  ```python
  evens = [x for x in range(10) if x % 2 == 0]
  ```

* if-else式

  ```python
  labels = ["even" if x%2==0 else "odd" for x in range(5)]
  ```

* ネストループ（直積）

  ```python
  pairs = [(x,y) for x in range(2) for y in range(3)]
  ```

---

> ## 関数型スタイル

* `map()`

  ```python
  list(map(str, [1, 2, 3]))
  list(map(lambda x: x*2, [1,2,3]))
  ```

* `filter()`

  ```python
  list(filter(lambda x: x > 0, [-1, 0, 1]))
  ```

* `map` + `filter`

  ```python
  list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, range(10))))
  ```

---

> ## 組み込み関数との併用

* `zip()`

  ```python
  list(zip([1,2], ["a","b"]))  # [(1,"a"), (2,"b")]
  ```

* `enumerate()`

  ```python
  for i, x in enumerate(["a", "b", "c"]):
      print(i, x)
  ```

* `any()` / `all()`

  ```python
  any(x > 5 for x in [1,2,6])   # True
  all(x > 0 for x in [1,2,3])   # True
  ```

* 集約関数

  ```python
  sum([1,2,3])
  max([1,5,3])
  min([1,5,3])
  sorted([3,1,2])
  ```

---

> ## 応用構造

* 多重代入・アンパック

  ```python
  a, b, c = [1, 2, 3]
  first, *rest = [1, 2, 3, 4]
  ```

* flatten（1段階）

  ```python
  nested = [[1,2], [3,4]]
  flat = [x for row in nested for x in row]
  ```

* 辞書化（zipから）

  ```python
  dict(zip(["a", "b"], [1, 2]))
  ```

* リスト内辞書のソート

  ```python
  data = [{"name": "A", "score": 90}, {"name": "B", "score": 80}]
  sorted_data = sorted(data, key=lambda x: x["score"], reverse=True)
  ```

---

> ## ユニーク化（重複除去）

* 順序不問

  ```python
  list(set([1, 2, 2, 3]))
  ```

* 順序保持

  ```python
  seen = set()
  result = []
  for x in [1, 2, 2, 3]:
      if x not in seen:
          seen.add(x)
          result.append(x)
  ```

---

> ## 高度な処理（設計・パフォーマンス）

* ジェネレータ内包（遅延評価）

  ```python
  gen = (x**2 for x in range(10**6))
  ```

* flatten + `itertools.chain`

  ```python
  from itertools import chain
  flat = list(chain.from_iterable([[1,2],[3,4]]))
  ```

* chunk分割

  ```python
  def chunk(lst, size):
      return [lst[i:i+size] for i in range(0, len(lst), size)]
  ```

* `collections.deque` の活用（両端高速）

  ```python
  from collections import deque
  dq = deque([1, 2, 3])
  dq.appendleft(0)
  dq.pop()
  ```

* groupbyで分類（前提：ソート済み）

  ```python
  from itertools import groupby
  data = ["apple", "apricot", "banana"]
  data.sort(key=lambda x: x[0])
  grouped = {k: list(g) for k, g in groupby(data, key=lambda x: x[0])}
  ```

---