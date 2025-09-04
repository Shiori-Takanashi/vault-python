## 📍1章: 型ヒントとは何か

---

### 📝静的型付けとPythonの動的性

Pythonは動的型付け言語であり、変数や関数の引数に明示的な型指定を必要としません。
しかし、**型ヒント（type hint）**を用いることで、開発時の安全性や補完機能を大きく向上させることができます。

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

上記の例では、`name`が`str`型であること、戻り値が`str`であることを明示しています。
これにより、IDEや型チェッカーが誤りを検出しやすくなります。



### 📝型ヒントの目的とメリット

| 観点         | 説明                                              |
| -- | -- |
| 保守性の向上     | 関数やクラスの使い方が明確になり、他人が読んでも理解しやすい                  |
| バグの早期発見    | 静的解析ツール（例：`mypy`, `pyright`）で**実行前**に型の不一致を検出可能 |
| 補完・リファクタ支援 | 型情報により、IDEでの補完や型ジャンプ、リファクタリングが容易                |
| ドキュメントの簡素化 | docstringを書かずとも、引数や戻り値の仕様をコード上で表現できる            |



### 📝型ヒントの基本構文（PEP 484）

| 要素         | 型ヒントの例                            |
| - |  -|
| 関数引数       | `def add(x: int, y: int) -> int:` |
| 変数の型指定     | `name: str = "Alice"`             |
| 戻り値の型      | `-> bool`                         |
| 引数なし・戻り値なし | `def reset() -> None:`            |



### 📝静的型チェッカーの活用

#### mypy

```bash
$ mypy myscript.py
```

*型ヒントに基づいて、実装と不整合がないかを検査。*

#### pyright（VS Code内蔵 or CLI）

```bash
$ pyright myscript.py
```

*リアルタイムな補完と検査が可能。VS Codeで使われる言語サーバーとしても広く利用されている。*



### 📝型ヒントは実行時には影響しない

型ヒントはあくまで**開発支援用のメタ情報**であり、Pythonのランタイムには一切影響を与えません：

```python
def square(n: int) -> int:
    return n * n

square("文字列")  # 実行時にはエラーにならない（型チェッカーでは警告される）
```



### 📝`__annotations__` によるアクセス

型ヒントは関数やクラスに保存されており、実行時にも確認できます：

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

print(greet.__annotations__)
# {'name': <class 'str'>, 'return': <class 'str'>}
```



### 📝まとめ

| 項目        | 内容                                                                |
| -- | -- |
| 型ヒントの導入目的 | 補完・検査・保守性の強化                                                      |
| チェック方法    | `mypy`, `pyright`, IDEによる静的解析                                     |
| 実行への影響    | 一切なし（ランタイムは型を無視）                                                  |
| 今後の展望     | Python 3.11以降、`Self`, `LiteralString`など進化中（`typing_extensions`参照） |

---

## 📍2章: 基本型

---

### 📝Pythonの基本型と型ヒントの書き方

Pythonの主要なプリミティブ型に型ヒントを付けることで、コードの意図や制約を明確にできます。

| 型名      | 説明                | 型ヒント例                    |
| ------- | ----------------- | ------------------------ |
| `int`   | 整数                | `age: int = 20`          |
| `float` | 浮動小数点数            | `pi: float = 3.14`       |
| `str`   | 文字列               | `name: str = "Alice"`    |
| `bool`  | 真偽値（True / False） | `is_active: bool = True` |

---

### 📝関数における型注釈

```python
def multiply(a: int, b: int) -> int:
    return a * b
```

* `a`, `b` の型は `int`
* 戻り値も `int` と明示することで誤用を防ぐ

---

### 📝型ヒント付き変数定義

型ヒントは変数にも適用可能です。明示することで補完精度が向上し、リファクタリング時の安心感が高まります。

```python
user_id: int = 1234
username: str = "tarou"
ratio: float = 0.75
is_admin: bool = False
```

---

### 📝bool型の注意点

Pythonでは `bool` は `int` のサブクラスであり、`True` は `1`, `False` は `0` として振る舞います。

```python
is_valid: bool = True
print(is_valid + 3)  # 出力: 4
```

* 静的解析の観点からは「論理型と数値型の混用は意図を明確にすべき」とされることが多い

---

### 📝floatの精度と注意点

`float` はIEEE 754に基づく浮動小数点数であり、**誤差を含む**計算になる可能性があります。

```python
x: float = 0.1 + 0.2
print(x == 0.3)  # False
```

→ 正確な比較が必要な場合は `decimal.Decimal` や `math.isclose()` の使用を検討する。

---

### 📝型ヒントを使った型制約の明確化

関数仕様を文書化せずとも、型注釈により意味を明示できます。

```python
def is_adult(age: int) -> bool:
    return age >= 18
```

* 引数に `str` を渡すと型チェッカーが警告を出す
* 補完やIDEの引数ヒントにも反映される

---

### 📝mypyによる型検査の例

```python
def greet(age: int) -> str:
    return "Hello"

greet("二十歳")  # mypy: error: Argument 1 to "greet" has incompatible type "str"; expected "int"
```

---

### 📝Python 3.6以降での型ヒント構文（変数）

```python
score: float
score = 92.5
```

この構文はモダンPythonでは標準的な記法。初期化なしで型だけを示すことも可能。

---

### 📝まとめ

| 型       | 注意点                    |
| ------- | ---------------------- |
| `int`   | 整数。論理値との混用に注意          |
| `float` | 精度誤差に注意。厳密な比較には非推奨     |
| `str`   | Unicode対応。常にクォートで囲む    |
| `bool`  | `int`のサブクラス。加算などの挙動に注意 |
| 型注釈全体   | 補完、検査、ドキュメントの省略が可能に    |

---
## 📍3章: リストと辞書 `List`, `Dict`, `set`, `tuple`

---

### 📝コレクション型の基本と型ヒント

Pythonの主要なコレクション型と、それぞれへの型ヒント指定は以下の通りです（`typing` モジュールを使用）。

| 型       | 型ヒントの例            | 説明                 |
| ------- | ----------------- | ------------------ |
| `list`  | `List[int]`       | 整数のリスト（順序あり）       |
| `dict`  | `Dict[str, int]`  | 文字列キーと整数値の辞書       |
| `set`   | `Set[str]`        | 重複を許さない文字列集合       |
| `tuple` | `Tuple[str, int]` | 位置ごとに異なる型を持つ固定長タプル |

※ `List`, `Dict` などは `typing` モジュールからのインポートが必要（Python 3.8以前）、Python 3.9以降ではビルトイン型 `list[int]` のような表記も可能。

---

### 📝List の型注釈

```python
from typing import List

names: List[str] = ["Alice", "Bob", "Charlie"]
scores: List[int] = [80, 92, 75]
```

* 要素型を明示することで補完やチェックが機能する
* ネスト構造にも対応（後述）

---

### 📝Dict の型注釈

```python
from typing import Dict

user_age: Dict[str, int] = {
    "tarou": 28,
    "satoshi": 32
}
```

* キーとバリューの型を指定
* `Dict[Any, Any]` のような曖昧な型は極力避ける

---

### 📝Set の型注釈

```python
from typing import Set

tags: Set[str] = {"python", "typing", "mypy"}
```

* 順序が保証されないこと、重複が排除されることに注意
* `set()` 初期化で型推論が働かないケースもある

---

### 📝Tuple の型注釈（固定長）

```python
from typing import Tuple

user: Tuple[str, int] = ("Taro", 25)
```

* 順番と長さを固定することで構造が明確になる
* UIデータや行データなど、**意味順序が重要な場面に有効**

---

### 📝可変長 Tuple

```python
from typing import Tuple

data: Tuple[int, ...] = (1, 2, 3, 4)
```

* `...` を使うことで「任意長の同一型タプル」と明示可能

---

### 📝ネスト構造の型ヒント

```python
from typing import List, Dict

students: List[Dict[str, int]] = [
    {"math": 90, "english": 85},
    {"math": 78, "english": 88}
]
```

* JSONや表形式データにおいて、入れ子の型指定が極めて重要
* 間違ったキーや型が入ってもチェッカーが検出できる

---

### 📝`List` vs `list`（`Dict` vs `dict`）の違いと実践

#### 違いの本質

| 比較対象             | 説明                                    |
| ---------------- | ------------------------------------- |
| `List[int]`      | `typing` モジュールに定義された「ジェネリクス型」         |
| `list[int]`      | Python 3.9以降のビルトイン型。PEP 585で導入された簡潔表記 |
| `Dict[str, int]` | 同様に `typing` の型定義                     |
| `dict[str, int]` | Python 3.9以降、正式に使える標準型ベースの表記          |

#### 実用上の使い分け（実務目線）

| 条件                    | 推奨スタイル         | 理由                               |
| --------------------- | -------------- | -------------------------------- |
| Python 3.8以前をサポート     | `List`, `Dict` | 古いバージョンとの互換性あり（typingを使う）        |
| Python 3.9以上のみで構成     | `list`, `dict` | 標準に統合され、読みやすくなった                 |
| PydanticやTypedDictと併用 | `List`, `Dict` | 多くの外部ライブラリがまだ `typing` に依存しているため |

```python
# Python 3.8 以前向け
from typing import List, Dict

users: List[Dict[str, str]] = [{"id": "u01", "name": "Naoya"}]

# Python 3.9 以降なら
users: list[dict[str, str]] = [{"id": "u01", "name": "Naoya"}]
```

> 型ヒントの一貫性のため、**プロジェクト内ではどちらかに統一**するのが望ましい。

---

### 📝Python 3.9以降の記法（ビルトイン型のジェネリクス対応）

```python
users: list[str] = ["A", "B"]
records: dict[str, int] = {"X": 1}
```

* `List[str]` より短く、読みやすい
* `from __future__ import annotations` と組み合わせれば、他の型と混在させやすい

---

### 📝まとめ

| 型                | 特徴                                               | 使用例                          |
| ---------------- | ------------------------------------------------ | ---------------------------- |
| `List`           | 順序あり・重複可                                         | `List[str]`, `List[Dict]`    |
| `Dict`           | キーと値に型制約あり                                       | `Dict[str, int]`             |
| `Set`            | 順序なし・重複なし                                        | `Set[str]`                   |
| `Tuple`          | 順序あり・長さ・位置ごとの型が固定                                | `Tuple[str, int]`            |
| ネスト              | JSONや入れ子構造に必須                                    | `List[Dict[str, List[int]]]` |


---
## 📍4章: ユニオン型とオプション型

---

### 📝ユニオン型とは何か

ユニオン型（`Union`）は「**複数の型のどれか**」を許容する型ヒントです。値がどの型でも良い場合、それらを明示的に指定できます。

```python
from typing import Union

def to_str(value: Union[int, float]) -> str:
    return str(value)
```

* 上記の `value` は `int` でも `float` でも受け付ける
* IDEや型チェッカーは両方の型を考慮して補完・検査する

---

### 📝Optional型とは何か

`Optional[X]` は `Union[X, None]` の簡易表記です。つまり、「**X型またはNoneを受け入れる**」ことを意味します。

```python
from typing import Optional

def greet(name: Optional[str]) -> str:
    if name is None:
        return "Hello, guest"
    return f"Hello, {name}"
```

---

### 📝Union の応用例

#### リテラルと組み合わせた値制約

```python
from typing import Union, Literal

def check_mode(mode: Union[Literal["fast"], Literal["safe"]]) -> None:
    ...
```

#### strまたはintどちらも受け付ける関数

```python
def stringify(x: Union[int, str]) -> str:
    return str(x)
```

---

### 📝Optional の適切な使い方

| 状況            | 書き方                              |
| ------------- | -------------------------------- |
| Noneが入り得る引数   | `Optional[str]`                  |
| デフォルトがNoneの引数 | `name: Optional[str] = None`     |
| 戻り値があるかもしれない  | `def find(...) -> Optional[int]` |

Pythonでは明示的な `Optional[...]` によって、「**Noneの可能性あり**」を設計に組み込むことができる。

---

### 📝`Union` と `Optional` の関係

| 型ヒント                    | 意味                                     |
| ----------------------- | -------------------------------------- |
| `Optional[int]`         | `Union[int, None]` と同義                 |
| `Union[int, str]`       | int か str どちらか                         |
| `Union[int, str, None]` | Optional\[int] + Optional\[str] 的な意味合い |

---

### 📝Unionの分岐処理（型絞り込み）

Pythonの動的型の特性により、ユニオン型は **if文による型判定**で使い分けできます。

```python
def get_length(data: Union[str, list]) -> int:
    if isinstance(data, str):
        return len(data)
    return len(data)  # list の場合
```

* mypy や pyright は `isinstance()` を見て型を自動で絞り込む

---

### 📝Python 3.10 以降の構文（|記号）

Python 3.10 では `Union` に代わり **パイプ記号 `|`** が使用可能です。

```python
def to_str(value: int | float) -> str:
    return str(value)

def greet(name: str | None) -> str:
    ...
```

* `Union[X, Y]` → `X | Y` に書き換え可能
* `Optional[X]` → `X | None`

※ 古いPythonバージョンでは使えないため、プロジェクト方針に応じて選択

---

### 📝Optionalの実務上の注意点

* **Noneを許すなら必ずOptionalを明示**
* 引数に `= None` と書いた場合も、**型注釈にOptionalを忘れないこと**

```python
# ✅ 正しい
def load(path: Optional[str] = None) -> None:
    ...

# ❌ 型ヒント不足
def load(path: str = None):  # pyright: error!
    ...
```


---
## 📍5章: リテラル型と厳密な値制限

---

### 📝リテラル型（`Literal`）とは何か

リテラル型は、「**特定の値だけを受け入れる**」と明示できる型ヒントです。値の取り得る範囲を限定することで、関数や変数の使い方を厳格に定義できます。

```python
from typing import Literal

def set_mode(mode: Literal["debug", "release"]) -> None:
    ...
```

この関数は、`"debug"` または `"release"` の文字列しか受け付けません。
それ以外の値を渡すと、`mypy` や `pyright` が静的エラーを報告します。

---

### 📝リテラル型の基本構文

```python
from typing import Literal

Status = Literal["ok", "error", "pending"]

def update(status: Status) -> None:
    ...
```

* リテラル型は **複数の候補値を列挙**する形で記述
* 複数の値をユニオン的に持つが、**値そのもの**を制限する

---

### 📝実務での用途：フラグ・状態・モード制御

| 使用場面    | リテラル型の例                                 |
| ------- | --------------------------------------- |
| モード切り替え | `Literal["train", "eval"]`              |
| ステータス管理 | `Literal["success", "fail", "pending"]` |
| ユーザーロール | `Literal["admin", "member", "guest"]`   |
| 色やラベル   | `Literal["red", "green", "blue"]`       |

---

### 📝引数や変数に制限をかける

```python
def retry_policy(level: Literal[0, 1, 2]) -> None:
    ...
```

* 文字列だけでなく **数値や真偽値** などもリテラルとして制限可能
* 実装の自由度を保ちながら、**インターフェースの明確化**ができる

---

### 📝Literal型とUnionの違い

| 型ヒント                       | 意味                             |
| -------------------------- | ------------------------------ |
| `Union[str, int]`          | `str` または `int` 型              |
| `Literal["A", "B"]`        | `"A"` または `"B"` という**値そのもの**だけ |
| `Union[Literal["A"], int]` | `"A"` または `int` のいずれかの値        |

→ `Literal` は **型** ではなく **具体的な値** に制限をかける点が特徴

---

### 📝型チェッカーによるリテラル検証

```python
from typing import Literal

def log_level(level: Literal["info", "warn", "error"]) -> None:
    print(level)

log_level("debug")  # pyright/mypy: error!
```

* 型チェッカーは定義外のリテラルを即座にエラーとして検出する
* IDE補完でも候補値を列挙してくれる

---

### 📝Python 3.8未満での使用

`Literal` は `typing_extensions` を使えば下位互換可能：

```python
from typing_extensions import Literal
```

→ Python 3.6, 3.7 でも問題なく使える

---

### 📝Enumとの使い分け

| 特徴       | Literal       | Enum                    |
| -------- | ------------- | ----------------------- |
| 実体は      | 単なる値（定数）      | クラスベースの列挙型              |
| 補完・制限    | IDE補完・型制約が有効  | 同様に有効                   |
| 実行時の属性参照 | 不可（ただの値）      | 可能（`MyEnum.OK` のように参照可） |
| 静的制約     | 軽量かつ高速        | 意味付け・機能付けが可能            |
| 適切な用途    | 関数の引数制限、軽い選択肢 | 定数管理、ロジックの分岐            |

> 意味を持つ状態管理などはEnum、軽量な引数制限にはLiteralが最適。

---

### 📝まとめ

| 項目            | 内容                                          |
| ------------- | ------------------------------------------- |
| `Literal` の定義 | 値を `"A"`, `"B"` など **固定値で制限**する             |
| 使い所           | モード指定、状態管理、ロール指定、レベル制御など                    |
| IDE補完         | 候補が自動補完される（選択ミスを防げる）                        |
| Python互換性     | 3.8未満でも `typing_extensions.Literal` で利用可    |
| Enumとの違い      | `Literal` は軽量な制限、`Enum` は意味付け・属性・ロジックを持つ構造体 |

---
## 📍6章: タプルと固定長シーケンス

---

### 📝タプル型（`Tuple`）とは何か

タプル（`tuple`）は **異なる型の値を固定順で保持**できる不変シーケンス型です。リストと異なり、**要素の数や型の順序が明確な構造**を持つときに使用されます。

```python
from typing import Tuple

user: Tuple[str, int] = ("Taro", 25)
```

* 第1要素は `str`、第2要素は `int` と型と順番が固定されている
* `user[0]`, `user[1]` でアクセス可能だが、構造で意味を持つ用途に適する

---

### 📝Tupleの基本構文と特徴

| 特徴            | 説明                                 |
| ------------- | ---------------------------------- |
| 順序あり、長さ固定     | `Tuple[str, int]` などで定義            |
| 異なる型を含められる    | リストとは違い、各要素の型を個別に指定可能              |
| 不変（immutable） | 要素の追加・削除・変更ができない                   |
| 使い所           | データ行、位置引数、複数戻り値など、構造の意味を明確にする場面で有用 |

---

### 📝可変長タプル（`...`）の使い方

```python
from typing import Tuple

scores: Tuple[int, ...] = (80, 90, 85, 100)
```

* 任意の長さで、すべて同じ型（ここでは `int`）を持つタプル
* リストと異なり変更不可な点に注意

---

### 📝タプルを返す関数の型ヒント

```python
def split_name(fullname: str) -> Tuple[str, str]:
    return tuple(fullname.split(" ", 1))
```

* 関数の戻り値に `Tuple` を指定することで、戻り値の構造を明示
* 多くの戻り値を返す場合、`NamedTuple` や `dataclass` の検討も可

---

### 📝構造の例：位置ごとの意味を持つデータ

```python
product: Tuple[int, str, float] = (101, "ノートPC", 129800.0)
```

* `product[0]`: 商品ID（`int`）
* `product[1]`: 商品名（`str`）
* `product[2]`: 価格（`float`）

→ タプルは「順序に意味がある」場面においてリストより適している。

---

### 📝タプルとリストの使い分け

| 観点   | タプル（`Tuple`）               | リスト（`List`）           |
| ---- | -------------------------- | --------------------- |
| 可変性  | 不可（immutable）              | 可（mutable）            |
| 要素型  | 要素ごとに型を指定（異種型OK）           | 全体に共通の型を指定することが多い     |
| 長さ   | 固定長または任意長（`Tuple[X, ...]`） | 可変長                   |
| 意味づけ | 各位置に意味あり（構造としての意味）         | 順序付きの同型要素の列挙          |
| 使い所  | 戻り値・構造データ・データ行             | 配列データ・反復処理・追加削除可能なリスト |

---

### 📝Python 3.9以降の記法との関係

Python 3.9以降でも `tuple[int, str]` のような書き方は **非対応**。タプルに関しては依然として `typing.Tuple` を使う必要があります（2025年現在）。

---

### 📝ネスト構造のタプル

```python
Matrix2x2: Tuple[Tuple[int, int], Tuple[int, int]]

matrix: Matrix2x2 = (
    (1, 2),
    (3, 4)
)
```

* 多次元配列・行列などを表現可能
* 型の階層が深くなる場合はエイリアスを使って見やすくする

---

### 📝タプルのアンパックと型ヒント

```python
data: Tuple[int, int, int] = (1, 2, 3)
a, b, c = data  # a: int, b: int, c: int
```

* アンパックしても型チェッカーは各変数の型を保持する
* 多変数代入と相性がよく、明示的な構造が期待される

---

### 📝まとめ

| 項目              | 内容                                              |
| --------------- | ----------------------------------------------- |
| `Tuple[X, Y]`   | 要素の型と順序を固定したデータ構造                               |
| `Tuple[X, ...]` | 任意長で要素の型がすべて同じな不変シーケンス                          |
| 用途              | 戻り値、意味ある構造データ、ネスト構造、固定長データの保持などに適する             |
| リストとの違い         | 不変性・型の異種性・位置の意味の強調                              |
| Pythonの制約       | `tuple[int, str]` の記法は未対応（`Tuple[int, str]`を使用） |

---
## 📍7章: Callable（関数型）

---

### 📝Callable型とは何か

`Callable` は「**関数や関数オブジェクトを受け取る変数・引数の型**」を示す型ヒントです。
引数の数や型、戻り値の型までを明確に指定でき、**高階関数**や**コールバック**を受け取る設計に不可欠です。

```python
from typing import Callable

def apply(f: Callable[[int, int], int], x: int, y: int) -> int:
    return f(x, y)
```

上記の関数 `apply` は、2つの `int` を受け取り `int` を返すような関数を引数として受け取れます。

---

### 📝Callableの基本構文

```python
Callable[[引数の型1, 型2, ...], 戻り値の型]
```

#### 例1: 数値を受け取って文字列を返す関数

```python
f: Callable[[int], str]
```

#### 例2: 引数なしで `bool` を返す関数

```python
check: Callable[[], bool]
```

#### 例3: 任意の関数（制約なし）

```python
from typing import Callable

any_func: Callable[..., Any]  # 任意の引数・戻り値
```

---

### 📝関数型を受け取る関数（コールバック）

```python
from typing import Callable

def repeat(n: int, func: Callable[[], None]) -> None:
    for _ in range(n):
        func()
```

* この関数は「引数なし・戻り値なし」の関数 `func` を `n` 回実行
* UIや非同期処理でよく使われるパターン

---

### 📝Callable を返す関数（関数を生成）

```python
def make_multiplier(factor: int) -> Callable[[int], int]:
    def multiply(x: int) -> int:
        return x * factor
    return multiply
```

* クロージャを返す関数の型ヒントに適用可能
* **関数を返す関数**の設計においても型安全性を保てる

---

### 📝実務での利用シーン

| シーン            | 内容例                                                          |
| -------------- | ------------------------------------------------------------ |
| 高階関数の引数        | `sorted(lst, key=Callable[[T], Any])`                        |
| イベントハンドラ       | `on_click(handler: Callable[[], None])`                      |
| デコレータ関数        | `def decorator(f: Callable[..., Any]) -> Callable[..., Any]` |
| 非同期処理のフック      | `on_complete(callback: Callable[[Result], None])`            |
| DI/Factoryパターン | `Callable[[], Service]` のように遅延インスタンス生成に利用されることも多い            |

---

### 📝Callable vs 関数の型名指定（Protocolとの比較）

```python
# Callableによる制約
def operate(f: Callable[[int, int], int]) -> int:
    return f(10, 20)

# Protocolを使って名前付き関数型にする例（後述）
```

* 関数の「名前」や「意味」を明確化したい場合は `Protocol` を用いた関数インタフェースの定義が適する（第14章で扱う）

---

### 📝可読性の工夫：Callableのエイリアス化

```python
from typing import Callable

BinaryOp = Callable[[int, int], int]

def calc(op: BinaryOp, x: int, y: int) -> int:
    return op(x, y)
```

→ **長くなりがちな Callable 型を短縮して再利用**することで、コード全体の可読性が大幅に向上する。

---

### 📝まとめ

| 項目                    | 内容                                                |
| --------------------- | ------------------------------------------------- |
| `Callable[[A, B], R]` | 引数型 `A, B` を取り、戻り値型 `R` を返す関数型                    |
| コールバック                | UI・非同期・イベント・Factoryなど多くの場面で活躍                     |
| 引数なし                  | `Callable[[], R]` で表現                             |
| 任意引数                  | `Callable[..., R]` により柔軟な関数定義が可能                  |
| 可読性向上                 | `Callable` を別名にすることで長く複雑な型定義を短縮・再利用               |
| Protocolとの比較          | 名前・構造を明示した関数型にしたい場合は `Protocol` ベースの設計が有効（14章で詳述） |

---
## 📍8章: Any型と型の曖昧性

---

### 📝`Any` とは何か

`Any` は「**あらゆる型を許容する特別な型**」で、**型チェックを意図的に無効化**するための型注釈です。
どんな値でも代入・受け取り・返却できるため、**型安全性が失われるリスク**も伴います。

```python
from typing import Any

data: Any = "文字列"
data = 123          # OK
data = [1, 2, 3]     # これもOK
```

---

### 📝`Any` の使用効果

| 特性         | 説明                        |
| ---------- | ------------------------- |
| 柔軟性が高い     | どんな型でも代入・受け取り可能           |
| 静的検査をバイパス  | mypy/pyright での型エラーが出なくなる |
| 推論の妨げになる   | IDEや型チェッカーが型の内訳を追跡できなくなる  |
| 避けるべき場面が多い | 型安全な設計を目指す場合、むやみに使うと危険    |

---

### 📝`Any` の典型的な使い道（限定的に使う）

#### 1. JSONのような不定構造の受け取り

```python
from typing import Any

def parse_json(data: Any) -> None:
    ...
```

→ 外部入力やAPIレスポンスの初期処理で、「まだ構造が確定していない」場面に使用する。

#### 2. デバッグ・ラフなプロトタイプ

```python
def debug_print(value: Any) -> None:
    print(value)
```

→ 任意型を受け取りたいが、本番では厳格な型に書き換える前提。

---

### 📝`object` との違い

| 型        | 説明                             |
| -------- | ------------------------------ |
| `Any`    | 静的検査をスルー。何でも代入・使用できる           |
| `object` | すべての型の共通祖だが、**使うにはキャストや検査が必要** |

```python
x: object = "hello"
x.upper()     # ❌型エラー（object型にはupperがない）

y: Any = "hello"
y.upper()     # ✅通る（ただし安全性は保証されない）
```

→ `object` は型安全で、`Any` は柔軟性優先

---

### 📝`Any` が混入した場合の伝播

```python
def unsafe(x: Any) -> int:
    return x + 1
```

* `x` が何であっても型エラーは出ない
* しかし、`str + 1` のような危険な演算も検出されない

→ **`Any` は毒のように他の型注釈に影響を及ぼす**ため、使用箇所は厳密に管理する

---

### 📝mypyの `--disallow-any` 系オプション

| オプション名                      | 効果                          |
| --------------------------- | --------------------------- |
| `--disallow-any-unimported` | 外部ライブラリからの未注釈 `Any` を拒否     |
| `--disallow-any-expr`       | `Any` 型の式を使うとエラー            |
| `--disallow-any-generics`   | `List[Any]` などの曖昧なジェネリクスを禁止 |

→ 型チェックの厳密さを段階的に高められる

---

### 📝`cast()` との併用：Anyから明示的に変換

```python
from typing import Any, cast

x: Any = "hello"
y: str = cast(str, x)
```

* `cast()` は型の明示的な変換を意味し、型チェッカーはその型として扱う
* 実行時には何も起きず、あくまで「型ヒント上の扱い」

---

### 📝実務での指針：Anyの利用ルール

| 状況               | 判断                                          |
| ---------------- | ------------------------------------------- |
| 外部ライブラリの戻り値に型がない | 一時的に `Any` で受け、変換処理で明示的に型指定                 |
| 初期実装・プロトタイピング    | 一時的に使用し、後に厳密な型に置き換える                        |
| 公開関数のインタフェース     | 原則 `Any` は避け、できる限り具体的な型を明示する                |
| データ構造が確定していないとき  | `Any` → `Dict[str, Union[...]]` 等に段階的に落とし込む |

---

### 📝まとめ

| 項目       | 内容                                         |
| -------- | ------------------------------------------ |
| `Any`    | 任意の型を許容。静的型チェックをバイパスする                     |
| `object` | すべての型の親だが、操作にはキャストが必要                      |
| 利用場面     | JSON・外部入力・一時的な実装・外部ライブラリ未対応など              |
| リスク      | 型の伝播によって安全性が崩れ、バグが検出できなくなる可能性あり            |
| 推奨対応     | castで明示、もしくは構造が判明次第 `TypedDict` などで明確化していく |

---
## 📍9章: TypeVarとジェネリック

---

### 📝`TypeVar` とは何か

`TypeVar` は「**任意の型に対して型制約をかけずに汎用的な関数やクラスを定義できる**」仕組みで、\*\*ジェネリック（総称型）\*\*の基盤になります。
Pythonの静的型チェックにおいて、**同一性と制約を両立した抽象性の高い設計**が可能になります。

```python
from typing import TypeVar

T = TypeVar("T")

def identity(x: T) -> T:
    return x
```

* `T` は任意の型を表し、**引数と戻り値が同じ型であること**を保証します

---

### 📝ジェネリックな関数の活用例

```python
T = TypeVar("T")

def first_item(lst: list[T]) -> T:
    return lst[0]
```

* `list[int]` を渡せば戻り値は `int`、`list[str]` なら `str` と解釈される
* 実行時には影響せず、**静的な整合性の検査・補完支援**に活用される

---

### 📝制約付きTypeVar（bound）

```python
from typing import TypeVar

# Tはstrのサブクラスに限定
T = TypeVar("T", bound=str)

def shout(x: T) -> T:
    return x.upper()
```

* `bound=...` によって、Tが特定の型またはそのサブクラスであることを強制できます

---

### 📝複数制約のTypeVar（制限付きユニオン）

```python
S = TypeVar("S", int, float)

def square(x: S) -> S:
    return x * x
```

* `S` は `int` または `float` のいずれかである必要がある（他の型では型エラー）

---

### 📝ジェネリック関数と引数同士の型一致を保証

```python
U = TypeVar("U")

def pair(a: U, b: U) -> tuple[U, U]:
    return (a, b)
```

* 引数 `a` と `b` が**同じ型でなければならない**という制約を静的に与えられる
* `pair(1, "abc")` は型チェッカーによりエラーになる

---

### 📝ジェネリッククラスの基本構文（次章で詳述）

```python
from typing import Generic

T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

* インスタンスごとに異なる型 `T` を使い分け可能
* ジェネリッククラスにおいても `TypeVar` が中核

---

### 📝型の共変・反変（高度な使用）

| 用語                | 説明                                            |
| ----------------- | --------------------------------------------- |
| 共変（covariant）     | サブクラス型への代入を許す（例：`List[Dog]` → `List[Animal]`） |
| 反変（contravariant） | スーパークラス型への代入を許す                               |
| 不変（invariant）     | 上記どちらも不可（Pythonの標準の振る舞い）                      |

```python
T_co = TypeVar("T_co", covariant=True)
```

※ 実務では共変の方がやや登場頻度が高いが、原則として TypeVar は **不変（invariant）** であることに注意

---

### 📝ジェネリック

| 用途               | 利点                                  |
| ---------------- | ----------------------------------- |
| 任意型に対応した汎用関数定義   | 型ごとに関数を作らなくてよくなる                    |
| 引数と戻り値の型の一致保証    | 使用者にも一貫した挙動を期待させる                   |
| 型の制約・絞り込み        | `bound` や具体的な型セットで柔軟かつ安全に           |
| クラスにおけるインスタンス型分離 | コンテナ型やラッパー型を汎用的に使える（BoxやResponseなど） |

---

### 📝まとめ

| 項目             | 内容                                      |
| -------------- | --------------------------------------- |
| `TypeVar("T")` | 任意型Tを定義し、関数やクラスで再利用できる                  |
| bound指定        | Tが特定型のサブクラスであるよう制限できる                   |
| 制限付きTypeVar    | 複数の型だけを許容（例：`TypeVar("S", int, float)`） |
| 型一致の保証         | 複数の引数・戻り値に同一型を適用し、誤用を検出可能               |
| ジェネリック利用       | 関数・クラス・構造体など多くのパターンで使われる、型安全な再利用性向上手段   |

---
## 📍10章: TypedDictと構造型

---

### 📝`TypedDict` とは何か

`TypedDict` は、Pythonの辞書（`dict`）に対して**キーごとに明示的な型を定義できる構造型**です。
「**辞書形式の構造データ**に対して、クラスのように型安全性と補完性を与える」ための仕組みです。

```python
from typing import TypedDict

class User(TypedDict):
    id: str
    name: str
    age: int
```

* 上記は `Dict[str, Any]` より**構造が明確**
* IDE補完や型チェックに強い

---

### 📝通常の辞書とTypedDictの違い

| 比較項目  | 通常の`dict`          | `TypedDict`                  |
| ----- | ------------------ | ---------------------------- |
| 型安全性  | なし                 | **あり**（キーと値の型を明示）            |
| キーの定義 | 書き手・読み手が意識しないといけない | 明示的に定義され、補完・検査が可能            |
| IDE補完 | 期待できない             | 補完される（静的ツールと連携）              |
| 利用場面  | 自由な構造（柔軟性重視）       | APIレスポンス・設定ファイル・構造を固定したデータなど |

---

### 📝TypedDictの書き方

---

#### クラス定義スタイル（推奨）

```python
class Book(TypedDict):
    title: str
    author: str
    year: int
```

#### インラインスタイル（非推奨）

```python
Book = TypedDict("Book", {"title": str, "author": str, "year": int})
```

> クラススタイルは補完・継承・可読性の面で優れている。

---

### 📝インスタンスの使い方

```python
book: Book = {
    "title": "吾輩は猫である",
    "author": "夏目漱石",
    "year": 1905
}
```

* キーの不足・型の不一致があると、`mypy`や`pyright`が警告
* `Dict[str, Any]` では検出できないミスも防げる

---

### 📝任意フィールド（`total=False`）

一部のキーを**任意にしたい**場合は `total=False` を使用：

```python
class PartialUser(TypedDict, total=False):
    id: str
    name: str
    age: int
```

* `total=False` にすると**すべてのキーが任意（省略可）**
* Python 3.11以降では `NotRequired` / `Required` でもっと柔軟に指定可能（→第24章）

---

### 📝TypedDictの継承

```python
class UserBase(TypedDict):
    id: str

class NamedUser(UserBase):
    name: str
```

* 共通フィールドをまとめて、**構造的に分割・再利用可能**
* 過剰な継承は避け、**浅い構造設計**が推奨される

---

### 📝動的辞書との違い：明示性と安全性

```python
user: dict = {"id": 1, "name": "naoya"}  # ❌ 型不明、補完なし
user: User = {"id": "u01", "name": "naoya", "age": 28}  # ✅ 型安全
```

→ `TypedDict` は **柔軟な辞書のメリットを保持しつつ、構造に明確な制約を与える**

---

### 📝実務での用途

| 用途             | 例                              |
| -------------- | ------------------------------ |
| APIレスポンス構造の型定義 | `UserResponse`, `BookData`など   |
| JSONデータ読み込み    | `Dict[str, Any]` より安全な読み込み     |
| 設定ファイル・オプション管理 | `ConfigTypedDict`で厳格な構造チェックを実現 |
| テストデータ構築       | モック辞書に補完と構造を与え、生成ミスを防止         |

---

### 📝まとめ

| 項目          | 内容                                       |
| ----------- | ---------------------------------------- |
| `TypedDict` | 辞書に対して構造的な型注釈を可能にする構文                    |
| キーの型定義      | クラス形式で定義、IDE補完・型検査に強い                    |
| 任意キーの制御     | `total=False` や `NotRequired` で部分的な省略が可能 |
| 利用用途        | API、構成ファイル、モック、構造定義の厳密化など                |
| dictとの違い    | `TypedDict` は**構造・意味・型の明示性**が強化され、誤用が減る  |

---
## 📍11章: Final, ClassVar, Literal

---

### 📝Final（再代入禁止・オーバーライド防止）

#### `Final` とは

`Final` は、**変数や属性・メソッドが再代入・上書きされないことを明示**する型ヒントです。

```python
from typing import Final

MAX_RETRY: Final[int] = 5
```

* `MAX_RETRY` は以後変更不可（型チェッカーが再代入を検出）
* 実行時は影響なし（静的解析のみ）

#### 再代入を防ぐ例

```python
MAX: Final[int] = 10
MAX = 20  # ❌ mypy: Cannot assign to final name "MAX"
```

#### クラスのメソッドに対して使う

```python
from typing import final

class Base:
    @final
    def connect(self) -> None:
        ...
```

```python
class Sub(Base):
    def connect(self):  # ❌ 型エラー：finalメソッドをオーバーライドした
        ...
```

* `@final` は **メソッドのオーバーライド禁止**
* `@final class` も可能（継承を禁止）

---

### 📝ClassVar（インスタンス属性ではなくクラス属性）

#### `ClassVar` とは

`ClassVar` は、**インスタンスごとの属性ではなくクラス全体で共有される属性**であることを示します。

```python
from typing import ClassVar

class User:
    group: ClassVar[str] = "admin"
    id: int
```

| 属性名     | 解釈           |
| ------- | ------------ |
| `group` | クラス変数        |
| `id`    | インスタンス変数（個別） |

#### クラス変数とインスタンス変数の違い

| 比較     | ClassVar          | 通常の属性         |
| ------ | ----------------- | ------------- |
| スコープ   | クラス全体で共有          | 各インスタンスで独立    |
| 意味     | 全体設定、定数、ID共有など    | 個別のプロパティ、状態情報 |
| 型ヒント効果 | 静的解析ツールが明示的に区別できる | 特に区別されない      |

> Python本体では区別しないが、型チェッカーやIDEでの**誤用検出や補完の質が向上**する

---

### 📝Literal（再掲）

`Literal` は、**指定された値そのものだけを許す型**です（第5章ですでに解説済み）。
ここでは他2つとの関係性を明確にするために簡潔にまとめます。

```python
from typing import Literal

def set_mode(mode: Literal["debug", "release"]) -> None:
    ...
```

| 型ヒント       | 目的                    | 使用対象      |
| ---------- | --------------------- | --------- |
| `Final`    | 変数の再代入禁止 / メソッドの上書き防止 | 変数・関数・クラス |
| `ClassVar` | インスタンス変数との区別          | クラス変数     |
| `Literal`  | 値そのものの制限              | 引数や定数     |

---

### 📝3つの併用例

```python
from typing import Final, ClassVar, Literal

class Config:
    VERSION: Final[str] = "1.0.0"           # 再代入禁止
    TYPE: ClassVar[Literal["dev", "prod"]] = "dev"  # クラス全体の限定値
```

* `VERSION` は変更不可能なバージョン定数
* `TYPE` はクラス共通のモード指定（devかprod限定）

---

### 📝実務での使いどころ

| 機能         | 使用例                          |
| ---------- | ---------------------------- |
| `Final`    | 定数、環境設定、絶対に上書きされたくないメソッドの保護  |
| `ClassVar` | 設定値、シングルトンID、ロギングカテゴリなどの共有状態 |
| `Literal`  | モード、フラグ、ステータスなどの選択肢制限        |

---

### 📝まとめ

| 型          | 内容                        | 主な使用対象           |
| ---------- | ------------------------- | ---------------- |
| `Final`    | 再代入・継承・上書きを禁止。安全性・設計意図を明示 | 定数、基底メソッド、環境構成など |
| `ClassVar` | クラス属性を明示的に区別。共有スコープを安全に示す | 設定値、ロジックレベルの定数など |
| `Literal`  | 値の限定により補完・検査精度を強化。選択肢を固定  | 引数・状態管理・モード切替など  |

---
## 📍12章: NewTypeと安全なエイリアス

---

### 📝`NewType` とは何か

`NewType` は「**既存の型に新しい意味を与えるためのラッパー型**」であり、**型安全性を高めるための仕組み**です。
例えば、`int` を使って ID や 年齢 を区別したいときに利用します。

```python
from typing import NewType

UserId = NewType("UserId", int)
```

これにより、`UserId` は `int` と同じ値を持ちながら、**別の型として扱われるようになります（静的解析上）**。

---

### 📝通常の型エイリアスとの違い

```python
UserId = int         # これは完全に同じ型（型区別されない）
UserId = NewType("UserId", int)  # これは別の型として扱われる
```

| 型指定方法                   | 型チェッカーによる扱い             | 意味の違い           |
| ----------------------- | ----------------------- | --------------- |
| `UserId = int`          | 完全に同一型と認識               | 意味だけ付けたエイリアス    |
| `UserId = NewType(...)` | **別型として区別される（intではない）** | 型安全な識別子・意味づけが可能 |

---

### 📝使用例：IDや識別子に意味を持たせる

```python
UserId = NewType("UserId", int)
ProductId = NewType("ProductId", int)

def get_user_name(user_id: UserId) -> str:
    ...

uid: UserId = UserId(123)
pid: ProductId = ProductId(123)

get_user_name(uid)  # ✅ OK
get_user_name(pid)  # ❌ mypy: 型が一致しない
```

* 値は同じ `int` でも、意味が違う → **混同によるバグを静的に防止**

---

### 📝ランタイムでの挙動

`NewType` は \*\*実行時にはただの関数（関数オブジェクト）\*\*です：

```python
print(UserId(42))           # 42
print(type(UserId(42)))     # <class 'int'>
```

→ **型チェックのための仕組みであり、実行時には通常の型と同一**

---

### 📝ジェネリクスと併用可能

```python
T = TypeVar("T")

def get_id(x: T) -> T:
    return x

get_id(UserId(100))  # ✅ OK
```

→ 型安全を維持したまま、汎用的な関数に利用可能

---

### 📝制限と注意点

| 注意点                      | 説明                                                |
| ------------------------ | ------------------------------------------------- |
| 実行時の型は元の型と同じ             | `UserId` は `int` のラッパーでしかなく、`isinstance` では区別できない |
| `UserId + 10` は普通にできてしまう | 型チェックを抜けると通常の `int` として振る舞うため注意が必要                |
| `UserId` を辞書キーに使う場合      | 型チェッカー的には問題ないが、**実行時の挙動に注意（intとして使われる）**          |

---

### 📝使うべき場面

| シーン                      | 利点                                |
| ------------------------ | --------------------------------- |
| IDの型分離（UserId, BookIdなど） | 意味の違いを明示し、**引数の取り違えを防ぐ**          |
| 同じ基本型の複数バリエーション          | 意味論的な型安全性の向上                      |
| セキュリティチェックなどの識別型         | 型を通じた検査の強化。**非セキュア型と混在しないよう分離**可能 |

---

### 📝TypedDictとの併用例

```python
UserId = NewType("UserId", int)

class UserData(TypedDict):
    id: UserId
    name: str
```

→ JSONで受け取った `id` に `UserId(...)` を適用すれば、**構造と意味の両立が可能**になる。

---

### 📝まとめ

| 機能                    | 内容                                                    |
| --------------------- | ----------------------------------------------------- |
| `NewType(name, type)` | 新しい**型意味を持つエイリアス**を作成（ただし実体は元の型）                      |
| 型安全性の向上               | `int` 同士でも `UserId`, `ProductId` など**区別可能**に          |
| 実行時は同一型               | ランタイムでは `int` と同様に振る舞う（静的解析ツールにのみ有効）                  |
| 適用場面                  | ID, ラベル, セキュリティレベル, 意味付けの強い構造の整理に有効                   |
| 他機能との併用               | `TypedDict`, `TypeVar`, `Literal` との組み合わせで意味的厳格性を構築可能 |

---

## 📍13章: クラスの型ヒント

---

### 📝クラスにおける型ヒントの必要性

クラスは**複数の属性と振る舞い（メソッド）をもつ構造体**であり、型ヒントによって以下を明示できます：

* **インスタンス変数の型**
* **メソッドの引数・戻り値の型**
* **初期化時に必要な引数**
* **クラス外からの使用制約（public/private）**

```python
class Book:
    title: str
    author: str
    pages: int

    def __init__(self, title: str, author: str, pages: int) -> None:
        self.title = title
        self.author = author
        self.pages = pages
```

---

### 📝属性への型ヒント

#### クラス定義内に記述（Python 3.6以降）

```python
class User:
    name: str
    age: int
```

* インスタンス生成前から型が明示でき、IDE補完や型検査が機能する
* `__init__` 内だけに書くよりも明確性が高い

#### `__init__` 内に記述する例（OKだが推奨度は下がる）

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age
```

> Pythonでは **両方併用されることが多い**（型安全性と補完のため）

---

### 📝クラスメソッドと型注釈

```python
class Rectangle:
    width: float
    height: float

    def __init__(self, w: float, h: float) -> None:
        self.width = w
        self.height = h

    def area(self) -> float:
        return self.width * self.height
```

* 戻り値があるメソッドでは `-> 型` を明示
* `__init__` は戻り値 `-> None` と書くのが原則

---

### 📝クラス変数 vs インスタンス変数

```python
from typing import ClassVar

class Config:
    VERSION: ClassVar[str] = "1.0"
    name: str
```

| 変数の種類    | 宣言                               | 特徴                  |
| -------- | -------------------------------- | ------------------- |
| インスタンス変数 | `self.name: str` または `name: str` | 各インスタンスごとに独立        |
| クラス変数    | `ClassVar[str]`                  | クラスで共有される。補完や検査で明示可 |

---

### 📝selfの扱いと型の明示

`self` の型は通常 `Book` や `User` など自分自身を意味しますが、**Python 3.11以降**では `Self` 型が導入されました（→15章で詳述）。

---

### 📝プロパティと型ヒント

```python
class User:
    _id: int

    @property
    def id(self) -> int:
        return self._id
```

* `@property` メソッドにも戻り値の型ヒントを忘れずに付けること
* `@setter` にも引数の型を指定できる

---

### 📝`__repr__`, `__str__`, `__eq__` などの特殊メソッドも型付け可能

```python
def __str__(self) -> str:
    return f"{self.title} by {self.author}"
```

---

### 📝実務での活用ポイント

| 項目            | 型ヒントを使う利点                           |
| ------------- | ----------------------------------- |
| モデルクラス（データ表現） | 属性構造を明示して IDE補完・バグ予防                |
| サービス・ロジック     | 引数と戻り値の型制約で**使用方法の誤解を排除**           |
| ドキュメント代替      | docstring が不要になる場合も多く、構造そのもので意図が伝わる |
| デバッグ性・保守性     | IDEの型補完が強化され、関係性を崩さず安全にリファクタ可能      |

---

### 📝型ヒント付きクラスの全体例

```python
class Person:
    name: str
    age: int

    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    def is_adult(self) -> bool:
        return self.age >= 18
```

---

### 📝まとめ

| 構成要素      | 型ヒントの書き方                           | 補足                      |
| --------- | ---------------------------------- | ----------------------- |
| インスタンス変数  | `name: str`                        | クラス内の先頭 or `__init__` 内 |
| クラス変数     | `ClassVar[str]`                    | 設定値や共通値                 |
| コンストラクタ   | `def __init__(...) -> None:`       | 明示必須                    |
| メソッドの戻り値型 | `def foo(...) -> bool:`            | 戻り値が明確なときは必ず型を書く        |
| プロパティ     | `@property def name(self) -> str:` | 読み取り専用のプロパティに明示         |

---
## 📍14章: 抽象基底クラスと Protocol

---

### 📝抽象基底クラス（ABC）とは何か

抽象基底クラス（ABC: Abstract Base Class）は、「**インターフェース（契約）としてのクラス**」を定義するための仕組みです。
Pythonでは `abc` モジュールを用いて、**継承先でのメソッド実装を強制**できます。

```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def fetch(self) -> str:
        ...
```

* このクラスを継承したクラスは、`fetch()` を**必ず実装しなければならない**

---

### 📝ABCの構文と特徴

```python
from abc import ABC, abstractmethod

class Writer(ABC):
    @abstractmethod
    def write(self, content: str) -> None:
        ...
```

#### 派生クラス

```python
class FileWriter(Writer):
    def write(self, content: str) -> None:
        with open("log.txt", "w") as f:
            f.write(content)
```

| 特徴        | 内容                         |
| --------- | -------------------------- |
| 抽象メソッド    | `@abstractmethod` をつける     |
| インスタンス化禁止 | 抽象メソッドを実装しないままではインスタンス生成不可 |
| 継承強制      | サブクラスでの明示的な実装を義務づける        |

---

### 📝Protocol（構造的サブタイピング）

`Protocol` は ABC とは異なり、「**duck typingベースの構造的インタフェース**」を提供します。
**明示的に継承しなくても、構造が一致していれば型チェックが通る**のが最大の特徴です。

```python
from typing import Protocol

class Writer(Protocol):
    def write(self, content: str) -> None:
        ...
```

#### 実装クラス（明示的な継承なしでもOK）

```python
class ConsoleWriter:
    def write(self, content: str) -> None:
        print(content)

def output(w: Writer) -> None:
    w.write("Hello!")
```

* `ConsoleWriter` は `Writer` を継承していなくても、構造が一致すれば OK
* **duck typing × 静的型チェック**の融合

---

### 📝Protocolの活用例

#### 高階関数のインタフェース設計

```python
class CallableFunc(Protocol):
    def __call__(self, x: int, y: int) -> int:
        ...
```

#### \_\_len\_\_や\_\_getitem\_\_のようなメソッド構造にも対応

```python
from typing import Protocol

class Sized(Protocol):
    def __len__(self) -> int:
        ...
```

→ `len(obj)` に対応するクラスであれば、`Sized` として扱える

---

### 📝@runtime\_checkable で実行時チェック可能に

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class JSONSerializable(Protocol):
    def to_json(self) -> str:
        ...

class User:
    def to_json(self) -> str:
        return '{"id": 1}'

assert isinstance(User(), JSONSerializable)  # ✅ True
```

→ 通常、`Protocol` は `isinstance()` 判定できないが、`@runtime_checkable` を使うと **実行時にも型チェック可能**

---

### 📝ABC vs Protocol の違い

| 比較項目      | 抽象基底クラス（ABC）      | Protocol（構造的型）              |
| --------- | ----------------- | --------------------------- |
| 継承の必要性    | 必須                | 不要（構造だけで一致すればOK）            |
| インスタンス化制限 | 抽象メソッド未実装時は不可     | 制限なし                        |
| 構造との一致    | 明示的な継承が必要         | メソッド構造が一致すれば適合              |
| 実行時判定     | `isinstance()` OK | `@runtime_checkable` が必要    |
| 適用場面      | 継承ベースの強い設計、制御の厳密化 | 柔軟なプラグイン設計、Duck Typingの型安全化 |

---

### 📝実務での使い分け

| 目的                     | 選ぶべき型                  |
| ---------------------- | ---------------------- |
| 厳格な継承構造、意図的な設計統制をしたい   | 抽象基底クラス（ABC）           |
| 柔軟なインターフェース抽象、構造で判断したい | Protocol               |
| ライブラリ設計やDIコンテナ、コールバック  | Protocol + Callable など |
| クラス階層で明示的に継承させたい       | ABC                    |

---

### 📝まとめ

| 項目                   | 内容                                       |
| -------------------- | ---------------------------------------- |
| `ABC`                | 抽象メソッドを定義し、継承クラスでの明示的実装を強制。インスタンス生成不可    |
| `Protocol`           | 構造的サブタイプ。継承不要で、構造が一致すれば型適合。IDE補完・チェックに有効 |
| `@runtime_checkable` | `isinstance()` 判定をProtocolでも可能にするデコレータ   |
| 適用対象                 | API設計・インタフェース統一・プラグイン構造・DI設計など           |
| 併用ポイント               | ABCで強制、Protocolで柔軟にカバー、場面に応じた使い分けが重要     |

---
## 📍15章: self と cls の型

---

### 📝`self` の型とは何か？

`self` はインスタンスメソッドにおいて**呼び出し元インスタンス自身**を表します。
通常は型ヒントとしてそのクラス自身（例：`User`）を指定するか、**Python 3.11以降では `Self` 型**を使うことで、より汎用的・再利用可能になります。

---

### 📝`self` にクラス名を使った明示的な型注釈

```python
class User:
    def clone(self: "User") -> "User":
        return User()
```

* クラス定義中ではクラス名を文字列で囲う（**前方参照**を解決するため）
* ただし、クラス名の変更に弱く、**再利用性に乏しい**

---

### 📝`Self` 型（Python 3.11以降）

```python
from typing import Self

class User:
    def clone(self) -> Self:
        return User()
```

| 特徴                 | 説明                                        |
| ------------------ | ----------------------------------------- |
| 再帰的な型推論            | サブクラスで使っても**そのサブクラス自身**を返すと解釈される          |
| 自動的に自クラスに展開        | 明示的なクラス名指定が不要で、保守性が高い                     |
| typing拡張から導入 → 本体化 | `typing_extensions` → Python 3.11本体へ組み込み済 |

---

### 📝`cls` の型（クラスメソッドにおける）

クラスメソッドでは `cls` を第一引数とし、**その呼び出し元クラス自身**を示します。
こちらも `Self` を使えば、**正しい戻り値型（インスタンス）を表現できます**。

```python
class Book:
    def __init__(self, title: str) -> None:
        self.title = title

    @classmethod
    def create(cls, name: str) -> Self:
        return cls(name)
```

→ `cls(...)` の戻り値は「そのクラスのインスタンス」であり、`-> Self` とすれば動的なクラス継承もカバーできます。

---

### 📝`Self` と `TypeVar` の違い

| 型         | 用途               | 違い                          |
| --------- | ---------------- | --------------------------- |
| `Self`    | 「呼び出し元のクラス型」     | **明示不要で、サブクラス展開にも対応**       |
| `TypeVar` | 任意型（汎用関数やクラス）    | 明示的に定義し、他の引数との**型一致や制限**に使う |
| `Type[T]` | クラスそのもの（`cls` 用） | `cls` がクラスオブジェクトであることを明示する  |

#### `Type[T]` の例：

```python
from typing import Type, TypeVar

T = TypeVar("T", bound="User")

class User:
    @classmethod
    def create(cls: Type[T], name: str) -> T:
        return cls(name)
```

---

### 📝実務での使い分け

| ケース                     | 推奨型注釈                             |
| ----------------------- | --------------------------------- |
| `self` を戻す              | `-> Self`（3.11未満なら `"ClassName"`） |
| クラスメソッドがインスタンスを返す       | `cls: Type[T]`, `-> T`            |
| 再帰的なクラス構築・継承時に戻り値の精度が必要 | `-> Self` または `-> T`              |

---

### 📝`Self` を含む構文の互換性

| Pythonバージョン | `Self` 使用可否 | 対応方法                                 |
| ----------- | ----------- | ------------------------------------ |
| 3.11以降      | ✅ 公式サポート    | `from typing import Self`            |
| 3.8〜3.10    | ⛔ 本体未対応     | `from typing_extensions import Self` |

---

### 📝まとめ

| 概念     | 内容                                              |
| ------ | ----------------------------------------------- |
| `self` | インスタンスメソッドの第1引数。型注釈は `"ClassName"` か `Self` が推奨 |
| `cls`  | クラスメソッドの第1引数。`Type[T]` を使えばクラス継承にも対応可能          |
| `Self` | クラス自身の戻り値型として使う。サブクラス対応も含め、クリーンな型記述を実現          |
| 型安全性   | `Self` によってチェーンメソッド・ビルダーの戻り型の**静的保証**が可能        |
| 推奨     | Python 3.11以上では **`Self` を標準とする設計がベストプラクティス**   |

---
## 📍16章: ジェネリッククラスと TypeVar

---

### 📝ジェネリッククラスとは何か

ジェネリッククラスは、**クラスが扱う型を外部から注入（パラメータ化）できるクラス構造**です。
`TypeVar` を用いて、複数の型に対応する **再利用性の高いクラス**を定義できます。

```python
from typing import Generic, TypeVar

T = TypeVar("T")

class Box(Generic[T]):
    def __init__(self, content: T):
        self.content = content

    def get(self) -> T:
        return self.content
```

* `Box[int]`, `Box[str]` など、**型注入により多様な使い方**が可能

---

### 📝ジェネリッククラスの書き方と構造

#### 最低限の構文

```python
from typing import TypeVar, Generic

T = TypeVar("T")

class Container(Generic[T]):
    value: T

    def __init__(self, value: T) -> None:
        self.value = value
```

* `Generic[T]` を継承することで、`T` をクラススコープで利用可能
* `__init__`・メソッド・プロパティに型が伝播される

---

### 📝利用例：数値や文字列に対応する共通Boxクラス

```python
int_box = Box 
str_box = Box[str]("hello")

print(int_box.get())  # 42
print(str_box.get())  # "hello"
```

→ **型チェックツールは get() の戻り値型を自動推論**できる
（int\_box → int、str\_box → str）

---

### 📝ジェネリッククラスのメリット

| 特徴      | 内容                            |
| ------- | ----------------------------- |
| 型の再利用性  | 同じロジックを複数の型で再利用可能             |
| 型の一貫性保証 | メソッド・属性間で同一の型パラメータを通じて整合性を保てる |
| 型安全性の向上 | IDE補完・pyright/mypyでの自動検出が有効   |
| リファクタ耐性 | 型変更の影響範囲が明確で、構造が柔軟            |

---

### 📝型パラメータを複数使う

```python
K = TypeVar("K")
V = TypeVar("V")

class Pair(Generic[K, V]):
    def __init__(self, key: K, value: V) -> None:
        self.key = key
        self.value = value
```

* `Pair[str, int]`, `Pair[int, float]` など柔軟に使える

---

### 📝制約付きジェネリッククラス（bound）

```python
T = TypeVar("T", bound=str)

class LabelBox(Generic[T]):
    def __init__(self, label: T):
        self.label = label
```

* `T` は `str` もしくは `str` のサブクラスのみ受け付ける
* 不正な型は `mypy` で検出可能

---

### 📝型付きデータ構造の構築に有効

```python
class Result(Generic[T]):
    data: T
    success: bool

    def __init__(self, data: T, success: bool) -> None:
        self.data = data
        self.success = success
```

```python
r1 = Result[str]("OK", True)
r2 = Result[int](200, True)
```

→ 汎用的なレスポンスクラス・キャッシュ・ラッパー・DIコンテナなどで大活躍

---

### 📝`__class_getitem__` によるカスタマイズ（高度）

```python
class MyType:
    def __class_getitem__(cls, item):
        print(f"Called with {item}")
        return cls
```

```python
MyType[int]  # 実行時に出力：Called with <class 'int'>
```

→ 型パラメータ使用時の動作をカスタマイズできる（ライブラリ内部で使われる）

---

### 📝実務での使用例

| クラス名         | 型パラメータ | 用途例                      |
| ------------ | ------ | ------------------------ |
| `Box[T]`     | 1つ     | 汎用のコンテナ、キャッシュなど          |
| `Pair[K, V]` | 2つ     | キーと値をペアで管理したい場合（マッピングなど） |
| `Result[T]`  | 1つ     | APIや計算結果のラッパー構造に         |
| `Factory[T]` | 1つ     | クラス/インスタンスの生成器として型安全に構成  |

---

### 📝まとめ

| 概念             | 内容                                       |
| -------------- | ---------------------------------------- |
| `Generic[T]`   | ジェネリッククラスの基本構文。`T` を型パラメータとして持たせる        |
| 複数の型パラメータ      | `Generic[K, V]` によって複数の型を持つ構造体を設計可能      |
| bound指定        | `T = TypeVar("T", bound=Base)` で型の制限を与える |
| 実用的な活用         | Box、Pair、Result、Container などの汎用データクラスに最適 |
| 型安全・補完・再利用性の向上 | 同じ構造で複数の型を安全に扱える。補完精度が高まり、コードの意図が明示化される  |

---
## 📍17章: Enumとの併用

---

### 📝Enum（列挙型）とは何か

`Enum` は「**特定の選択肢の集合を型として定義できる構造**」です。
Pythonの `enum` モジュールを使うと、**状態・定数・モードなどを明確な名前付き値**で扱えるようになります。

```python
from enum import Enum

class Status(Enum):
    OK = "ok"
    ERROR = "error"
    PENDING = "pending"
```

* `Status.OK` のように名前で扱える
* IDE補完や `==` による比較が明確

---

### 📝Enumの基本構文と使い方

```python
from enum import Enum

class Role(Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"
```

```python
def is_admin(role: Role) -> bool:
    return role == Role.ADMIN
```

* 関数引数などにも型ヒントとして利用でき、**誤入力を防止**
* 値だけでなく**名前（属性名）でも意味が伝わる**

---

### 📝IntEnum：整数値と互換性がある列挙型

```python
from enum import IntEnum

class Level(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
```

* `IntEnum` は `int` と比較・計算が可能

```python
Level.HIGH > Level.MEDIUM  # ✅ True
Level.LOW + 1              # ✅ 2
```

→ 数値として使いたいとき（優先度・点数など）に適している

---

### 📝Enumと`Literal`の違い

| 比較項目   | `Enum`                    | `Literal`             |
| ------ | ------------------------- | --------------------- |
| 実体     | オブジェクト（クラス + 値）           | 値そのもの（型チェック用）         |
| 使い方    | `Role.ADMIN` など明示的な名前付き参照 | `"admin"` のような生値      |
| 構造の明示性 | 明確な構造・拡張性あり               | 単純なリスト。意味は外から判断する必要あり |
| 拡張・比較  | 比較・列挙・属性取得が容易             | `==` 比較と補完のみ          |
| 適用シーン  | ロジック分岐、状態管理、設定構造など        | 引数制限、軽量な状態スイッチなど      |

---

### 📝Enumと型ヒントの連携

```python
from enum import Enum

class Mode(Enum):
    TRAIN = "train"
    EVAL = "eval"

def run(mode: Mode) -> None:
    if mode == Mode.TRAIN:
        ...
```

* 型ヒントに `Mode` を使うことで、**誤った文字列を防止**
* IDE補完・静的解析も有効

---

### 📝実行時の利用：名前と値の取得

```python
print(Mode.TRAIN.name)  # TRAIN
print(Mode.TRAIN.value) # "train"
```

---

### 📝Enumのループ処理や比較

```python
for mode in Mode:
    print(mode.value)
```

```python
if user_input in Mode.__members__:
    mode = Mode[user_input]
```

* `__members__` によって文字列→Enumの変換が可能

---

### 📝Enum + `TypedDict` の組み合わせ

```python
from typing import TypedDict

class Config(TypedDict):
    mode: Mode
    verbose: bool
```

* 設定構造における**選択肢の安全な制限**に利用できる

---

### 📝実務での用途まとめ

| 用途      | Enumクラス名例           | 説明                  |
| ------- | ------------------- | ------------------- |
| ロール管理   | `UserRole`          | 権限・アクセス制御の切替        |
| ステータス管理 | `Status`, `State`   | 成功/失敗/処理中 などの状態遷移   |
| 処理モード選択 | `Mode`, `Phase`     | 学習/推論/評価モードなど       |
| 優先度・レベル | `Priority`, `Level` | 数値型と併用して直感的な比較処理が可能 |
| 設定オプション | `ConfigKey`         | 設定項目をEnumで安全に識別     |

---

### 📝まとめ

| 概念        | 内容                                         |
| --------- | ------------------------------------------ |
| `Enum`    | 名前付きの定数集合。選択肢や状態を安全・明示的に管理                 |
| `IntEnum` | 数値比較・演算が可能な列挙型。優先度や段階で有効                   |
| `Literal` | 型チェック目的のリテラル限定値。軽量な切替やスイッチ引数に向く            |
| 型ヒントとの連携  | `Enum`型を引数や戻り値に指定することで補完・誤入力防止が可能          |
| 実務での利点    | 状態管理、ロール、優先度、設定項目などに意味を持たせ、**構造と意味を両立**できる |

---
## 📍18章: 例外と型ヒント

---

### 📝`raise` と型ヒントの関係

Pythonの例外処理（`raise`）は通常、型ヒントと直接的な関係はありません。
ただし、関数が**正常に終了しないことを明示**する場合には、特殊な型ヒント `NoReturn` を使います。

```python
from typing import NoReturn

def fatal_error(msg: str) -> NoReturn:
    raise RuntimeError(msg)
```

* この関数は**絶対に戻ってこない**
* `NoReturn` は**到達不能な制御フロー**を示すための型

---

### 📝`NoReturn` の使用場面

| シーン            | 内容                          |
| -------------- | --------------------------- |
| 致命的なエラー発生時     | `raise` によって即座に停止           |
| システム終了・強制終了処理  | `sys.exit()` を呼ぶ関数          |
| 一方通行のループ・ブロッカー | 無限ループまたは中断不能処理の中で例外を発生させる設計 |

```python
import sys

def terminate() -> NoReturn:
    sys.exit(1)
```

---

### 📝例外を返さない設計を強調する

```python
def validate(x: int) -> None:
    if x < 0:
        raise ValueError("must be non-negative")
```

* この関数は戻り値を返さないが、`raise` によって異常系へ分岐
* `None` を明示することで「成功時は何も返さない」ことを示す

---

### 📝`try-except` と型ヒントの関係

```python
def safe_div(a: int, b: int) -> float:
    try:
        return a / b
    except ZeroDivisionError:
        return 0.0
```

* 通常は型ヒントに影響しない（戻り値型はあくまで `float`）
* `Optional[float]` を返すような設計も考慮可能

---

### 📝Optionalと例外処理の使い分け

| 設計             | 説明                         |
| -------------- | -------------------------- |
| `Optional` を使う | 戻り値がない場合でも「正常系」として扱える      |
| 例外で表現する        | 明確な失敗（エラー）を呼び出し元に伝えたい場合に使う |

```python
def find_user(user_id: str) -> Optional[dict]:
    ...

def get_user_or_raise(user_id: str) -> dict:
    user = find_user(user_id)
    if user is None:
        raise KeyError(f"{user_id} not found")
    return user
```

---

### 📝例外クラス自体に型を使う（構造化）

```python
class InvalidAgeError(ValueError):
    def __init__(self, age: int) -> None:
        self.age = age
        super().__init__(f"Invalid age: {age}")
```

* 属性に型ヒントを付けることで補完・型チェックも有効
* `__init__` にも `-> None` を忘れずに

---

### 📝例外処理と `NoReturn` の相互運用

```python
def validate_id(id_: str) -> None:
    if not id_.startswith("u"):
        fatal_error("Invalid ID")  # NoReturnを使う関数
```

→ `NoReturn` を使う関数が `raise` を含んでいると、**それ以降のコードが到達不能**として扱われる。
これにより静的解析ツールが「このif文のelse側は常に通る」と推論可能になる。

---

### 📝ツールでの活用（mypy, pyright）

| ツール       | 例外型の静的解析対応                     |
| --------- | ------------------------------ |
| `mypy`    | `NoReturn` 関数を使ったフロー制御の誤りを検出可能 |
| `pyright` | 到達不能コード、戻り値型の不整合などを検出          |
| 補完ツール     | IDEで `raise` を含む戻り型の補完が明確になる   |

---

### 📝まとめ

| 概念           | 内容                                            |
| ------------ | --------------------------------------------- |
| `NoReturn`   | 正常に戻ることがない関数を示す型。`raise` や `sys.exit()` と併用   |
| `-> None`    | 成功時に値を返さないことを明示する                             |
| 例外クラスの型注釈    | 属性や `__init__` に型ヒントをつけることで、構造の補完と検査が可能に      |
| Optionalとの分離 | 成功/失敗を「戻り値で表現」するか「例外で通知」するかは**設計意図に応じて選択**すべき |
| 静的解析との相性     | `NoReturn` や構造化例外によって、より安全なフロー制御と型検査が可能       |

---
## 📍19章: mypyと型チェック実行

---

### 📝`mypy` とは何か

`mypy` は Python に対する **静的型チェッカー** です。
型ヒント（type hint）を使ったコードに対して、**実行前に型の不整合やミスを検出**できます。

```bash
$ mypy script.py
```

* コンパイル不要のPythonコードに対して **型による厳格なチェック**を導入できる
* 保守性・可読性の高いコードベースを構築するために非常に有効

---

### 📝基本的な使い方

```bash
$ mypy myfile.py
```

```python
def add(x: int, y: int) -> int:
    return x + y

add("3", 4)  # ❌ 型エラー
```

```bash
error: Argument 1 to "add" has incompatible type "str"; expected "int"
```

---

### 📝チェック対象の例と検出パターン

| ミスの種類                 | 検出例                                       |
| --------------------- | ----------------------------------------- |
| 引数の型不一致               | `add("abc", 1)` で `int` 期待に `str` 渡し      |
| 戻り値型の不一致              | `def f() -> str: return 123`              |
| 属性未定義                 | `obj.name` が未定義（クラスの型ヒント未指定）              |
| Union型の誤使用            | `Union[int, str]` に対して `.upper()` 呼び出しなど  |
| OptionalをNoneチェックせず使用 | `x: Optional[str]` に対して直接 `.upper()` 呼び出し |

---

### 📝`mypy.ini` 設定ファイルでの高度な制御

#### ファイル例：

```ini
[mypy]
strict = True
ignore_missing_imports = True
disallow_untyped_defs = True
warn_unused_ignores = True
```

| オプション名                   | 説明                         |
| ------------------------ | -------------------------- |
| `strict`                 | 一括で厳格モードを有効に（以下すべてON）      |
| `disallow_untyped_defs`  | 型注釈のない関数をエラー扱いに            |
| `warn_unused_ignores`    | `# type: ignore` の無駄な使用を検出 |
| `ignore_missing_imports` | 外部ライブラリの型ヒント不足を黙認（便宜的）     |

---

### 📝ディレクトリ・プロジェクトへの適用

```bash
$ mypy src/
$ mypy main.py tests/ --config-file mypy.ini
```

* `src/`, `tests/` のように複数対象に一括適用可能
* CIツールやGitHub Actionsに組み込みやすい

---

### 📝`# type: ignore` の活用

一時的に特定の行の型チェックをスキップしたい場合に使用：

```python
do_something()  # type: ignore
```

> 誤用を防ぐために `warn_unused_ignores = True` を有効化しておくとよい

---

### 📝`mypy` と他ツールの比較

| ツール名      | 特徴                                |
| --------- | --------------------------------- |
| `mypy`    | 歴史ある静的型チェッカー。柔軟かつ精度が高い            |
| `pyright` | Microsoft製。高速・補完強力（VS Code向け）     |
| `pylance` | Pyrightベースの補完エンジン（型検出はpyrightに準拠） |

→ **mypyはCIやCLIチェックに、pyrightは日常開発（補完）に向く**

---

### 📝型チェックのCI連携例（GitHub Actions）

```yaml
- name: Run mypy
  run: mypy src tests --config-file mypy.ini
```

* プッシュ・PR時に型チェックを自動実行できる
* 失敗時にCIがエラーを返すことで、型の品質を強制できる

---

### 📝よくある質問と対策

| 疑問                     | 回答                                                     |
| ---------------------- | ------------------------------------------------------ |
| Pythonは動的型なのに型チェックは必要？ | 実行前にバグを防げる・補完やドキュメントが強化される                             |
| 型が面倒にならないか？            | `strict=False` など段階的に導入すれば負担は少ない                       |
| 外部ライブラリの型がない           | `py.typed` 対応のライブラリを優先 or `ignore_missing_imports` を利用 |
| エラーが大量に出てしまう           | 部分的に `# type: ignore` や `@overload` で対応                |

---

### 📝まとめ

| 概念         | 内容                                      |
| ---------- | --------------------------------------- |
| `mypy`     | Pythonの静的型チェックツール。型注釈に従い実行前に不整合を検出      |
| チェック対象     | 関数定義・戻り値・クラス属性・Optionalチェックなど幅広い        |
| `mypy.ini` | プロジェクト単位で設定し、CIやコマンドで一貫性を保つ             |
| 他ツールとの併用   | `pyright`と併用することで、開発時・CI時に補完と検査の両立が可能   |
| 導入メリット     | バグの早期発見、補完精度向上、保守性の強化、型安全なチーム開発への橋渡しに有効 |

---
## 📍20章: VSCodeとpyrightの連携

---

### 📝`pyright` とは何か？

`pyright` は Microsoft によって開発された **超高速な静的型チェッカー** であり、VS Code の Python補完エンジン `Pylance` のベースでもあります。

* 型注釈（PEP 484）に対応
* `mypy` よりもチェック速度が非常に速い
* VS Codeと完全統合されており、補完・警告表示が即時に反映される

---

### 📝VSCodeでの連携方法

#### 必須条件

| 項目                 | 推奨設定内容                                  |
| ------------------ | --------------------------------------- |
| 拡張機能               | ✅ Python、✅ Pylance（Pyrightエンジン）         |
| settings.json      | `python.analysis.typeCheckingMode` を有効に |
| pyrightconfig.json | プロジェクト設定ファイル（後述）で制御可能                   |

---

### 📝`settings.json` の設定例

```json
{
  "python.analysis.typeCheckingMode": "strict",
  "python.linting.enabled": true,
  "python.linting.mypyEnabled": false,
  "python.analysis.autoImportCompletions": true
}
```

#### `typeCheckingMode` の値

| モード        | 内容                           |
| ---------- | ---------------------------- |
| `"off"`    | 型チェックを無効化（補完のみ）              |
| `"basic"`  | 軽度なチェック（型なし関数などは許容）          |
| `"strict"` | 厳格な型チェック（`mypy --strict` 相当） |

---

### 📝`pyrightconfig.json` の導入（プロジェクト単位）

プロジェクトごとの設定を厳密にしたい場合、ルートに `pyrightconfig.json` を配置：

```json
{
  "include": ["src", "tests"],
  "exclude": ["scripts", "**/__pycache__"],
  "typeCheckingMode": "strict",
  "reportMissingImports": true,
  "reportOptionalMemberAccess": true,
  "reportPrivateUsage": "warning"
}
```

| オプション名                       | 内容                           |
| ---------------------------- | ---------------------------- |
| `include`                    | チェック対象のディレクトリ                |
| `exclude`                    | チェック除外対象                     |
| `typeCheckingMode`           | チェック厳度（off / basic / strict） |
| `reportMissingImports`       | import先の型ヒントがないと警告           |
| `reportOptionalMemberAccess` | Optional型に対する属性アクセスのチェック     |
| `reportPrivateUsage`         | `_private` なメンバの使用に対して警告出力   |

---

### 📝pyrightの警告と補完

#### 補完の特徴

* `Literal`, `TypedDict`, `Enum` も補完対象
* `TypeVar` による関数の型推論も対応済み
* `Callable` の構造チェックや `Protocol` の一致判定も強力

#### エラー表示例（`strict` モード）

```python
def hello(name: str) -> int:
    return f"Hello, {name}"  # ❌ Expected int, got str
```

---

### 📝VSCodeの型チェックフロー

```plaintext
ファイル保存 → Pyrightが即時型チェック → エディタ上に警告（赤波線）＋補完候補提示
```

* 補完時にも型が尊重される（戻り値型・引数型を元に推論）
* 複雑な型でも `Ctrl+Hover` で定義や推論内容を確認できる

---

### 📝VSCode × pyright の補完例

```python
from typing import TypedDict

class User(TypedDict):
    name: str
    age: int

user: User = {"name": "Naoya", "age": 30}
user[""]  # ← 補完候補として "name", "age" が表示される
```

---

### 📝mypy との併用は可能？

| 項目     | 回答                                       |
| ------ | ---------------------------------------- |
| 同時利用   | ✅ 可（VSCode: pyright、CI: mypy）            |
| 設定ファイル | 分離して管理（`mypy.ini`, `pyrightconfig.json`） |
| 実務運用   | 開発補完は pyright、品質保証は mypy という使い分けが主流      |

---

### 📝よくある問題と対策

| 問題                          | 原因と対策                                        |
| --------------------------- | -------------------------------------------- |
| `pyright` が型を認識しない          | `py.typed` 非対応ライブラリ → `typings` を別途追加        |
| import に波線（Missing Imports） | `reportMissingImports` → false または venv設定確認  |
| TypedDict の補完が効かない          | dictではなく**クラス形式の TypedDict** を使用すること         |
| settings.json を変えても動作が変わらない | Pythonインタープリタ再選択 → workspace設定優先で反映されないことがある |

---

### 📝まとめ

| 項目                   | 内容                                                       |
| -------------------- | -------------------------------------------------------- |
| `pyright`            | Microsoft製の高速型チェッカー。Pylanceで補完・警告を即時提供                   |
| VSCodeとの統合           | `settings.json` で制御。`strict` で厳格な検査が可能                   |
| `pyrightconfig.json` | プロジェクト単位でルール設定。CIとの共存にも最適                                |
| `mypy` との併用          | VSCodeでは pyright、CIでは mypy を使うのが実務的                      |
| 補完精度・チェック精度          | `TypedDict`, `Protocol`, `Enum`, `TypeVar` すべてに対応。補完も高精度 |

---
## 📍21章: テストとの協調（型ヒント × pytest）

---

### 📝型ヒントはテストでも活きるか？

答えは **Yes**。
**pytest においても型ヒントは補完・品質・設計の観点で有効に機能**します。

* fixture や テスト関数 の型注釈が補完に効く
* IDEや型チェッカーによって fixture の誤用や戻り値の不一致を検出できる
* 特に `TypedDict` や `Literal`、`Enum` との併用でテストデータの意味を明示可能

---

### 📝pytest のテスト関数に型ヒントを付ける

```python
def test_add() -> None:
    assert 1 + 1 == 2
```

* 戻り値は必ず `-> None` とする（pytestが返り値を使うことはない）
* 型ヒントがなくても動作するが、**一貫した設計のために明示が望ましい**

---

### 📝テスト引数に型を付ける（fixtureやパラメータ）

```python
import pytest

@pytest.fixture
def sample_data() -> dict[str, int]:
    return {"x": 1, "y": 2}

def test_sum(sample_data: dict[str, int]) -> None:
    assert sample_data["x"] + sample_data["y"] == 3
```

→ fixture にも型を付けておくことで、**型補完やチェックが働く**

---

### 📝fixture同士の連携にも型ヒントが有効

```python
@pytest.fixture
def user_id() -> str:
    return "u01"

@pytest.fixture
def user(user_id: str) -> dict[str, str]:
    return {"id": user_id, "name": "Naoya"}
```

→ fixture 間の依存関係を**型付きで安全に記述できる**

---

### 📝`conftest.py` に定義するfixtureも型ヒント対応

```python
# tests/conftest.py
import pytest

@pytest.fixture
def sample_book() -> dict[str, str]:
    return {"title": "吾輩は猫である", "author": "夏目漱石"}
```

```python
# test_library.py
def test_book(sample_book: dict[str, str]) -> None:
    assert sample_book["author"] == "夏目漱石"
```

---

### 📝`TypedDict` × テストの強力な組み合わせ

```python
from typing import TypedDict

class BookDict(TypedDict):
    title: str
    author: str

@pytest.fixture
def book() -> BookDict:
    return {"title": "坊ちゃん", "author": "夏目漱石"}

def test_book(book: BookDict) -> None:
    assert "夏目" in book["author"]
```

* テストデータの構造を **IDE補完・型チェック**できる
* `Literal` や `Enum` で入力値も制限可能（→ 17章）

---

### 📝pytestで `mypy` を使う

mypy を使って、テストコードも含めてチェック可能：

```bash
$ mypy tests/
```

`mypy.ini` に以下を追加：

```ini
[mypy-tests.*]
disallow_untyped_defs = True
```

→ テストコードの型も **本番コードと同様に厳密チェック**できる

---

### 📝pytest × `pyright` の補完メリット

| 項目        | 効果                                  |
| --------- | ----------------------------------- |
| fixture補完 | 戻り値型が補完され、キーや属性に対して誤りを防止            |
| タイポ検出     | 誤ったキー・構造の辞書を渡したときに警告される             |
| Unionの処理  | 型ごとの分岐が正しく補完され、Optionalチェックも適切に促される |

---

### 📝実務での応用

| 対象               | 型ヒント適用例                                     |
| ---------------- | ------------------------------------------- |
| 入力データの構造         | `TypedDict`, `dataclass`, `Enum`などで明示       |
| テスト対象クラス・関数の戻り値型 | `-> str`, `-> list[Book]` など明示的に記述          |
| fixture戻り値       | `-> BookDict`, `-> list[UserDict]` など構造化    |
| パラメータテスト         | `@pytest.mark.parametrize(..., str)` で型補助可能 |

---

### 📝まとめ

| 項目             | 内容                                             |
| -------------- | ---------------------------------------------- |
| テスト関数の戻り値      | `-> None` を明示（補完と意図の伝達）                        |
| fixtureの型注釈    | 戻り値型を定義することで、依存元でも型チェック・補完が可能                  |
| `TypedDict`活用  | テストデータの構造を明確化し、破壊的変更を防止                        |
| mypy/pyright対応 | テストコードも含めて静的検査の対象に含めることで、一貫性ある品質保証が実現          |
| 実務導入の利点        | テスト対象・入力・補完・構造・再利用性を型情報で強化し、**安全なリファクタ**が可能になる |

---
## 📍22章: ドキュメント生成と補完支援

---

### 📝型ヒントはドキュメントの一部である

型ヒントを明示することで、**関数やクラスの仕様がコードから直接読み取れるようになる**ため、
**補完支援・ドキュメント生成・IDEナビゲーション**が大幅に向上します。

```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

* 引数 `name` が `str` 型、戻り値も `str` であることが明確
* `docstring` がなくても補完上で関数仕様が説明される

---

### 📝型ヒントが補完・ナビゲーションに与える影響

| 機能         | 型ヒントによる改善点                     |
| ---------- | ------------------------------ |
| IDE補完      | 引数・戻り値・属性の補完精度が大幅に向上           |
| 定義ジャンプ     | 呼び出し元から正しい関数やクラス定義に移動しやすくなる    |
| refactor支援 | 型の不一致や誤用を防いだ安全なリネーム・変更が可能に     |
| 保守性向上      | 型だけで仕様が説明でき、ドキュメントを都度書き直す必要が減少 |

---

### 📝ドキュメントツールとの連携：Sphinx + autodoc

#### Sphinxとは？

* Python公式推奨のドキュメント生成ツール
* `reStructuredText` または `Markdown` で記述可能
* `autodoc` を用いると **docstringと型ヒントから自動生成**される

#### 型ヒント付きの関数例：

```python
def add(x: int, y: int) -> int:
    """2つの整数を加算します"""
    return x + y
```

→ Sphinxでは以下のような出力に変換：

```plaintext
add(x: int, y: int) → int
    2つの整数を加算します
```

---

### 📝`napoleon` による Google / NumPy 形式のdocstring対応

```python
def multiply(x: int, y: int) -> int:
    """
    Multiply two numbers.

    Args:
        x (int): First number
        y (int): Second number

    Returns:
        int: Product
    """
```

→ `napoleon` 拡張を使うことで、型ヒントと併用して**より明確なドキュメント生成**が可能

---

### 📝補完支援ツール：VSCode, PyCharm, pyright, pylance

| ツール              | 型ヒントの扱い                         |
| ---------------- | ------------------------------- |
| VSCode + Pylance | 関数・変数・引数の型を即時補完、定義へジャンプも対応      |
| pyright          | 型ヒントに基づく高速な型チェック、補完支援に優れる       |
| PyCharm          | 型ヒントとdocstringの統合補完、メソッド候補提示に強い |

---

### 📝docstring + 型ヒントの両立

| アプローチ           | 内容                                     |
| --------------- | -------------------------------------- |
| 型ヒントのみに依存する     | 引数や戻り値の型はすべて型ヒントに書き、docstringは説明だけにする  |
| docstringにも型を書く | 型チェックが不要な場合や、説明目的に型を書く（やや重複）           |
| napoleonスタイルを使う | Google形式やNumPy形式のdocstringで統一感のある記述が可能 |

---

### 📝クラス定義の補完例（TypedDictやdataclass）

```python
from typing import TypedDict

class Book(TypedDict):
    title: str
    author: str
```

* 補完では `book["title"]` のようにキー名と型が即時に提示される
* フィールド変更時の追従も正確

---

### 📝実務での恩恵

| 作業対象     | 型ヒント・補完がもたらす支援                                    |
| -------- | ------------------------------------------------- |
| 関数開発     | 型情報が自動補完され、誤使用が起きにくい                              |
| コードレビュー  | 引数や戻り値の型から仕様が読み取れ、設計の意図が明確になる                     |
| ドキュメント生成 | `sphinx` + `autodoc` によってdocstringと型ヒントを一体化して出力可能 |
| ライブラリ利用側 | 呼び出し側でも IDE補完が効くため、**仕様を知らずとも安全に使える**             |

---

### 📝まとめ

| 項目         | 内容                                                  |
| ---------- | --------------------------------------------------- |
| 型ヒントの役割    | 仕様の明示、補完支援、ナビゲーション、ドキュメント代替                         |
| Sphinxとの連携 | `autodoc`・`napoleon` によって docstringと統合したドキュメント生成が可能 |
| IDE補完の精度   | `pyright`, `Pylance`, `PyCharm` により構造補完・型警告が即時に機能   |
| 実務的価値      | コーディング・レビュー・文書化の全領域で設計の透明性と保守性が向上                   |

---
## 📍23章: プロジェクト設計と型

---

### 📝型ヒントは設計そのものである

型ヒントは「**実装を制約する情報**」であると同時に、「**設計の意図をコードに埋め込む手段**」でもあります。

* 型によってレイヤ間の責務を明確にする
* 入出力の構造が設計段階で確定する
* 単体テストやリファクタの安全性が高まる

---

### 📝レイヤ別に型をどう活用するか

| レイヤ       | 型ヒントの役割                       | 利用例                                             |
| --------- | ----------------------------- | ----------------------------------------------- |
| UI層（View） | 入力バリデーション、表示対象の構造明示           | `TypedDict`, `Enum`, `Literal`                  |
| アプリケーション層 | ユースケースの入出力、データ構造の変換           | `dataclass`, `Protocol`, `TypeVar`              |
| ドメイン層     | ビジネスルールの定義、エンティティ型の制約         | `Final`, `ClassVar`, `Self`, `NewType`          |
| インフラ層     | DB/APIなどの外部構造との変換、非構造データの受け入れ | `Any`, `cast`, `TypedDict`, `Optional`, `Union` |

---

### 📝型ヒントを中心に設計するとは？

#### 型駆動設計（Type-Driven Design）

* **「この関数は何を受け取り、何を返すか」を最初に型で定義**
* 実装より先に、型ヒントを通じて**責務・仕様・制約を設計**
* 実装が後から変わっても、**型が変わらなければ依存側は壊れない**

---

### 📝型でインターフェースを制約する

```python
class Repository(Protocol):
    def save(self, data: dict[str, Any]) -> None: ...
    def load(self, key: str) -> dict[str, Any]: ...
```

* 実装はどんなクラスでもよいが、「この構造を満たすこと」が契約になる（→Protocol）

---

### 📝データ入出力に型を定義する

#### APIレスポンスの例：

```python
class UserResponse(TypedDict):
    id: str
    name: str
    email: str
```

* JSONレスポンスの受け取りと検証を安全に行える
* テストデータ生成にも使い回し可能

---

### 📝DBやI/O境界では `Any` を使い、すぐに明確化

```python
def load_json(path: str) -> Any:
    ...

def parse_user(data: Any) -> UserResponse:
    ...
```

* 外部から入ってくる曖昧なデータには `Any` を使いつつ、
* **内部に持ち込む前に厳密な型に変換することが重要**

---

### 📝型によるドキュメント化・自動補完・保守性強化

| 利点           | 説明                                                  |
| ------------ | --------------------------------------------------- |
| 仕様の明示性       | 関数・クラスを読むだけで「何を渡せばよいか」が明確になる                        |
| 自動補完         | 引数のキー、型、戻り値の候補がIDE上に出る                              |
| 型変更時の検出      | 型を変えたらすぐに依存側のエラーが分かり、**破壊的変更を防止**できる                |
| テストの構造にも活用可能 | `TypedDict`, `Enum`, `Literal` を使って**データ構造の再利用**が可能 |

---

### 📝型ヒントによるプロジェクト設計戦略

| 原則                  | 説明                                                   |
| ------------------- | ---------------------------------------------------- |
| 入力は必ず型をつける          | 引数・ファイル読込・環境変数などの「外部入力」は最優先で型を明示                     |
| 出力型も必ず定義する          | 返すデータ・構造の変化は呼び出し側に影響するため、戻り値型は必須                     |
| 境界で型を変換する           | I/OやAPIとの境界では `Any` → `TypedDict` 等への**即時型変換**を行う    |
| 曖昧な構造を内部に持ち込まない     | `dict[str, Any]` のまま内部ロジックに渡すと、**型安全性が崩壊**する         |
| 単位テスト・統合テストも型ベースで構成 | テストデータも `TypedDict` や `Literal` で記述して**仕様との整合を担保**する |

---

### 📝まとめ

| 項目           | 内容                                              |
| ------------ | ----------------------------------------------- |
| 型中心設計とは？     | 関数やクラスの**入出力を先に型定義し、そこから設計・実装を進める**方法論          |
| レイヤ別の型役割     | UI:構造明示、アプリ層:仕様制約、ドメイン:意味づけ、インフラ:境界変換           |
| 型によるインターフェース | `Protocol`, `TypedDict`, `Enum` による構造定義と契約化     |
| Anyの扱い       | 入力では許容しつつ、**内部ロジックには明示型を通す**ことで設計の安全性を保つ        |
| 型ヒントの設計的効果   | ドキュメント代替・補完精度向上・リファクタ安全性・設計意図の自動伝達といった多面的な恩恵がある |

---
## 📍24章: `typing_extensions` の活用

---

### 📝`typing_extensions` とは何か？

`typing_extensions` は、**Python本体にまだ組み込まれていない型ヒント機能を先行して提供するライブラリ**です。

* 古いPythonバージョンでも最新の型ヒントを使える
* `Literal`, `TypedDict`, `Final`, `Self`, `NotRequired` などが段階的に本体入りしてきた経緯あり
* **型互換性の維持・モダン機能の早期利用**が目的

```bash
$ pip install typing_extensions
```

---

### 📝代表的な拡張型と対応バージョン

| 型名            | Python本体導入バージョン | `typing_extensions`での提供 | 説明                           |
| ------------- | --------------- | ----------------------- | ---------------------------- |
| `Literal`     | 3.8             | ✅                       | 限定された値のみを許容                  |
| `Final`       | 3.8             | ✅                       | 再代入禁止・オーバーライド防止              |
| `TypedDict`   | 3.8             | ✅（3.8未満では拡張経由）          | 構造型辞書の定義                     |
| `NotRequired` | 3.11            | ✅                       | `TypedDict` の部分フィールドを定義      |
| `Required`    | 3.11            | ✅                       | `total=False` 下での強制フィールド指定   |
| `Self`        | 3.11            | ✅                       | 自クラスインスタンス自身の型表現（→15章）       |
| `Unpack`      | 3.11            | ✅                       | タプル型の分解やジェネリクス展開で使用          |
| `Annotated`   | 3.9             | ✅                       | 型情報に追加メタ情報を添付（バリデーション・説明などに） |

---

### 📝`NotRequired`, `Required` の活用

#### 例：一部任意フィールドの構造

```python
from typing_extensions import TypedDict, NotRequired, Required

class UserData(TypedDict, total=False):
    id: Required[str]
    name: str
    age: NotRequired[int]
```

* `id` は必須、`age` は任意、`name` は全体として optional
* **粒度の細かい型制御**が可能になり、JSON構造の柔軟な表現に強い

---

### 📝`Self` 型のバックポート

```python
from typing_extensions import Self

class Book:
    def set_title(self, title: str) -> Self:
        self.title = title
        return self
```

* Python 3.11未満でも `Self` を使った **メソッドチェーン設計** が可能

---

### 📝`Unpack` の活用（高度）

```python
from typing_extensions import Unpack, TypedDict

class Params(TypedDict):
    x: int
    y: int

def func(**kwargs: Unpack[Params]) -> None:
    ...
```

→ 関数のキーワード引数に `TypedDict` の中身を安全に展開できる
（Pydantic・FastAPIなどで活用）

---

### 📝`Annotated` で型＋説明・バリデーションを統合

```python
from typing_extensions import Annotated

Name = Annotated[str, "ユーザーのフルネーム"]
```

→ 今後は `pydantic`, `FastAPI`, `datamodel-code-generator` などでも標準的に利用される予定

---

### 📝実務での導入指針

| シーン                  | 活用例                                     |
| -------------------- | --------------------------------------- |
| 複雑な `TypedDict` 定義   | `Required`, `NotRequired` で柔軟に対応        |
| チェーンメソッド設計           | `Self` により戻り値型が崩れない                     |
| 共通ユーティリティの再利用        | `Literal`, `Final` による制限付き設定            |
| Pythonバージョンを問わない開発環境 | `typing_extensions` により**将来機能を前倒しで使える** |

---

### 📝まとめ

| 概念                  | 内容                                                                       |
| ------------------- | ------------------------------------------------------------------------ |
| `typing_extensions` | 最新型ヒントを古いバージョンでも使える互換ライブラリ                                               |
| 提供機能                | `Literal`, `TypedDict`, `Self`, `Final`, `NotRequired`, `Annotated` など多数 |
| 実用的メリット             | 型安全性・補完性を損なわずに最新の構造型・構文を利用可能                                             |
| 利用推奨シーン             | Python 3.8以前対応、柔軟な構造型定義、高度な型注釈設計、高精度なライブラリ構築                             |

---
