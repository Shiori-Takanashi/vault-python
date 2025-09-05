
> # for文の基本

Pythonの`for`文は、イテラブル（反復可能オブジェクト）に対して繰り返し処理を行う構文である。

```python
for i in [1, 2, 3]:
    print(i)
```

---

> ## rangeを使った数値の繰り返し

整数の範囲を指定して繰り返す。

```python
for i in range(5):  # 0〜4
    print(i)
```

```python
for i in range(2, 6):  # 2〜5
    print(i)
```

```python
for i in range(10, 0, -2):  # 10から2まで2ずつ減少
    print(i)
```

---

> ## 辞書の反復

キー、値、両方を繰り返す。

```python
d = {"a": 1, "b": 2}

for k in d:
    print(k)  # キーのみ

for v in d.values():
    print(v)  # 値のみ

for k, v in d.items():
    print(k, v)  # キーと値
```


---

> ## enumerateでインデックス付き

リスト等の要素にインデックスを付けて繰り返す。

```python
lst = ["apple", "banana", "cherry"]

for i, fruit in enumerate(lst):
    print(i, fruit)
```


---

> ## zipで並列処理

複数のイテラブルを同時に処理する。

```python
names = ["Alice", "Bob"]
scores = [85, 90]

for name, score in zip(names, scores):
    print(name, score)
```

---

> ## 内包表記での使用

リスト、集合、辞書を一行で生成。

```python
squares = [i**2 for i in range(5)]
evens = {i for i in range(10) if i % 2 == 0}
double_map = {i: i*2 for i in range(3)}
```

---

> ## ネストされたfor文

多次元データの反復に使う。

```python
matrix = [[1, 2], [3, 4]]

for row in matrix:
    for val in row:
        print(val)
```

---

> ## break / continue / elseの利用

繰り返しの制御や完了時の処理。

```python
for i in range(5):
    if i == 3:
        break  # ループ終了
    print(i)
```

```python
for i in range(5):
    if i == 3:
        continue  # スキップ
    print(i)
```

```python
for i in range(5):
    print(i)
else:
    print("すべて完了")  # breakされない場合に実行される
```

---

> ## ファイルオブジェクトやジェネレータでの使用

ファイル行や逐次データに対応。

```python
with open("example.txt", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
```

---

> ## 非同期イテレータとの併用（Python 3.6+）

`async for` は非同期ジェネレータに対して使う。

```python
async def fetch_lines():
    for i in range(3):
        yield i

async def main():
    async for line in fetch_lines():
        print(line)
```

---


> ## 複数代入・アンパックの応用

ループ内で構造を分解して複数変数に代入する。

```python
points = [(1, 2), (3, 4), (5, 6)]

for x, y in points:
    print(f"x={x}, y={y}")
```

---

> ## 辞書のネスト構造の解体

辞書の中の辞書・タプルを分解して処理。

```python
data = {
    "A": ("alice", 90),
    "B": ("bob", 85)
}

for key, (name, score) in data.items():
    print(f"{key}: {name} -> {score}")
```

---

> ## ジェネレータ式との組み合わせ

リストを生成せずメモリ効率良く反復。

```python
for square in (i**2 for i in range(1000000)):
    if square > 100:
        break
    print(square)
```

---

> ## itertoolsと組み合わせた制御

標準ライブラリの `itertools` による高機能ループ。

```python
from itertools import cycle, islice

colors = cycle(["red", "green", "blue"])  # 無限ループ

for color in islice(colors, 5):  # 5回だけ取り出す
    print(color)
```

```python
from itertools import product

for a, b in product([1, 2], ["A", "B"]):
    print(a, b)
```


---

> ## 条件付き内包（複雑な条件式）

条件付きでフィルタ処理＋値変換を同時に行う。

```python
filtered = [x**2 for x in range(10) if x % 2 == 0 and x > 4]
```

---

> ## try-exceptをfor文の中で使う

ループ中の例外処理を行う。

```python
items = ["1", "2", "x", "3"]

for item in items:
    try:
        num = int(item)
        print(num)
    except ValueError:
        print("無効な値")
```

---

> ## 無限ループからの早期脱出

明示的に `break` を活用した sentinel 的使用。

```python
for _ in range(1000):  # 適当な上限
    value = input("数字を入力（qで終了）:")
    if value == "q":
        break
    print(int(value) * 2)
```

---

> ## 関数と組み合わせた高度な反復処理

ループ内で関数を呼び、条件分岐や結果を蓄積。

```python
def classify(n):
    if n % 15 == 0:
        return "FizzBuzz"
    elif n % 3 == 0:
        return "Fizz"
    elif n % 5 == 0:
        return "Buzz"
    return str(n)

for i in range(1, 21):
    print(classify(i))
```

---

> ## ループ変数が外に残ることに注意

ループ終了後も `i` などの変数はスコープに残る。

```python
for i in range(5):
    pass

print(i)  # 4
```


---

> ## for + else を探索アルゴリズムで活用

探索失敗時の通知として `else` 節が使える。

```python
target = 7
for i in range(5):
    if i == target:
        print("見つかった")
        break
else:
    print("見つからなかった")
```

---

> ## reversed() で逆順にループ

```python
for i in reversed(range(5)):
    print(i)
```

---

> ## sorted() でソートされた順に反復

```python
scores = {"Alice": 88, "Bob": 75, "Charlie": 92}

for name in sorted(scores, key=scores.get, reverse=True):
    print(name, scores[name])
```

---

> ## all / any + for（内包表記）による一括条件判定

ループ中にフラグ変数を立てる代わりに論理関数を使う。

```python
nums = [2, 4, 6]

if all(n % 2 == 0 for n in nums):
    print("全て偶数")

if any(n > 5 for n in nums):
    print("1つ以上が5より大きい")
```

---

> ## ループの進行状況を表示（tqdm）

外部ライブラリだが、実務では頻出。

```python
from tqdm import tqdm

for i in tqdm(range(1000000)):
    # 処理
    pass
```

---

> ## for文の中でyieldするジェネレータ関数

イテレータを生成する再帰・分割統治などで使用。

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

for val in flatten([1, [2, [3, 4]]]):
    print(val)
```

---

> ## 条件分岐をループ対象で枝分かれさせる（filter的処理）

`if` をループ対象に適用し、複数リストに分岐。

```python
evens, odds = [], []

for i in range(10):
    (evens if i % 2 == 0 else odds).append(i)
```

---

> ## 不要な変数に `_` を使う

慣習的に使用し、Pythonic な記法として定着している。

```python
for _ in range(3):  # 使わない変数
    print("繰り返し")
```

```python
for _, val in enumerate(["a", "b", "c"]):  # インデックス不要
    print(val)
```

---

> ## map/filter の代替としての for文の使用

読みやすさ優先で明示的なループにする場合がある。

```python
# filter + map の代替
results = []
for i in range(10):
    if i % 2 == 0:
        results.append(i ** 2)
```

---

