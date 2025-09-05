## 1章: 特殊メソッドとは何か

---

### 定義と呼び出しのメカニズム

Pythonの特殊メソッド（Dunder Method）は `__xxx__` の形式で定義され、特定の構文・組み込み関数に応じて暗黙に呼び出される。これにより、ユーザー定義クラスでもPythonの文法と自然に連携できる。

| 構文例           | 呼び出される特殊メソッド                   |
| ------------- | ------------------------------ |
| `len(obj)`    | `obj.__len__()`                |
| `str(obj)`    | `obj.__str__()`                |
| `obj[0]`      | `obj.__getitem__(0)`           |
| `obj + other` | `obj.__add__(other)`           |
| `if obj:`     | `obj.__bool__()`               |
| `with obj:`   | `obj.__enter__()`／`__exit__()` |

 `__xxx__`形式は明示的に呼び出すことも可能だが、本来は構文との対応で暗黙的に使う設計が前提である。

---

### 組み込み関数との関係

特殊メソッドは、Pythonの組み込み関数と直接対応している。これにより、クラスのインスタンスが標準的な関数や構文で自然に動作するようになる。

| 組み込み関数        | 対応する特殊メソッド     |
| ------------- | -------------- |
| `repr(obj)`   | `__repr__()`   |
| `format(obj)` | `__format__()` |
| `abs(obj)`    | `__abs__()`    |
| `iter(obj)`   | `__iter__()`   |
| `next(obj)`   | `__next__()`   |
| `bytes(obj)`  | `__bytes__()`  |

---

### 明示呼び出し vs 暗黙呼び出し

| 種類     | 例               | 説明                      |
| ------ | --------------- | ----------------------- |
| 暗黙呼び出し | `str(obj)`      | 組み込み構文や関数によって自動実行       |
| 明示呼び出し | `obj.__str__()` | ユーザーが直接呼び出す（デバッグなどで使用可） |

 通常は暗黙呼び出しで使用し、明示的な呼び出しは挙動確認や特殊な制御が必要な場合に限定される。

---

### 特殊メソッドを定義する意義

* Pythonの構文と**一体化したクラス設計**が可能になる
* インスタンスが**関数・演算子・イテレータ・コンテキスト**として機能する
* 他者が使いやすく、**直感的に動作するオブジェクト**を提供できる
* `list`, `dict`, `int` などの組み込み型もすべて特殊メソッドによって振る舞いが定義されている

---

### よく使われる特殊メソッドカテゴリ

| カテゴリ       | 主なメソッド例                                          |
| ---------- | ------------------------------------------------ |
| 初期化・破棄     | `__init__`, `__del__`, `__new__`                 |
| 文字列・表示     | `__str__`, `__repr__`, `__format__`              |
| 数値・演算子     | `__add__`, `__sub__`, `__eq__`, `__lt__`など       |
| イテレータ・コンテナ | `__iter__`, `__next__`, `__getitem__`, `__len__` |
| コンテキスト管理   | `__enter__`, `__exit__`                          |
| 属性制御       | `__getattr__`, `__setattr__`, `__delattr__`      |
| 呼び出し       | `__call__`                                       |

---

### 注意点とベストプラクティス

* 過剰な特殊メソッドの実装は**可読性を損なう**
* `__getattribute__`や`__del__`は**慎重に扱う**
* `@functools.total_ordering` を使うと比較演算の実装が楽になる
* Pythonicなクラス設計の鍵は「**構文に自然に馴染むふるまい**」を提供すること

---
## 2章: インスタンス生成・初期化・破棄

---

### `__new__`: インスタンス生成の本体

| 特徴        | 内容                                               |
| --------- | ------------------------------------------------ |
| 呼び出しタイミング | クラスからインスタンスを生成する**直前**                           |
| 戻り値       | 通常は `super().__new__(cls)` により**インスタンスを返す**      |
| 主な用途      | 不変型（`tuple`, `str`など）の継承／メタクラス処理／Singletonパターンなど |
| 注意点       | `__init__`より**先に**実行されるため、**生成ロジックが分離**される       |

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        return super().__new__(cls)

    def __init__(self, x):
        print("Initializing with", x)

obj = MyClass(10)
# 出力:
# Creating instance
# Initializing with 10
```

---

### `__init__`: 通常の初期化処理

| 特徴        | 内容                                      |
| --------- | --------------------------------------- |
| 呼び出しタイミング | `__new__` によって生成された**インスタンスに対して初期化**を行う |
| 引数        | `__new__` や `__init__` に渡される引数は共通       |
| 戻り値       | `None` を返すのが原則（戻り値を使う設計ではない）            |

```python
class User:
    def __init__(self, name):
        self.name = name

u = User("Alice")
print(u.name)  # => Alice
```

---

### `__del__`: インスタンス破棄時に呼ばれるが非推奨

| 特徴        | 内容                                      |
| --------- | --------------------------------------- |
| 呼び出しタイミング | オブジェクトが**ガーベジコレクション対象**になるタイミング         |
| 用途        | 外部リソースの明示的な開放（だがwith構文を使う方が望ましい）        |
| 注意点       | 実行タイミングが**非決定的**／循環参照で呼ばれないことも／例外が無視される |

```python
class TempFile:
    def __del__(self):
        print("Cleaning up...")

f = TempFile()
del f  # 即座に呼ばれるとは限らない
```

---

### 実装上のポイントと補足

* `__init__`と`__new__`はセットで理解すべき

  * 不変オブジェクトの設計では`__new__`を活用
  * 通常のデータクラス的な使い方では`__init__`のみで十分
* `__del__`に依存せず、`with`文（コンテキストマネージャ）によるリソース管理が推奨される
* 明示的に破棄したい場合は `del obj` ではなく `close()` メソッドを設けるのが一般的

---

### 補助: `__init_subclass__` と `__class_getitem__`

| メソッド名               | 用途                     | 主なユースケース                      |
| ------------------- | ---------------------- | ----------------------------- |
| `__init_subclass__` | サブクラス生成時のフック           | 抽象基底クラス／制約付き継承チェックなど          |
| `__class_getitem__` | `MyClass[T]` のような記法に対応 | ジェネリクス構文（Python 3.9以降の型ヒント対応） |

これらは特殊メソッドだが、インスタンスではなく**クラスに関する振る舞い**を制御する。

---
## 3章: 呼び出し可能オブジェクト

---

### `__call__`: インスタンスを関数のように使う

| 項目      | 内容                                                 |
| ------- | -------------------------------------------------- |
| 目的      | インスタンスを `obj()` のように**関数呼び出し可能にする**                |
| 引数・戻り値  | 任意の引数を受け取り、任意の値を返す                                 |
| 実行タイミング | `obj(...)` の形式でインスタンスを呼び出した瞬間に `__call__()` が実行される |

```python
class Greeter:
    def __call__(self, name: str) -> None:
        print(f"Hello, {name}!")

greet = Greeter()
greet("Ash")  # => Hello, Ash!
```

---

### 主なユースケース

#### ① 関数オブジェクト（状態保持関数）

関数的に振る舞うが、**状態を持つオブジェクト**を設計できる。

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self):
        self.count += 1
        return self.count

c = Counter()
print(c())  # => 1
print(c())  # => 2
```

#### ② デコレータ構文への応用

`__call__` を使うと、**インスタンスをデコレータとして機能させる**ことが可能。

```python
class Decorator:
    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print("Before")
            result = func(*args, **kwargs)
            print("After")
            return result
        return wrapper

log = Decorator()

@log
def run():
    print("Running")

run()
# 出力:
# Before
# Running
# After
```

#### ③ モデル・コマンドパターン

実行可能なユニットとして設計されたオブジェクト（例: 機械学習モデルの `model(input)`、CLIコマンドクラスなど）

---

### 補足と注意点

* クラスメソッド・静的メソッドとの違いに注意（`__call__`は**インスタンス**に対応）
* lambdaや関数と同じ感覚で使えるが、**コンテキストや状態を持てる**のが大きな利点
* 一見関数のように見えるが、内部ではクラスインスタンスであるため**柔軟な制御が可能**

---

### 実務での位置づけ

| 分類         | 用途・利点                                         |
| ---------- | --------------------------------------------- |
| 高階関数・デコレータ | 構文上は関数のように使えるが、内部で高度な振る舞いを実装可能                |
| 設定済み関数     | 状態を持った「プリセット関数」のような用途 |
| 関数のラッパ     | 実行回数制限、ロギング、遅延評価、失敗時のリトライ処理など                 |

---
## 4章: 文字列表現とバイト表現

---

### `__str__`: ユーザー向けの文字列表現

| 項目    | 内容                      |
| ----- | ----------------------- |
| 呼び出し元 | `str(obj)`、`print(obj)` |
| 目的    | **人間にとって読みやすい形式**での出力   |
| 推奨内容  | 要約・簡潔な説明／エンドユーザー視点での表示  |

```python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f"ユーザー: {self.name}"

print(User("田中"))  # => ユーザー: 田中
```

---

### `__repr__`: 開発者向けの内部表現

| 項目    | 内容                                                     |
| ----- | ------------------------------------------------------ |
| 呼び出し元 | `repr(obj)`、REPL出力、デバッグ出力                              |
| 目的    | **正確・再現性重視の内部表現**（原則 `eval(repr(obj)) == obj` が成り立つ形式） |
| 推奨内容  | クラス名＋引数＋状態など、デバッグ時に役立つ情報                               |

```python
class User:
    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return f"User(name='{self.name}')"

u = User("佐藤")
print(repr(u))  # => User(name='佐藤')
```

---

### `__format__`: f-string 対応

| 項目      | 内容                              |
| ------- | ------------------------------- |
| 呼び出し元   | `f"{obj}"`, `format(obj)`       |
| 利点      | **複雑なフォーマット指定**への対応／型ごとの出力切替が可能 |
| フォールバック | 定義されていない場合は `__str__()` が使用される  |

```python
class Item:
    def __init__(self, price):
        self.price = price

    def __format__(self, format_spec):
        return f"{self.price:,.2f} 円"

item = Item(123456.789)
print(f"価格: {item}")  # => 価格: 123,456.79 円
```

---

### `__bytes__`: バイト列への変換

| 項目    | 内容                        |
| ----- | ------------------------- |
| 呼び出し元 | `bytes(obj)`              |
| 目的    | **バイナリ出力**や通信時の形式指定が可能になる |
| 注意点   | 戻り値は `bytes` 型に限定される必要がある |

```python
class Packet:
    def __init__(self, data):
        self.data = data

    def __bytes__(self):
        return self.data.encode("utf-8")

p = Packet("Ping")
print(bytes(p))  # => b'Ping'
```

---

### 比較と役割の整理

| メソッド         | 対象     | 主な目的                 | 呼び出し例              |
| ------------ | ------ | -------------------- | ------------------ |
| `__str__`    | ユーザー   | 親しみやすい文字列出力          | `print(obj)`       |
| `__repr__`   | 開発者    | 再現性ある表現／デバッグ支援       | `repr(obj)` ／ REPL |
| `__format__` | フォーマット | `f"{obj}"`対応／指定形式の出力 | `f"{obj:,.2f}"`    |
| `__bytes__`  | バイナリ   | バイト列への変換（通信・暗号化等）    | `bytes(obj)`       |

---

### 実務におけるベストプラクティス

* `__repr__` は**常に定義すべき**（トラブル時の追跡に有効）
* `__str__` は**ユーザー出力を意識**した設計を行う（例：UI表示、ログ）
* `__format__` は金額・日付・サイズ等の**構造的な出力制御**に適する
* `__bytes__` は**外部との通信や保存**が関係する場合のみ必要に応じて定義

---
## 5章: 数値・論理・型変換

---

### 数値型への変換

#### `__int__`：整数への変換

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def __int__(self):
        return int(self.celsius)

t = Temperature(36.6)
print(int(t))  # => 36
```

#### `__float__`：浮動小数点への変換

```python
class Ratio:
    def __init__(self, value):
        self.value = value

    def __float__(self):
        return float(self.value)

r = Ratio("3.14")
print(float(r))  # => 3.14
```

#### `__complex__`：複素数型への変換

```python
class Phase:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag

    def __complex__(self):
        return complex(self.real, self.imag)

p = Phase(3, 4)
print(complex(p))  # => (3+4j)
```

---

### 論理値の判定

#### `__bool__`：真偽値評価

| 使用文脈         | 実行される構文                          |
| ------------ | -------------------------------- |
| `if obj:`    | `obj.__bool__()` または `__len__()` |
| `while obj:` | 同上                               |

```python
class User:
    def __init__(self, active):
        self.active = active

    def __bool__(self):
        return self.active

print(bool(User(True)))   # => True
print(bool(User(False)))  # => False
```

> `__bool__()` が定義されていない場合、`__len__()` の戻り値が 0 なら False、それ以外は True。

---

### スライスやビット演算での整数化

#### `__index__`: 整数として扱う

| 使用場面                        | 内容                           |
| --------------------------- | ---------------------------- |
| スライスインデックス                  | `lst[obj]`, `arr[obj:obj+1]` |
| `bin()`, `hex()`, `oct()`関数 | 整数型としての出力に利用                 |
| `range(obj)`                | 明示的に整数として扱う必要がある             |

```python
class ID:
    def __init__(self, value):
        self.value = value

    def __index__(self):
        return self.value

print(bin(ID(7)))    # => 0b111
a = ['a', 'b', 'c', 'd']
print(a[ID(2)])       # => 'c'
```

---

### 型変換メソッドの比較

| メソッド名         | 対応関数                       | 役割                   |
| ------------- | -------------------------- | -------------------- |
| `__int__`     | `int(obj)`                 | 整数への変換               |
| `__float__`   | `float(obj)`               | 浮動小数点への変換            |
| `__complex__` | `complex(obj)`             | 複素数への変換              |
| `__bool__`    | `bool(obj)`                | 真偽値判定                |
| `__index__`   | `bin(obj)`, `range(obj)` 等 | 整数としての扱い（スライスやビット演算） |

---

### 注意とベストプラクティス

* `__int__`, `__float__`, `__bool__` などの戻り値は **対応する型でなければならない**
* `__index__` は「整数型の代替」であり、戻り値は必ず `int` 型
* `__bool__` は **常に明示的に定義**すべき（直感的な真偽値評価を保証する）
* `__len__` を使って代用するケースもあるが、動作の意図を明確にするなら `__bool__` の実装が望ましい

---
## 6章: イテレータとジェネレータのプロトコル

---

### `__iter__`: イテレータの取得

| 内容    | 説明                                      |
| ----- | --------------------------------------- |
| 目的    | `for item in obj:` などの構文に対応             |
| 戻り値   | **イテレータオブジェクト**（通常は `self` か `iter(…)`） |
| 構文の背景 | `iter(obj)` または暗黙に `for` 文などで呼ばれる       |

```python
class MyList:
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        return iter(self.data)

ml = MyList([1, 2, 3])
for x in ml:
    print(x)  # => 1 2 3
```

---

### `__next__`: 次の値の返却

| 内容   | 説明                                |
| ---- | --------------------------------- |
| 目的   | `next(obj)` や `for` ループ内での要素取得に対応 |
| 要件   | `StopIteration` を投げるとループが終了       |
| 使用条件 | クラス自身が **状態を保持しながら要素を1つずつ返す**必要あり |

```python
class CountUp:
    def __init__(self, limit):
        self.i = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.i < self.limit:
            val = self.i
            self.i += 1
            return val
        raise StopIteration

cu = CountUp(3)
for x in cu:
    print(x)  # => 0 1 2
```

---

### `__reversed__`: 逆順イテレーション

| 内容    | 説明                                |
| ----- | --------------------------------- |
| 呼び出し元 | `reversed(obj)`                   |
| 要件    | **逆順での要素取得をサポート**したいときに定義         |
| 戻り値   | イテレータ（通常 `reversed(self.data)` 等） |

```python
class Rev:
    def __init__(self, data):
        self.data = data

    def __reversed__(self):
        return reversed(self.data)

r = Rev([1, 2, 3])
for x in reversed(r):
    print(x)  # => 3 2 1
```

---

### イテレータ vs イテラブル

| 種類    | 必要なメソッド                | 説明                               |
| ----- | ---------------------- | -------------------------------- |
| イテラブル | `__iter__`             | `for`文などに使える                     |
| イテレータ | `__iter__`, `__next__` | 状態を保持しながら要素を1つずつ返す（自身がイテラブルでもある） |

```python
iterable = [1, 2, 3]           # イテラブル（リスト）
iterator = iter(iterable)     # イテレータ取得

print(next(iterator))  # => 1
```

---

### ジェネレータとの関係

| 構文         | 挙動                                 |
| ---------- | ---------------------------------- |
| `yield`使用  | 関数内で状態を保持しながら逐次値を返す ⇒ 自動的にイテレータになる |
| `return`のみ | 通常の関数。繰り返し処理は不可                    |

```python
def count_up(n):
    for i in range(n):
        yield i

for x in count_up(3):
    print(x)  # => 0 1 2
```

---

### 実装指針と注意点

* 要素が明確にあるなら `__iter__` + `iter(...)` で委譲するのが簡潔
* 状態付き処理をしたい場合は `__next__` を含むイテレータとして設計
* `__reversed__` は省略可能だが、逆順操作が必要な場合に便利
* 内部に `yield` を使うことで **簡潔にイテレータを定義**することも可能（ジェネレータ）

---
## 7章: コンテナプロトコル（インデックス・キーアクセス）

---

### `__getitem__`: 要素の取得

| 項目  | 内容                   |
| --- | -------------------- |
| 用途  | `obj[key]` の構文に対応    |
| 引数  | `key`（整数・文字列・スライスなど） |
| 戻り値 | 対応する要素を返す            |

```python
class Box:
    def __init__(self, items):
        self.items = items

    def __getitem__(self, key):
        return self.items[key]

b = Box(["apple", "banana", "cherry"])
print(b[1])  # => banana
```

---

### `__setitem__`: 要素の代入

| 項目  | 内容                                 |
| --- | ---------------------------------- |
| 用途  | `obj[key] = value` に対応             |
| 注意点 | `__getitem__` だけ定義すると**読み取り専用**になる |

```python
class Storage:
    def __init__(self):
        self.data = {}

    def __setitem__(self, key, value):
        self.data[key] = value

    def __getitem__(self, key):
        return self.data[key]

s = Storage()
s["x"] = 42
print(s["x"])  # => 42
```

---

### `__delitem__`: 要素の削除

| 項目  | 内容                        |
| --- | ------------------------- |
| 用途  | `del obj[key]` に対応        |
| 戻り値 | `None`（副作用として内部データが変更される） |

```python
class Memory:
    def __init__(self):
        self.slots = {0: "A", 1: "B"}

    def __delitem__(self, key):
        del self.slots[key]

    def __getitem__(self, key):
        return self.slots[key]

m = Memory()
del m[0]
```

---

### `__contains__`: `in` 演算子への対応

| 項目  | 内容                            |
| --- | ----------------------------- |
| 用途  | `value in obj` の形式に対応         |
| 戻り値 | 真偽値（`True` / `False`）         |
| 補足  | 定義がない場合、`__iter__` による探索が使われる |

```python
class TagSet:
    def __init__(self, tags):
        self.tags = set(tags)

    def __contains__(self, item):
        return item in self.tags

ts = TagSet(["python", "cli"])
print("cli" in ts)  # => True
```

---

### `__len__`: 要素数の取得

| 項目  | 内容             |
| --- | -------------- |
| 用途  | `len(obj)` に対応 |
| 戻り値 | 要素数（int）       |

```python
class Bag:
    def __init__(self, items):
        self.items = items

    def __len__(self):
        return len(self.items)

bag = Bag([1, 2, 3])
print(len(bag))  # => 3
```

---

### プロトコル一覧と目的

| メソッド名          | 主な役割    | 優先度        |
| -------------- | ------- | ---------- |
| `__getitem__`  | 要素の取得   | ◎          |
| `__setitem__`  | 要素の代入   | ◯（ミュータブル時） |
| `__delitem__`  | 要素の削除   | △          |
| `__contains__` | `in` 判定 | ◯          |
| `__len__`      | 要素数の取得  | ◯          |

---

### スライス対応の応用

```python
class Sliceable:
    def __init__(self, data):
        self.data = data

    def __getitem__(self, key):
        return self.data[key]  # int でも slice でも対応

s = Sliceable([0, 1, 2, 3, 4])
print(s[1:4])  # => [1, 2, 3]
```

---

### 実装上の注意点

* `__getitem__` を定義することで **for文や in にも対応可能**
* スライス対応時は `isinstance(key, slice)` で分岐することが多い
* `__contains__` を定義しておくと `in` の評価が高速になる（なければイテレート探索される）

---

## 8章: 属性アクセス制御

---

### `__getattr__`: 存在しない属性にアクセスしたとき

| 項目   | 内容                         |
| ---- | -------------------------- |
| 発動条件 | 属性が見つからなかった場合にのみ呼び出される     |
| 典型用途 | **動的な属性生成**、デフォルト値の提供、プロキシ |
| 注意点  | 存在する属性には**影響しない**（呼ばれない）   |

```python
class Dynamic:
    def __getattr__(self, name):
        return f"<{name}>は定義されていません"

d = Dynamic()
print(d.foo)  # => <foo>は定義されていません
```

---

### `__getattribute__`: すべての属性アクセスに介入

| 項目   | 内容                                                       |
| ---- | -------------------------------------------------------- |
| 発動条件 | 属性アクセス時に**常に呼ばれる**（`obj.attr` すべて）                       |
| 特徴   | `__getattr__` より優先される                                    |
| 注意点  | **無限再帰を防ぐためには `super().__getattribute__(name)` を使う必要あり** |

```python
class Watcher:
    def __getattribute__(self, name):
        print(f"[GET] {name}")
        return super().__getattribute__(name)

    def __init__(self):
        self.x = 100

w = Watcher()
print(w.x)
```

---

### `__setattr__`: 属性代入時に介入

| 項目   | 内容                                                    |
| ---- | ----------------------------------------------------- |
| 発動条件 | `obj.attr = value` の形式で代入されたとき                        |
| 注意点  | 無限再帰を防ぐために `super().__setattr__(name, value)` を使う必要あり |

```python
class Tracker:
    def __setattr__(self, name, value):
        print(f"[SET] {name} = {value}")
        super().__setattr__(name, value)

t = Tracker()
t.data = 42
```

---

### `__delattr__`: 属性削除時に介入

| 項目   | 内容                             |
| ---- | ------------------------------ |
| 発動条件 | `del obj.attr` が実行されたとき        |
| 実行目的 | 削除ログの記録や、保護された属性の削除ブロックなどに利用可能 |

```python
class Guard:
    def __delattr__(self, name):
        print(f"[DEL] {name}")
        super().__delattr__(name)

g = Guard()
g.x = 99
del g.x
```

---

### 用途と設計パターン

| シーン        | 使用メソッド             | 目的                       |
| ---------- | ------------------ | ------------------------ |
| 動的属性の定義    | `__getattr__`      | 存在しない属性の補完／デフォルト応答       |
| すべてのアクセス監視 | `__getattribute__` | ロギング／条件付き分岐／アクセス制限など     |
| 属性代入の制御    | `__setattr__`      | 値のバリデーション／ログ出力／設定の封じ込めなど |
| 属性削除の防御    | `__delattr__`      | 特定属性の削除を禁止／削除時ログ         |

---

### 実装上の注意

* `__getattribute__` と `__setattr__` は **極めて強力だがバグを招きやすい**
* 内部属性アクセスには `super().__getattribute__()` を使うのが鉄則
* Python 3.8以降では `__class_getitem__`, `__init_subclass__` 等も同様に属性操作の対象になる

---

### 応用例：アクセス制限つきオブジェクト

```python
class ReadOnly:
    def __init__(self):
        self._value = 123

    def __setattr__(self, name, value):
        if hasattr(self, name):
            raise AttributeError("再代入は禁止されています")
        super().__setattr__(name, value)

ro = ReadOnly()
# ro._value = 456  # => AttributeError
```

---

### まとめ

| メソッド               | トリガー         | 主な用途                |
| ------------------ | ------------ | ------------------- |
| `__getattr__`      | 存在しない属性にアクセス | 動的応答／プロキシ／既定値など     |
| `__getattribute__` | すべての属性アクセス   | 完全制御／ロギング／制限        |
| `__setattr__`      | 属性代入時        | バリデーション／設定制限／通知処理など |
| `__delattr__`      | 属性削除時        | 削除の防止／イベント通知など      |

---
## 9章: 比較演算子のオーバーロード

---

### 比較演算に対応する特殊メソッド一覧

| 演算子  | メソッド名    | 説明          |
| ---- | -------- | ----------- |
| `==` | `__eq__` | 等価比較（等しいか）  |
| `!=` | `__ne__` | 非等価比較（異なるか） |
| `<`  | `__lt__` | より小さいか      |
| `<=` | `__le__` | 以下か         |
| `>`  | `__gt__` | より大きいか      |
| `>=` | `__ge__` | 以上か         |

---

### `__eq__` と `__ne__`

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)

    def __ne__(self, other):
        return not self == other

p1 = Point(1, 2)
p2 = Point(1, 2)
print(p1 == p2)  # => True
print(p1 != p2)  # => False
```

---

### すべての大小比較を定義するのは手間

4つの順序演算（`<`, `<=`, `>`, `>=`）を全て実装するのは冗長になりやすい。

---

### `functools.total_ordering` による簡略化

| 機能   | 最低限 `__eq__` と1つの順序演算を定義すれば残りを自動生成する |
| ---- | ------------------------------------ |
| 利用方法 | クラスの定義にデコレータを追加                      |

```python
from functools import total_ordering

@total_ordering
class Size:
    def __init__(self, value):
        self.value = value

    def __eq__(self, other):
        return self.value == other.value

    def __lt__(self, other):
        return self.value < other.value

print(Size(10) <= Size(20))  # => True
print(Size(10) > Size(5))    # => True
```

> 内部的には `__le__`, `__gt__`, `__ge__` などが自動実装される。

---

### 比較ロジックの一貫性

| 指針            | 内容                                       |
| ------------- | ---------------------------------------- |
| 同値比較は対称性を保つべき | `a == b` ⇔ `b == a`                      |
| 順序比較は推移性を持つべき | `a < b and b < c` なら `a < c` が成り立つ必要がある  |
| 型チェックの明示化     | `isinstance(other, MyClass)` による安全性確保    |
| 値型での比較を模倣すべき  | ユーザーが `int`, `str` と同じ感覚で使えるようにすることが望ましい |

---

### 応用ユースケース

* ソート対象のクラス（例：スコア、日付、座標など）
* 重複検出（`set` 内の比較、辞書キーの同一性など）
* フィルタリングや判定処理（例：閾値より大きいオブジェクトだけを抽出）

---

### 実装指針まとめ

| メソッド             | 実装指針                                       |
| ---------------- | ------------------------------------------ |
| `__eq__`         | すべての比較の前提。必ず定義する                           |
| `__lt__`など       | 並び順が必要な場合のみ明示的に実装                          |
| `total_ordering` | コード量削減に効果的。順序が一意に定まるクラスで利用推奨               |
| 型安全の確保           | 比較対象に想定外の型が来たら `NotImplemented` を返すのが正しい実装 |

```python
def __eq__(self, other):
    if not isinstance(other, MyClass):
        return NotImplemented
    ...
```

---
## 10章: 数値演算・ビット演算オーバーロード

---

### 算術演算のオーバーロード

| 演算子  | メソッド名          | 説明       |
| ---- | -------------- | -------- |
| `+`  | `__add__`      | 加算       |
| `-`  | `__sub__`      | 減算       |
| `*`  | `__mul__`      | 乗算       |
| `/`  | `__truediv__`  | 除算（小数）   |
| `//` | `__floordiv__` | 除算（切り捨て） |
| `%`  | `__mod__`      | 剰余       |
| `**` | `__pow__`      | べき乗      |

```python
class Number:
    def __init__(self, value):
        self.value = value

    def __add__(self, other):
        return Number(self.value + other.value)

    def __repr__(self):
        return f"Number({self.value})"

a = Number(3)
b = Number(4)
print(a + b)  # => Number(7)
```

---

### 右辺演算のオーバーロード

| 演算子 | メソッド名      | 呼び出される条件            |
| --- | ---------- | ------------------- |
| `+` | `__radd__` | 左辺の型が右辺の型に対応していない場合 |
| `-` | `__rsub__` | `3 - obj` などの形      |

```python
class N:
    def __init__(self, v):
        self.v = v

    def __radd__(self, other):
        return N(other + self.v)

    def __repr__(self):
        return f"N({self.v})"

print(10 + N(5))  # => N(15)
```

---

### インプレース演算（代入付き演算）

| 演算子  | メソッド名      | 説明     |
| ---- | ---------- | ------ |
| `+=` | `__iadd__` | 加算して代入 |
| `*=` | `__imul__` | 乗算して代入 |
| `-=` | `__isub__` | 減算して代入 |

```python
class Total:
    def __init__(self, x):
        self.x = x

    def __iadd__(self, other):
        self.x += other
        return self

    def __repr__(self):
        return f"Total({self.x})"

t = Total(10)
t += 5
print(t)  # => Total(15)
```

---

### ビット演算のオーバーロード

| 演算子  | メソッド名        | 説明       |       |
| ---- | ------------ | -------- | ----- |
| `&`  | `__and__`    | ビットAND   |       |
| \`   | \`           | `__or__` | ビットOR |
| `^`  | `__xor__`    | ビットXOR   |       |
| `<<` | `__lshift__` | 左シフト     |       |
| `>>` | `__rshift__` | 右シフト     |       |

```python
class Flag:
    def __init__(self, value):
        self.value = value

    def __and__(self, other):
        return Flag(self.value & other.value)

    def __repr__(self):
        return f"Flag({self.value})"

f1 = Flag(0b1010)
f2 = Flag(0b1100)
print(f1 & f2)  # => Flag(8)
```

---

### オーバーロードの構造と優先順位

| 分類     | メソッド                    | 呼ばれる条件                |
| ------ | ----------------------- | --------------------- |
| 通常演算   | `__add__`, `__mul__` など | 左辺の型が演算をサポートしている場合    |
| 右辺演算   | `__radd__`, `__rmul__`  | 左辺が対応していない場合に右辺型で処理する |
| インプレース | `__iadd__`, `__imul__`  | `+=` などの代入演算がある場合     |

---

### 実装の指針と注意点

* 戻り値は **新しいインスタンスにすべき**（`__iadd__` 以外）
* `__r*__` を定義しておくと **組み込み型との混在**にも対応できる
* `NotImplemented` を返すことで **他方の演算メソッドに委譲**される

```python
def __add__(self, other):
    if not isinstance(other, MyClass):
        return NotImplemented
    ...
```

---

### 活用例

| 用途                    | オーバーロード演算子                         |
| --------------------- | ---------------------------------- |
| 数値計算クラス               | `__add__`, `__sub__`, `__mul__` など |
| 単位付き値（Length, Massなど） | 算術演算の明示的な制御                        |
| フラグビット処理              | `__and__`, `__or__`, `__xor__`     |
| コスト計算・合計              | `__iadd__` や `__radd__`            |

---
## 11章: 単項演算・型変換演算子

---

### 単項演算とは

Pythonでは、**1つのオペランドに対して作用する演算子**に対しても特殊メソッドを定義できる。以下はその代表例。

---

### `__neg__`: 符号反転（`-obj`）

| 演算子 | 対応メソッド    | 説明             |
| --- | --------- | -------------- |
| `-` | `__neg__` | マイナス符号（負の値に変換） |

```python
class Num:
    def __init__(self, x):
        self.x = x

    def __neg__(self):
        return Num(-self.x)

    def __repr__(self):
        return f"Num({self.x})"

print(-Num(10))  # => Num(-10)
```

---

### `__pos__`: 正符号（`+obj`）

| 演算子 | 対応メソッド    | 説明                   |
| --- | --------- | -------------------- |
| `+` | `__pos__` | 正の符号を明示する（意味を持たせるとき） |

```python
class Positive:
    def __init__(self, x):
        self.x = x

    def __pos__(self):
        return Positive(abs(self.x))

print(+Positive(-7))  # => Positive(7)
```

---

### `__abs__`: 絶対値（`abs(obj)`）

| 関数    | 対応メソッド    | 説明     |
| ----- | --------- | ------ |
| `abs` | `__abs__` | 絶対値の計算 |

```python
class Value:
    def __init__(self, x):
        self.x = x

    def __abs__(self):
        return abs(self.x)

print(abs(Value(-20)))  # => 20
```

---

### `__invert__`: ビット反転（`~obj`）

| 演算子 | 対応メソッド       | 説明          |
| --- | ------------ | ----------- |
| `~` | `__invert__` | ビット反転（1の補数） |

```python
class Bit:
    def __init__(self, x):
        self.x = x

    def __invert__(self):
        return Bit(~self.x)

print(~Bit(0b0101))  # => Bit(-6)
```

---

### 単項演算メソッドまとめ

| 演算子     | メソッド         | 説明           | 主な用途                     |
| ------- | ------------ | ------------ | ------------------------ |
| `-`     | `__neg__`    | 符号反転（負の値にする） | 数値の反転・損益など               |
| `+`     | `__pos__`    | 正符号（意味的な明示）  | 強制正の値、方向や極性の明示など         |
| `abs()` | `__abs__`    | 絶対値          | 大小比較やスカラー値取得など           |
| `~`     | `__invert__` | ビット反転（1の補数）  | フラグ処理、マスク反転、ビット演算の補数操作など |

---

### 注意と補足

* `__neg__` や `__abs__` は **数値系クラスでの基本**としてよく定義される
* `__invert__` は **整数型を意識したビット処理**のためのもので、非数値型には通常不要
* 単項演算は通常演算ほど頻出ではないが、**演算対象を独自型で扱う場合に重要**

---

### 応用例：方向ベクトル

```python
class Direction:
    def __init__(self, x):
        self.x = x

    def __neg__(self):
        return Direction(-self.x)

    def __abs__(self):
        return Direction(abs(self.x))

print(abs(Direction(-5)).x)  # => 5
```

---
## 12章: コンテキストマネージャ（with構文対応）

---

### `with`構文とは

リソースの確保と解放を安全かつ明示的に行うための構文。`try-finally` の簡潔な代替として使われる。`open()` や `threading.Lock()` など多くの標準APIが対応している。

---

### `__enter__`: コンテキスト開始時の処理

| メソッド名       | タイミング             | 戻り値                         |
| ----------- | ----------------- | --------------------------- |
| `__enter__` | `with` ブロックの直前に実行 | `as` 節に渡すオブジェクト（通常は `self`） |

```python
class Resource:
    def __enter__(self):
        print("リソース確保")
        return self
```

---

### `__exit__`: コンテキスト終了時の処理

| メソッド名      | タイミング                   | 引数（例外情報）                        | 戻り値              |
| ---------- | ----------------------- | ------------------------------- | ---------------- |
| `__exit__` | `with` ブロックの終了時または例外発生時 | `exc_type`, `exc_val`, `exc_tb` | `True` を返すと例外を抑制 |

```python
class Resource:
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("リソース解放")
```

---

### 基本構文の例

```python
class MyManager:
    def __enter__(self):
        print("Start")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("End")

with MyManager():
    print("Inside block")
# 出力:
# Start
# Inside block
# End
```

---

### 例外発生時の処理

```python
class SilentManager:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"[例外検知] {exc_type}")
        return True  # 例外を握りつぶす（外に伝播させない）

with SilentManager():
    raise ValueError("問題発生")  # 例外が握り潰される
```

---

### 応用：ファイル処理の安全な確保

```python
class MyFile:
    def __init__(self, path):
        self.path = path
        self.fp = None

    def __enter__(self):
        self.fp = open(self.path, "w")
        return self.fp

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.fp:
            self.fp.close()

with MyFile("test.txt") as f:
    f.write("Hello")
```

---

### 用途と実務応用

| 利用場面       | 内容                                      |
| ---------- | --------------------------------------- |
| ファイル操作     | `open`, `with open(...) as f:` など       |
| DBトランザクション | `commit`／`rollback` 処理の安全な制御            |
| スレッド同期     | `threading.Lock` などの acquire/release 処理 |
| 一時リソース管理   | テンポラリファイル・一時ログ・仮想環境の切替など                |

---

### `contextlib` による簡易記述

関数ベースで書ける簡易構文も存在する：

```python
from contextlib import contextmanager

@contextmanager
def custom():
    print("開始")
    yield
    print("終了")

with custom():
    print("処理中")
```

---

### 実装指針まとめ

| メソッド        | 主な責務             | 戻り値／動作                   |
| ----------- | ---------------- | ------------------------ |
| `__enter__` | リソースの確保、状態の初期化   | `as` に渡すオブジェクト（通常は self） |
| `__exit__`  | リソースの解放、後始末、例外処理 | 戻り値が True なら例外を握り潰す      |

---
