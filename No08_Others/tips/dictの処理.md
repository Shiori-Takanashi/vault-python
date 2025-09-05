> # Dictの処理

---

> ## 辞書の基本操作

```python
d = {"a": 1, "b": 2}

d["a"]         # → 1
d.get("b")     # → 2
d.get("x", 0)  # → 0（存在しない場合のデフォルト）
```

---

> ## キー・値・ペアの取り出し

```python
for key in d:
    print(key)

for value in d.values():
    print(value)

for key, value in d.items():
    print(key, value)
```

---

> ## 辞書内包表記

```python
squares = {x: x**2 for x in range(5)}
```

条件付き：

```python
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
```

---

> ## 辞書のキーでソート

```python
sorted_keys = sorted(d)  # ['a', 'b']
for key in sorted(d):
    print(key, d[key])
```

---

> ## 辞書の値でソート

```python
sorted_items = sorted(d.items(), key=lambda x: x[1], reverse=True)
```

---

> ## zipで辞書を作る

```python
keys = ["name", "age"]
values = ["Alice", 30]

d = dict(zip(keys, values))  # {'name': 'Alice', 'age': 30}
```

---

> ## 辞書の更新

```python
d["new"] = 100
d.update({"a": 10})
```

---

> ## 存在チェック

```python
if "a" in d:
    ...
```

---

> ## 辞書の複製（シャローコピー）

```python
d2 = d.copy()
```

---

> ## ネスト辞書へのアクセス

```python
data = {"user": {"name": "Alice", "age": 30}}

name = data["user"]["name"]
```

例外安全：

```python
name = data.get("user", {}).get("name", "Unknown")
```

---

> ## 辞書の値をリスト化・集約

```python
total = sum(d.values())
maximum = max(d.values())
```

---

> ## defaultdictで初期化付き辞書を構築

```python
from collections import defaultdict

dd = defaultdict(int)
dd["a"] += 1
```

リストを初期値に：

```python
dd = defaultdict(list)
dd["a"].append(1)
```

---

> ## Counter（出現回数の辞書）

```python
from collections import Counter

c = Counter(["apple", "banana", "apple"])
# → {'apple': 2, 'banana': 1}
```

---

> ## 辞書のフィルタリング

```python
filtered = {k: v for k, v in d.items() if v > 1}
```

---

> ## 辞書を複数リストに変換

```python
keys = list(d.keys())
values = list(d.values())
items = list(d.items())
```

---

> ## JSONとの相互変換（実務頻出）

```python
import json

json_str = json.dumps(d, ensure_ascii=False, indent=2)
d2 = json.loads(json_str)
```

---

> ## 辞書のマージ（Python 3.9以降）

```python
d1 = {"a": 1}
d2 = {"b": 2}

merged = d1 | d2
```

従来の方法：

```python
merged = {**d1, **d2}
```

---

> ## 辞書をキーでグループ化（手動）

```python
grouped = {}
for item in [{"type": "A", "id": 1}, {"type": "B", "id": 2}, {"type": "A", "id": 3}]:
    t = item["type"]
    grouped.setdefault(t, []).append(item)
```

---

> ## dictのキー・値の変換（再構成）

```python
d = {"a": 1, "b": 2}

# キーと値を入れ替え（注意：値はユニークでなければならない）
inverted = {v: k for k, v in d.items()}
```

---

> ## flatten処理：入れ子辞書を平坦化

```python
def flatten(d, parent_key="", sep="."):
    items = {}
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.update(flatten(v, new_key, sep=sep))
        else:
            items[new_key] = v
    return items

nested = {"a": {"b": 1, "c": 2}, "d": 3}
flat = flatten(nested)
# {'a.b': 1, 'a.c': 2, 'd': 3}
```

---

> ## dictの重複キー集約（groupby風）

```python
from collections import defaultdict

data = [("A", 1), ("B", 2), ("A", 3)]

grouped = defaultdict(list)
for k, v in data:
    grouped[k].append(v)
```

---

> ## ネスト辞書の安全アクセス（関数化）

```python
def deep_get(d, keys, default=None):
    for k in keys:
        if isinstance(d, dict):
            d = d.get(k, default)
        else:
            return default
    return d

config = {"a": {"b": {"c": 42}}}
val = deep_get(config, ["a", "b", "c"])  # → 42
```

---

> ## dict型のマージ＋競合時のルール指定（reduce）

```python
from functools import reduce

dicts = [{"a": 1}, {"b": 2}, {"a": 3}]
merged = reduce(lambda acc, x: {**acc, **x}, dicts, {})
# 最後の"a":3が残る
```

---

> ## dictの値がリストのパターン展開（クロス）

```python
from itertools import product

d = {"color": ["red", "blue"], "size": ["S", "M"]}

for combo in product(*d.values()):
    entry = dict(zip(d.keys(), combo))
    print(entry)
```

---

> ## 複数キーに同じ値を割り当てる初期化

```python
keys = ["a", "b", "c"]
init_val = 0

d = dict.fromkeys(keys, init_val)
```

---

> ## dictの filter / map 的操作（明示的ループ）

```python
# 値を2倍
d2 = {k: v * 2 for k, v in d.items()}

# キーにprefix付与
d3 = {f"pre_{k}": v for k, v in d.items()}
```

---

> ## JSONのキー順維持（Python 3.7+はdict標準）

```python
import json

d = {"z": 0, "a": 1, "m": 2}
json_str = json.dumps(d, ensure_ascii=False)  # キーの順序は保持される
```

---

> ## dictをpandasに変換（構造が整っている場合）

```python
import pandas as pd

data = {"name": ["Alice", "Bob"], "score": [90, 80]}
df = pd.DataFrame(data)
```

辞書のリストから変換：

```python
records = [{"name": "Alice", "score": 90}, {"name": "Bob", "score": 80}]
df = pd.DataFrame.from_records(records)
```

---

> ## キーの構造からツリー状に展開

```python
flat = {"a.b": 1, "a.c": 2, "d": 3}

def unflatten(d, sep="."):
    result = {}
    for k, v in d.items():
        keys = k.split(sep)
        cur = result
        for part in keys[:-1]:
            cur = cur.setdefault(part, {})
        cur[keys[-1]] = v
    return result

nested = unflatten(flat)
# → {"a": {"b": 1, "c": 2}, "d": 3}
```

---

