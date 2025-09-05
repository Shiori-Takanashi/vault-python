## 1章: エラーと例外の違い

---

### エラーと例外の分類

Pythonにおいて「エラー（Error）」と「例外（Exception）」は密接に関係していますが、厳密には異なる概念です。

| 種類                     | 対象                | 例                  | 捕捉可能か             |
| ---------------------- | ----------------- | ------------------ | ----------------- |
| **構文エラー**（SyntaxError） | プログラムの文法違反        | `if x = 3`（=の誤用）   | ❌（実行前に発生）         |
| **例外**（Exception）      | 実行時の異常動作（予測可能な問題） | `int("abc")`（変換失敗） | ⭕（`try-except`で可） |

---

### 構文エラー（SyntaxError）

* **発生タイミング**：Pythonのインタプリタがスクリプトを**実行前に構文解析**する段階で検出

* **例**：

  ```python
  for i in range(5)
      print(i)
  ```

  ⟶ `SyntaxError: expected ':'`

* `^`（キャレット）は、**問題のある箇所**を示す（後述）

---

### 例外（Exception）

* **発生タイミング**：スクリプトの**実行中**に起こる

* **例**：

  ```python
  x = 1 / 0
  ```

  ⟶ `ZeroDivisionError: division by zero`

* 通常、`try-except`構文で処理を継続できる

---

### コンパイルエラー vs 実行時エラー

Pythonは**インタプリタ型**ですが、内部的に**一時的にバイトコードへコンパイル**されます。そのため、エラーには以下のような分類があります：

| 区分        | 説明                       | 例                                 |
| --------- | ------------------------ | --------------------------------- |
| コンパイル時エラー | インタプリタがソースをバイトコードに変換できない | `SyntaxError`, `IndentationError` |
| 実行時エラー    | 実行中に発生する異常処理             | `ValueError`, `KeyError` 等        |

---

### なぜ区別が重要か？

* **構文エラー**は**事前に修正**が必要（コードがまったく動かない）
* **例外**は**状況に応じてハンドリング**できる（柔軟な設計が可能）

例：

```python
try:
    result = int(input("数値を入力: "))
except ValueError:
    print("数値を入力してください。")
```

---

### 補足：Exceptionも「Error」を名乗る

Pythonの標準例外は多くが「〇〇Error」という名前を持ちますが、これは**命名の慣習**であり、すべて`Exception`のサブクラスです。`SyntaxError`もまた、`Exception`のサブクラスです（正確には`BaseException`系譜の下位）。

---

### まとめ

* `SyntaxError`：構文の誤り → **実行すらされない**
* `Exception`：実行中のエラー → **対処すれば継続可能**
* 区別を理解することが、**デバッグ効率とコード保守性の第一歩**となる

---
## 2章: 構文エラー（SyntaxError）の読み方

---

### 構文エラーとは何か？

構文エラー（`SyntaxError`）は、**Pythonの文法ルールに違反しているコード**に対して発生します。これはプログラムの**実行前**に検出されるため、構文エラーがあるとスクリプトは一切実行されません。

---

### エラーメッセージの基本構造

典型的な構文エラーの出力例：

```python
for i in range(3)
    print(i)
```

出力されるエラーメッセージ：

```
  File "sample.py", line 1
    for i in range(3)
                     ^
SyntaxError: expected ':'
```

---

### `^` の位置の意味

* キャレット（`^`）は、**エラーが検出された箇所**を示します。
* 通常は**その前の構文要素に問題**があると考えるのが妥当です。

上記例では、`for i in range(3)` の直後に `:` が必要であるため、`SyntaxError: expected ':'` という指摘になります。

---

### よくある構文エラーのパターン

#### 1. コロン（`: `）の付け忘れ

```python
def hello()
    print("Hello")
```

```
SyntaxError: expected ':'
```

#### 2. 括弧の閉じ忘れ

```python
print("Hello"
```

```
SyntaxError: unexpected EOF while parsing
```

#### 3. インデントの誤り

```python
def greet():
print("Hi")
```

```
IndentationError: expected an indented block
```

#### 4. 不正な識別子や予約語の誤用

```python
class = "MyClass"
```

```
SyntaxError: invalid syntax
```

---

### `SyntaxError` と `IndentationError`

* `IndentationError` は `SyntaxError` の一種
* インデント（字下げ）に特化した構文エラーであり、以下のように派生関係を持つ：

```text
BaseException
 └── SyntaxError
      └── IndentationError
```

---

### スクリプト全体を対象にしたエラー診断の流れ

1. 実行して `SyntaxError` が出たら、**該当行と直前の行を確認**
2. `^` の位置を見て、**その直前の記号やキーワードの使い方を再確認**
3. コロン・括弧・インデント・予約語・クォートなどを**体系的に確認**

---

### 実行時エラーとの違い

| 項目      | 構文エラー（SyntaxError）   | 実行時エラー（例外）                        |
| ------- | -------------------- | --------------------------------- |
| 検出タイミング | 実行前                  | 実行中                               |
| 原因      | 文法ルール違反              | データや状態に依存した異常                     |
| 例       | `missing ':'`, `EOF` | `ValueError`, `ZeroDivisionError` |

---

### まとめ

* `SyntaxError` は**最初に直面するエラー**であり、**修正しないと実行不能**
* `^` の**直前の要素**に注目し、**パターンを覚えることが有効**
* 初期段階では、**インデント・コロン・括弧**に絞って検出練習をすると効果的

---
## 3章: 例外（Exception）の基本構造

---

### Pythonにおける例外とは？

例外（Exception）は、**実行中に発生する異常な状況**を示す仕組みです。たとえば、存在しない変数の参照や不適切な型の操作など、通常の処理の流れを中断させる出来事を表します。

```python
x = int("abc")
# ValueError: invalid literal for int() with base 10: 'abc'
```

---

### Tracebackの読み方（基本構造）

エラー発生時には「Traceback（トレースバック）」という形式でエラー情報が出力されます：

```python
Traceback (most recent call last):
  File "example.py", line 1, in <module>
    x = 1 + "2"
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

| 行   | 内容                                    |
| --- | ------------------------------------- |
| 1行目 | Tracebackの開始。**直近の呼び出しからさかのぼって表示**される |
| 2行目 | エラー発生箇所のファイル・行番号・関数名の情報               |
| 3行目 | 実行されたコードの該当行（またはその一部）                 |
| 最終行 | **エラーの種類と説明文**                        |

---

### 主な組み込み例外の違いと意味

#### `TypeError`

* **型の不一致**によるエラー
* 例：

  ```python
  1 + "2"
  ```

  ⟶ `TypeError: unsupported operand type(s) for +: 'int' and 'str'`

#### `ValueError`

* **型は正しいが、値が不正**
* 例：

  ```python
  int("abc")
  ```

  ⟶ `ValueError: invalid literal for int() with base 10: 'abc'`

#### `NameError`

* **未定義の変数を使用**
* 例：

  ```python
  print(undefined_variable)
  ```

  ⟶ `NameError: name 'undefined_variable' is not defined`

---

### 例外処理構文の導入

これらの例外は `try-except` によって**安全に処理可能**です。

```python
try:
    x = int(input("数字を入力: "))
except ValueError:
    print("数字ではありません。")
```

---

### 補足：Exceptionの階層と役割

例外はクラス構造で定義されており、すべて `BaseException` を起点とした階層構造を持っています：

```text
BaseException
 └── Exception
      ├── TypeError
      ├── ValueError
      ├── NameError
      └── ...
```

`BaseException` は通常のアプリケーションでは捕捉せず、`Exception`以下を対象にするのが原則です。

---

### 実用視点での例外の重要性

* ユーザー入力の検証
* ファイル・ネットワークの操作
* 外部APIとの連携
* ライブラリとのインタフェース設計

いずれも例外処理の有無で**信頼性が大きく変わる**

---

### まとめ

* `TypeError` は「型の不一致」、`ValueError` は「値の不正」、`NameError` は「変数未定義」
* Tracebackは**最下行が最も重要**（例外の種類と説明）
* `try-except` を用いて**例外発生時にも処理を継続可能**
* Pythonの例外は**クラスとして階層化**されているため、**設計上のカスタマイズも可能**

---
## 4章: 例外クラスの階層構造

---

### Pythonの例外はすべてクラスである

Pythonの例外は**クラスベースのオブジェクト**であり、すべての例外は `BaseException` を起点とした\*\*階層構造（継承関係）\*\*を持ちます。これは例外の種類ごとに整理・制御しやすくするための設計です。

---

### 階層構造の概要（抜粋）

```text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   └── OverflowError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── TypeError
    ├── ValueError
    ├── NameError
    └── ...
```

---

### `BaseException` vs `Exception`

| クラス名            | 用途                  | 備考       |
| --------------- | ------------------- | -------- |
| `BaseException` | 全例外の親、**特殊な例外用**    | 通常は捕捉しない |
| `Exception`     | **一般的なアプリケーション用例外** | 通常はこれを継承 |

`SystemExit` や `KeyboardInterrupt` などは `BaseException` 直下であり、**try-exceptでは原則捕捉しない**方がよい例外です。

---

### `ArithmeticError` と `LookupError` の役割

---

#### `ArithmeticError`

* 算術演算に関連する例外の親クラス
* 具体例：

  * `ZeroDivisionError`: 0で割った
  * `OverflowError`: 浮動小数点演算の範囲を超えた

#### `LookupError`

* **インデックスやキーの探索失敗**を扱う親クラス
* 具体例：

  * `IndexError`: リストの範囲外アクセス
  * `KeyError`: 辞書に存在しないキー

`except LookupError:` のように**上位クラスでまとめて処理**することも可能。

---

### クラス階層が意味するもの

* **上位のクラスを指定すれば複数の例外を一括で捕捉**可能
* ただし、**汎用的すぎる捕捉はデバッグ困難**になる恐れあり

例：

```python
try:
    risky_operation()
except ArithmeticError:
    print("算術演算でエラーが発生しました。")
```

---

### 独自の例外定義におけるベースクラス

自作の例外は `Exception` を継承するのが通例：

```python
class MyCustomError(Exception):
    pass
```

`BaseException` を直接継承するのは、**システム終了や中断などの重大な処理を扱うときのみ**

---

### 例外を階層として理解するメリット

* **意図的に抽象度を調整してエラーを捕捉**できる
* ライブラリ開発において、**独自例外を汎用的に扱える**
* `isinstance(e, LookupError)` のように**動的に種類を確認可能**

---

### まとめ

* 例外は `BaseException` を起点とする**階層構造**
* 通常は `Exception` 以下のみを捕捉対象とする
* `ArithmeticError` や `LookupError` は**中間階層の抽象クラス**
* 上位クラスで**まとめてハンドリング**することで柔軟な設計が可能

---
## 5章: try-except構文の基本

---

### try-exceptとは何か？

`try-except` は、**例外が発生する可能性のある処理を安全に実行**するための基本構文です。Pythonにおける**例外処理の中核**を成す構文であり、これを正しく理解することでプログラムの堅牢性を大きく向上させることができます。

---

### 最低限の構文

```python
try:
    # エラーが発生するかもしれない処理
    x = int(input("数値を入力: "))
except ValueError:
    # エラーが発生した場合の処理
    print("数値ではありません。")
```

* `try` 節で実行される処理に例外が発生すると、
* ただちに対応する `except` 節にジャンプし、**処理を続行可能**になります。

---

### 複数の `except` 節

Pythonでは、**異なる種類の例外ごとに異なる処理を記述**できます：

```python
try:
    lst = [1, 2, 3]
    print(lst[5])
except IndexError:
    print("インデックス範囲外です。")
except ValueError:
    print("値の変換に失敗しました。")
```

* 上から順に評価され、**最初に一致した except が実行**されます。

---

### `except (A, B):` の構文

複数の例外を**まとめて同じ処理にしたい場合**：

```python
try:
    ...
except (ValueError, TypeError):
    print("値または型に関するエラーです。")
```

* タプルで複数の例外クラスを指定

---

### `else` 節の活用

* `try` 節が**例外なく成功した場合のみ** `else` が実行されます。

```python
try:
    x = int(input("数値を入力: "))
except ValueError:
    print("無効な数値です。")
else:
    print(f"{x} の2倍は {x * 2} です。")
```

* **成功時の処理を明確に分離**できる

---

### `finally` 節の役割

* **例外の有無に関係なく**、最後に必ず実行される処理です。

```python
try:
    f = open("sample.txt")
    content = f.read()
except FileNotFoundError:
    print("ファイルが見つかりません。")
finally:
    f.close()
    print("ファイルを閉じました。")
```

* ファイル・接続・ロックなど、**リソースの後処理に必須**

---

### 組み合わせのフロー

```text
try:
    # 本処理
except:
    # 例外発生時のみ
else:
    # 例外がなければ
finally:
    # 必ず実行
```

---

### よくある落とし穴

| 誤り例                         | 問題点                        |
| --------------------------- | -------------------------- |
| `except:`（例外クラス未指定）         | すべての例外を捕捉し、**バグの発見を困難にする** |
| `try` の範囲が広すぎる              | **何が例外原因か分からなくなる**         |
| `finally` で `close()` 等を忘れる | \*\*リソースリーク（未解放）\*\*が発生    |
| `else` に処理を書かずに済ませる         | 成功時と例外処理の**分離が不明確**になる     |

---

### まとめ

* `try-except` は例外発生時の**安全な処理継続のための仕組み**
* `else` は正常時、`finally` は後処理に使う
* **複数exceptやタプル指定**で柔軟な例外対応が可能
* \*\*構文全体の構成（try-except-else-finally）\*\*を理解して初めて、本格的なエラーハンドリングに踏み出せる

---
## 6章: 組み込み例外一覧と代表的な発生要因

---

### Pythonの例外はほぼすべて「組み込み例外」で構成されている

Pythonには、標準で用意されている例外クラスが多数あります。これらはすべて `Exception` を継承しており、**想定されうる異常状態を細かく分類**しています。代表的なものの発生タイミングとあわせて学ぶことで、**例外を事前に防ぐ設計**や、**適切に捕捉する処理**が可能になります。

---

### 算術・数値系

| 例外クラス                | 発生例                        | 説明                      |
| -------------------- | -------------------------- | ----------------------- |
| `ZeroDivisionError`  | `1 / 0`                    | ゼロ除算                    |
| `OverflowError`      | `math.exp(1000)`           | 浮動小数点が表現限界を超える（演算の結果）   |
| `FloatingPointError` | 非常にまれ／`sys.seterr()`有効時に発生 | 浮動小数点の内部エラー（基本は抑制されている） |

---

### 型・値に関するもの

| 例外クラス                 | 発生例              | 説明                       |
| --------------------- | ---------------- | ------------------------ |
| `TypeError`           | `1 + "2"`        | 型が合わない演算や関数呼び出し          |
| `ValueError`          | `int("abc")`     | 型は合っているが**値が不適切**        |
| `AssertionError`      | `assert 2 > 3`   | `assert` による明示的な失敗       |
| `NotImplementedError` | メソッドに `raise` 明示 | サブクラスに実装させるための**意図的な例外** |

---

### 名前・属性・参照

| 例外クラス               | 発生例                    | 説明                |
| ------------------- | ---------------------- | ----------------- |
| `NameError`         | `print(undefined_var)` | 定義されていない変数を参照     |
| `AttributeError`    | `"abc".append(1)`      | 存在しない属性やメソッドを呼び出し |
| `UnboundLocalError` | `print(x); x = 1`      | ローカル変数が参照時に未代入    |

---

### シーケンス・辞書アクセス

| 例外クラス           | 発生例              | 説明                      |
| --------------- | ---------------- | ----------------------- |
| `IndexError`    | `[1, 2][5]`      | リスト等の**範囲外インデックス**      |
| `KeyError`      | `{"a": 1}["b"]`  | 辞書に存在しないキー              |
| `StopIteration` | `next(iter([]))` | イテレータの終端（`for`内では無視される） |

---

### 入出力・ファイル系

| 例外クラス               | 発生例                  | 説明                     |
| ------------------- | -------------------- | ---------------------- |
| `FileNotFoundError` | `open("nofile.txt")` | 存在しないファイルを開こうとした       |
| `PermissionError`   | 書き込み禁止のファイルに出力       | OSレベルのパーミッション違反        |
| `IsADirectoryError` | ディレクトリを `open()` で扱う | ファイルであるべき対象がディレクトリ     |
| `IOError`           | OSやデバイスとの低レベル通信時の異常  | `OSError` に統合されたが互換性あり |

---

### その他のよくあるもの

| 例外クラス                 | 説明・備考                        |
| --------------------- | ---------------------------- |
| `ImportError`         | モジュールの読み込みに失敗                |
| `ModuleNotFoundError` | 上記のサブクラス。モジュール自体が見つからない場合に限定 |
| `RuntimeError`        | **特定の分類に属さない一般的な実行時エラー**     |
| `RecursionError`      | 関数の再帰が**最大スタック深さ**を超えた場合     |
| `KeyboardInterrupt`   | Ctrl+C などによる中断処理             |

---

### 組み込み例外一覧の活用方法

* `try-except` で**必要な例外のみを捕捉**するために分類を理解する
* `except Exception as e:` などで `type(e)` を調べて分類を知る
* コーディング前に\*\*「この処理で起こりうる例外は何か？」を逆算\*\*する習慣をつける

---

### まとめ

* Pythonには**数十種類の組み込み例外**があり、**状況ごとに分類されている**
* `TypeError`, `ValueError`, `KeyError`, `IndexError` は**最頻出例外**
* 一つひとつの例外の「発生条件」を覚えることが**デバッグ力向上の鍵**
* 例外分類は、**設計・保守・ログ管理・テスト**など全工程において重要

---
## 7章: 実行時の型エラー

---

### `TypeError` とは？

`TypeError` は、**「型の不一致」に起因する実行時エラー**です。関数呼び出し・演算・メソッド使用など、**異なる型のオブジェクトを不適切に組み合わせたとき**に発生します。

```python
x = 1 + "2"
# TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

---

### 主な発生パターンと原因分析

#### ① 異なる型の演算

```python
result = "abc" + 123
# TypeError: can only concatenate str (not "int") to str
```

* `+` 演算子は文字列同士／数値同士にしか使えない
* 暗黙的な変換は行われない（`JavaScript`との違い）

**対処法：**

```python
result = "abc" + str(123)
```

---

#### ② 関数やメソッドへの誤った型の引数

```python
round("3.14")
# TypeError: type str doesn't define __round__ method
```

* `round()` は数値を想定しており、文字列は非対応

**対処法：**

```python
round(float("3.14"))
```

---

#### ③ 呼び出し可能でないオブジェクトを実行しようとした

```python
x = 42
x()
# TypeError: 'int' object is not callable
```

* `()` は「呼び出し可能（callable）」な対象にしか使えない

---

#### ④ イテラブルでないものを `for` で回す

```python
for c in 123:
    print(c)
# TypeError: 'int' object is not iterable
```

* `for` 文は `__iter__()` を持つオブジェクトにしか使えない

---

#### ⑤ 型不一致な比較・ソート

```python
[1, "2", 3].sort()
# TypeError: '<' not supported between instances of 'str' and 'int'
```

* Python 3 では異なる型同士の大小比較は不可

---

### 関数の引数チェックの例

明示的な型検査で `TypeError` を**防止または明示**できます：

```python
def repeat_text(text: str, times: int) -> str:
    if not isinstance(text, str):
        raise TypeError("textはstr型でなければなりません")
    if not isinstance(times, int):
        raise TypeError("timesはint型でなければなりません")
    return text * times
```

---

### `TypeError` を捕捉して処理継続

```python
try:
    result = 1 + "2"
except TypeError as e:
    print(f"型エラーが発生しました: {e}")
```

* エラー内容をログに残し、処理を止めない設計が可能

---

### Pythonの「型」の理解が防御力になる

| 処理対象      | 求められる型                 | 例外発生例           |
| --------- | ---------------------- | --------------- |
| `+` 演算    | 両方が `str` または `int`    | `"abc" + 123`   |
| `()` 呼び出し | `function`, `class` など | `None()`        |
| `for` ループ | イテラブル（`list`, `str`）   | `for x in 100:` |
| `len()`   | `__len__()` 実装あり       | `len(10)`       |

---

### まとめ

* `TypeError` は**型の誤用**による最も基本的な例外
* 原因の大半は**演算・引数・呼び出し・ループ対象の誤り**
* 対策として、**事前の `isinstance()` チェック**や**try-except**が有効
* Pythonは**動的型付けだが、型の意識は常に求められる**

---
## 8章: 値に関するエラー（ValueError）

---

### `ValueError` とは何か？

`ValueError` は、\*\*「型は正しいが、値が不正なとき」\*\*に発生する例外です。つまり、関数や操作の対象として「期待される型」であっても、**中身の値が処理不能な場合**に起こります。

```python
int("abc")
# ValueError: invalid literal for int() with base 10: 'abc'
```

---

### `ValueError` の主な発生パターン

#### ① 文字列から数値への変換失敗

```python
float("3.14")      # OK
float("abc")       # NG → ValueError
```

* 対象は `str` 型で正しいが、値が数値として不正

---

#### ② range() などへの不適切な引数

```python
list(range(5, 1))  # → []
list(range(5, 1, 1))
# ValueError: range() arg 3 must not be zero
```

---

#### ③ `.index()` に存在しない値を指定

```python
[1, 2, 3].index(4)
# ValueError: 4 is not in list
```

* 型は合っているが、**値がリストに存在しない**

---

#### ④ アンパック処理の不一致

```python
x, y = [1, 2, 3]
# ValueError: too many values to unpack (expected 2)
```

---

#### ⑤ `int()` や `float()` に不適切なフォーマット

```python
int("３")  # 全角 → ValueError
int("")   # 空文字 → ValueError
```

---

### `ValueError` と `TypeError` の違い

| 例外           | 原因           | 発生例          |
| ------------ | ------------ | ------------ |
| `TypeError`  | 型が想定外        | `"3" + 1`    |
| `ValueError` | 型は合ってるが値が想定外 | `int("abc")` |

---

### 事前検証の技法：try-except と明示チェック

#### 方法① try-except による回避

```python
try:
    x = int(user_input)
except ValueError:
    print("数値として認識できません")
```

#### 方法② 事前に検証関数を使う

```python
def is_int(s: str) -> bool:
    try:
        int(s)
        return True
    except ValueError:
        return False
```

---

### なぜバリデーションチェックが有効なのか？

1. **処理の意図を明示できる**
   `try-except` だけに頼ると、どの値が許されるかが不透明になります。事前の値検証によって、\*\*「どこまでが正常か」\*\*をコード上に明示できます。

2. **バグの早期発見につながる**
   例外が起こった後の復旧より、**例外を起こさない方が安全**です。入力チェックによって不正値の通過を未然に防ぎ、後続処理の安全性を高められます。

3. **テストやロギングがしやすくなる**
   値の不正検知を明示的に設けておくことで、**ユニットテストでの検証が容易**になります。また、検証結果をログ出力しやすくなります。

4. **ユーザー体験の向上**
   GUI・CLI いずれにおいても、「なぜダメなのか」を明示できるため、**例外発生による中断よりも滑らかな対応**が可能になります。

---

### データ検証（Validation）の具体例

| チェック対象     | チェック項目        | 例                               |
| ---------- | ------------- | ------------------------------- |
| ユーザー入力     | 数値変換可能か       | `isdigit()`, `try-int`          |
| 配列長        | 必要数だけあるか      | `len(lst) >= 2`                 |
| 辞書やリストの中身  | 特定のキーや値が存在するか | `"key" in d`, `v in lst`        |
| ファイルや引数の内容 | 空でない・形式が正しいか  | `.strip() != ""`, `.endswith()` |

---

### 不適切な `ValueError` を起こさない設計

* **検証を呼び出し元で行う**
* 引数が想定通りかどうかを **早期に明示的に確認**
* Pythonでは「**EAFP原則（まず試して、失敗したら処理）**」が推奨される場面も多いが、`ValueError` に関しては\*\*事前チェック（LBYL原則）\*\*の方が適している場面が多い

---

### まとめ

* `ValueError` は「**型はOK、値がNG**」という例外
* ユーザー入力やデータ整形処理で**最も頻出する例外の一つ**
* `try-except` だけでなく、**入力検証・事前チェック**が極めて重要
* バリデーションは**安全性・保守性・UX・テスト性**のすべてを向上させる設計上の鍵

---
## 9章: KeyErrorとIndexError

---

### 両者は「アクセス失敗」による例外だが、対象が異なる

| 例外           | 対象                        | 発生条件              |
| ------------ | ------------------------- | ----------------- |
| `KeyError`   | `dict`（辞書）                | 存在しないキーを指定した場合    |
| `IndexError` | `list` や `tuple` などのシーケンス | 範囲外のインデックスを指定した場合 |

---

### `KeyError` の発生例と対処法

#### 発生例：

```python
d = {"name": "Pikachu"}
print(d["type"])
# KeyError: 'type'
```

#### よくある誤解：

* `"type" in d` → `False`
* でも `d["type"]` → `KeyError`

#### 回避法①：in句による存在確認

```python
if "type" in d:
    print(d["type"])
else:
    print("未定義")
```

#### 回避法②：`get()` を使う

```python
print(d.get("type", "不明"))
# ⇒ '不明'
```

---

### `IndexError` の発生例と対処法

#### 発生例：

```python
lst = [10, 20, 30]
print(lst[3])
# IndexError: list index out of range
```

* インデックス3は存在しない（0,1,2まで）

#### 回避法①：長さをチェック

```python
if len(lst) > 3:
    print(lst[3])
```

#### 回避法②：`try-except`で捕捉

```python
try:
    print(lst[3])
except IndexError:
    print("インデックス範囲外です")
```

---

### 両者の違いを深く理解する

| 特徴        | KeyError               | IndexError                |
| --------- | ---------------------- | ------------------------- |
| 対象構造      | `dict`, `set`（主にdict）  | `list`, `tuple`, `str` など |
| 存在確認の方法   | `"key" in dict`        | `0 <= i < len(seq)`       |
| 安全なアクセス方法 | `.get("key", default)` | `.pop(i)`, `.get(i)` はない  |
| デバッグ時の注意点 | キーが実際に定義されているか         | 長さとインデックスの対応を確認           |

---

### 典型的な混乱パターン

```python
data = [{"id": 1}, {"id": 2}]
print(data[2]["id"])
# IndexError: list index out of range

print(data[1]["type"])
# KeyError: 'type'
```

* **外側（list）→ IndexError**
* **内側（dict）→ KeyError**

---

### 応用：ループ処理中の保護

```python
for d in data:
    print(d.get("type", "不明"))
```

* KeyErrorを起こさず安全にループ処理可能

---

### まとめ

* `KeyError` は「辞書にキーがない」
* `IndexError` は「リスト等のインデックスが範囲外」
* 両者は**構造の種類によって使い分ける必要がある**
* 安全なアクセス方法（`.get()`, 長さチェック）を習慣化することで、**意図しないクラッシュを防げる**

---
## 10章: FileNotFoundErrorなどファイル系の例外

---

### ファイル操作に関連する例外は非常に多い

Pythonでファイルを開く・読む・書くといった操作を行う際、**対象ファイルやパスの存在、パーミッション、対象の型（ファイル or ディレクトリ）などに応じて様々な例外**が発生します。なかでも最も頻出するのが `FileNotFoundError` です。

---

### FileNotFoundError の概要

```python
with open("nofile.txt") as f:
    content = f.read()
# FileNotFoundError: [Errno 2] No such file or directory: 'nofile.txt'
```

* ファイルパスが存在しない場合に発生
* **タイポ・相対パスミス・作成忘れ**などが原因

#### 回避法①：ファイルの存在を確認してから開く

```python
import os

if os.path.exists("data.txt"):
    with open("data.txt") as f:
        ...
else:
    print("ファイルが見つかりません")
```

#### 回避法②：try-exceptで明示的に捕捉

```python
try:
    with open("data.txt") as f:
        ...
except FileNotFoundError:
    print("ファイルが存在しません")
```

---

### PermissionError の例と対処

```python
with open("/etc/shadow") as f:
    ...
# PermissionError: [Errno 13] Permission denied
```

* **アクセス権限が不足**しているファイルを開こうとしたときに発生
* OSが制御している領域や管理者権限の必要なファイルが対象

---

### IsADirectoryError / NotADirectoryError

```python
open("my_folder")  # フォルダを開こうとすると
# IsADirectoryError

open("path/to/file/data.txt/more.txt")
# NotADirectoryError
```

* パスの途中が**ディレクトリであるべきかどうか**に反している場合に発生
* 特に再帰的なパス構築や、サブディレクトリの展開時に頻出

---

### OSError／IOError の位置づけ

* `OSError` は**OSレベルのファイルシステムエラー**の親クラス
* `FileNotFoundError`, `PermissionError` などはすべて `OSError` のサブクラス
* `IOError` は古いPythonでの名前で、現代のPythonでは `OSError` に統合されているが、後方互換のため残存

---

### ファイル操作における基本チェック項目

| チェック内容     | チェック方法                     | 対応する例外               |
| ---------- | -------------------------- | -------------------- |
| パスが存在するか   | `os.path.exists()`         | `FileNotFoundError`  |
| ファイルであるか   | `os.path.isfile()`         | `IsADirectoryError`  |
| ディレクトリであるか | `os.path.isdir()`          | `NotADirectoryError` |
| 書き込み可能か    | `os.access(path, os.W_OK)` | `PermissionError`    |

---

### 補足：`open()` のモードと例外

| モード   | 概要        | 注意点                             |
| ----- | --------- | ------------------------------- |
| `'r'` | 読み込み      | ファイルが存在しないと `FileNotFoundError` |
| `'w'` | 書き込み（上書き） | 存在しなくても新規作成される                  |
| `'a'` | 追記        | 同上、存在しない場合は作成される                |
| `'x'` | 新規作成のみ    | 既に存在する場合は `FileExistsError`     |

---

### 実務上のポイント

* ファイル処理を含むスクリプトは、\*\*「ファイルがなければエラー」ではなく、「なければ作る／通知する」\*\*設計が望ましい
* 特にログファイル・設定ファイルなどは、**存在チェック → 初期化／作成** という流れが安全

---

### まとめ

* `FileNotFoundError` は**パス誤りや存在しないファイル**で発生
* `PermissionError`, `IsADirectoryError` などもファイル操作で頻出
* 事前チェック (`os.path` 系) または `try-except` による補足が有効
* ファイル系例外はすべて `OSError` 系に集約されており、**網羅的な捕捉も可能**

---
## 11章: with構文と例外処理の連携

---

### with構文の本質

`with` 文は、**リソースの取得と解放を安全かつ簡潔に記述する構文**です。ファイル、ネットワーク接続、ロックなど、**使用後に確実に後処理が必要な操作**に最適です。例外の有無にかかわらず、リソースを**自動的に解放**してくれます。

---

### なぜ `with` 構文が必要なのか？

```python
f = open("data.txt", "r")
try:
    content = f.read()
finally:
    f.close()
```

このような構文を、`with` で簡潔に表せます：

```python
with open("data.txt", "r") as f:
    content = f.read()
# 終了時に自動で f.close() が呼ばれる
```

---

### `with` が例外に強い理由

* `with` 文の背後では、対象のオブジェクトが `__enter__()` と `__exit__()` を実装しており、**例外が発生しても `__exit__()` が必ず実行される**よう設計されています。
* そのため、`finally` を自分で書かなくても、**リソース解放が保証**されます。

---

### ファイル操作での安全な例

```python
try:
    with open("data.txt", "r") as f:
        content = f.read()
except FileNotFoundError:
    print("ファイルが見つかりません")
```

* `open()` に失敗 → `with` は開始されず `FileNotFoundError`
* `read()` に失敗 → `__exit__()` に制御が移り `f.close()` は保証される

---

### 複数ファイルやリソースの同時管理

```python
with open("in.txt") as fin, open("out.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

* `with` 文は**カンマ区切りで複数リソースの同時管理**が可能
* いずれかで例外が出ても、**すべての `__exit__()` が呼ばれる**

---

### カスタムクラスでも活用可能

```python
class Demo:
    def __enter__(self):
        print("開始")
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("終了")

with Demo():
    print("処理中")
```

出力：

```
開始
処理中
終了
```

* `__exit__()` は**例外が発生しても必ず呼ばれる**
* `exc_type` などで例外情報を受け取ることも可能

---

### with構文で例外を握るパターン

```python
class SuppressError:
    def __enter__(self): return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        return True  # 例外を握って無視する

with SuppressError():
    raise ValueError("無視される例外")

print("ここまで到達する")
```

* `__exit__()` が `True` を返すと、**例外は発生しなかったことになる**
* **握り潰す設計は原則避けるが、明示的に使うケースも存在**

---

### `with` を使うべき場面

| 対象      | with推奨理由                              |
| ------- | ------------------------------------- |
| ファイル    | 読み終わり後の `.close()` を保証                |
| ソケット通信  | 接続の `.close()` を漏れなく行える               |
| スレッドロック | `threading.Lock()` で競合防止              |
| 一時ファイル  | `tempfile.NamedTemporaryFile()` の自動削除 |

---

### まとめ

* `with` は **例外が起きてもリソースを確実に解放**する構文
* `try-finally` の**安全性と簡潔さを両立**
* `open()` や `Lock()` を使うときは常に `with` を使う意識を
* カスタムクラスでも `__enter__`, `__exit__` を実装すれば `with` に対応できる

---
## 12章: `raise` による例外の送出

---

### `raise` 文とは？

`raise` は、Pythonにおいて**例外を明示的に発生させる文法**です。処理の中で「これは異常だ」と判断したとき、**制御を即座に中断し、エラーとして通知**するために使用されます。

---

### 基本構文と例

#### 組み込み例外を送出する

```python
raise ValueError("無効な値です")
```

* `ValueError` は標準例外
* メッセージは `str(e)` で取得可能

```python
try:
    raise TypeError("型が違います")
except TypeError as e:
    print(f"例外内容: {e}")
```

---

### 条件による発生制御

```python
def divide(a: float, b: float) -> float:
    if b == 0:
        raise ZeroDivisionError("0では割れません")
    return a / b
```

* **防御的プログラミング**（不正な状態を早期検出）
* `if 条件: raise ～` の構文は **検証 → 例外送出**の基本形

---

### 例外の再送出（中継）

例外を受け取って処理後、**再度上位へ投げることも可能**：

```python
try:
    ...
except ValueError as e:
    print("処理中に例外発生:", e)
    raise  # もとの例外を再送出
```

---

### カスタム例外の導入

独自の例外を定義するには、`Exception` を継承してクラスを作ります。

```python
class InvalidUserInputError(Exception):
    """ユーザー入力が不正な場合に使用"""
    pass
```

#### 使用例：

```python
def validate_age(age: int) -> None:
    if age < 0:
        raise InvalidUserInputError("年齢は0以上である必要があります")
```

---

### なぜカスタム例外を使うのか？

1. **意味的に明確**

   * 単なる `ValueError` ではなく、**何が問題かを例外名で表現できる**

2. **catchを分けて扱える**

   * `except InvalidUserInputError:` のように、**意図した例外だけ捕捉**できる

3. **ライブラリ開発での分界点**

   * 利用者に向けて「これはライブラリ側の定義した例外です」と明示できる
   * 例外のドキュメント化もしやすくなる

---

### カスタム例外の設計指針

* 基本的には `Exception` を継承
* `__init__` や `__str__` をオーバーライドして追加情報を持たせることも可能
* 名前は `...Error` で終えるのが通例（PEP 8準拠）

```python
class UserNotFoundError(Exception):
    def __init__(self, user_id: str):
        super().__init__(f"ユーザーが見つかりません: {user_id}")
        self.user_id = user_id
```

---

### 補足：raiseとreturnの役割分担

| 処理内容          | 使用すべき構文  | 意図                 |
| ------------- | -------- | ------------------ |
| 正常な値の返却       | `return` | 成功した処理の結果          |
| 異常状態の通知（継続不可） | `raise`  | 呼び出し元に**中断と対処**を促す |

---

### まとめ

* `raise` は**処理の強制終了とエラー通知**のための構文
* 組み込み例外を送出することで**即座に異常を伝えられる**
* カスタム例外を導入することで、**文脈に応じた明確なエラー処理設計**が可能に
* 例外を**握る／投げる／再送出する**技術は、堅牢なコードの土台となる

---
## 13章: Tracebackの読み解き

---

### Tracebackとは何か？

Traceback（トレースバック）は、**例外発生時に出力されるエラーの経路情報**です。Pythonは例外が発生すると、それまでに**呼び出された関数の履歴（スタックトレース）を逆順に出力**し、どこで何が起きたかを明示します。

---

### 基本構造の復習

```python
def foo():
    bar()

def bar():
    1 / 0

foo()
```

出力される Traceback：

```
Traceback (most recent call last):
  File "sample.py", line 6, in <module>
    foo()
  File "sample.py", line 2, in foo
    bar()
  File "sample.py", line 5, in bar
    1 / 0
ZeroDivisionError: division by zero
```

---

### 読み解きのポイント

| 要素            | 説明                                                          |
| ------------- | ----------------------------------------------------------- |
| Traceback 見出し | `Traceback (most recent call last):` — **直近の呼び出しが最後に記録**される |
| 各スタックの行       | ファイル名、行番号、関数名、該当コードが順に出力                                    |
| 最終行（例外）       | エラーの種類と説明 — **最も重要**                                        |

* **読み始めは最下行の例外種別から、次にどこで発生したかを上へたどる**
* **再帰やライブラリ呼び出しが含まれる場合、Tracebackが長くなる**ことがある

---

### 複雑な例：多段関数＋try-except なし

```python
def validate(value):
    return int(value)

def process(data):
    return validate(data["age"])

def main():
    d = {"name": "Taro"}
    process(d)

main()
```

```
Traceback (most recent call last):
  File "sample.py", line 10, in <module>
    main()
  File "sample.py", line 8, in main
    process(d)
  File "sample.py", line 5, in process
    return validate(data["age"])
KeyError: 'age'
```

* エラーは最終的に `KeyError: 'age'`
* 原因は `process()` 内の `data["age"]`
* 呼び出し元 `main()` から `process()` → `validate()` と**上から順に関数が呼ばれた経路が記録**

---

### 再帰関数でのTraceback

```python
def recurse(n):
    return recurse(n + 1)

recurse(1)
```

```
RecursionError: maximum recursion depth exceeded
```

* このようなケースでは Traceback が**数百行に及ぶ**
* 読み解きよりも「**構造の問題**」であることを把握するほうが重要

---

### Tracebackをプログラムから取得する

```python
import traceback

try:
    1 / 0
except Exception:
    print(traceback.format_exc())
```

* `traceback.format_exc()` により**Traceback文字列をプログラム上で取得可能**
* ログ出力やGUI表示に応用可能

---

### ログファイルへの保存例

```python
import logging
import traceback

try:
    risky()
except Exception:
    logging.error(traceback.format_exc())
```

* **標準エラーではなくログとして記録することで、後からの調査が可能**

---

### Traceback読み取りチェックリスト

| 項目               | 確認内容                        |
| ---------------- | --------------------------- |
| 最終行              | 例外の種類とメッセージを読む              |
| 発生箇所の関数・行番号      | 実際のファイルを確認してコードを再現          |
| 呼び出し元の関数の履歴      | プログラムのどこから例外に至ったかを確認する      |
| 再帰や無限ループがないか     | 同一関数の繰り返しがある場合は注意する         |
| 外部ライブラリ由来の例外かどうか | `site-packages` などが含まれるかを確認 |

---

### まとめ

* Tracebackは**例外発生の根本原因を特定する最重要情報**
* **最下行が一番重要**、その上から順に**呼び出し経路をたどる**
* 複雑な構成でも、**行番号・関数名・ファイル名**が明示されているため確実に追える
* ログ記録やツール統合によって、**例外原因の可視化と再現性を高めることができる**

---
## 14章: 例外のログ出力

---

### なぜログ出力が重要か？

例外は**即座にユーザーに通知するだけでなく、開発者が後で原因を追跡できるよう記録すること**が重要です。とくにCLIツール・API・バッチ処理・バックグラウンドジョブでは、**即時出力よりも永続的なログ**の方が実用的です。

---

### `print(e)` では不十分

```python
try:
    1 / 0
except ZeroDivisionError as e:
    print(e)  # ← 単に "division by zero" と表示されるだけ
```

* 原因は分かっても「**どこで起きたのか**」が記録されない
* **開発・運用ではスタックトレースが必須**

---

### `traceback` モジュールによる出力

```python
import traceback

try:
    1 / 0
except ZeroDivisionError:
    traceback.print_exc()
```

出力例：

```
Traceback (most recent call last):
  File "main.py", line 4, in <module>
    1 / 0
ZeroDivisionError: division by zero
```

* **print() とは異なり、Traceback全体を出力**
* そのまま `stderr` に出力される

---

### ログファイルに記録する：`logging` × `traceback`

---

#### 構成例①：標準ログへ記録

```python
import logging
import traceback

logging.basicConfig(filename="error.log", level=logging.ERROR)

try:
    raise ValueError("不正な値")
except Exception:
    logging.error("例外発生:\n%s", traceback.format_exc())
```

* `format_exc()` はスタックトレースを**文字列として取得**
* `logging.error()` に渡すことで、**整形済みのログ出力**が可能

---

#### 構成例②：出力先・整形を細かく制御

```python
import logging

logger = logging.getLogger("myapp")
handler = logging.FileHandler("myapp.log", encoding="utf-8")
formatter = logging.Formatter("[%(asctime)s] %(levelname)s - %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

try:
    int("abc")
except Exception:
    import traceback
    logger.error("例外発生:\n%s", traceback.format_exc())
```

ログファイル例：

```
[2025-07-15 20:05:12,334] ERROR - 例外発生:
Traceback (most recent call last):
  File "main.py", line 10, in <module>
    int("abc")
ValueError: invalid literal for int() with base 10: 'abc'
```

---

### `exc_info=True` の活用

`logging` には `exc_info=True` を指定する方法もある：

```python
try:
    risky_operation()
except Exception:
    logging.exception("例外が発生しました")  # exc_info=True と同じ
```

* `logger.exception(...)` は **エラーレベルのログに Traceback を自動追加**
* `except` 節の中でのみ有効（外では何も出ない）

---

### ログレベルと運用上の選択

| ログレベル    | 目的               | 例外との関係      |
| -------- | ---------------- | ----------- |
| DEBUG    | 詳細な内部状態記録        | 例外直前の状況など   |
| INFO     | 通常の処理進捗や完了       | 処理成功の記録など   |
| WARNING  | 無視可能だが注意すべき事象    | 例外未満のエラー兆候  |
| ERROR    | 明確なエラー           | 例外の記録       |
| CRITICAL | システム停止に直結する致命的障害 | 即時通知対象の例外など |

---

### ログ出力設計のポイント

* **stdoutではなくログへ残すべき例外**は、ユーザー対応・デバッグの両面で価値がある
* ログファイルの\*\*ローテーション設計（容量制限・日付管理）\*\*も運用には不可欠
* **テスト失敗や異常終了のときにもログに記録することで、再現性ある問題把握**が可能

---

### まとめ

* `print()` では例外の**構造情報が欠落**する
* `traceback.format_exc()` と `logging.error()` を組み合わせることで、**ファイルにも残せる有用なログ**になる
* `logger.exception()` は**例外時に特化した簡便な記述**
* 運用環境では**ログレベルを正しく使い分けることが信頼性の鍵**

---
## 15章:  try-except-else-finally の書き方

---

### 構文全体の構成

Pythonの例外処理構文は、以下4つの節から構成できます：

```python
try:
    # メイン処理
except ExceptionType:
    # 例外時処理
else:
    # try節が正常終了したときのみ実行
finally:
    # 例外の有無にかかわらず常に実行（後処理）
```

* **全て書く必要はない**が、適切に組み合わせることで**高い制御性と安全性**が得られる

---

### 制御フローの正確な理解

| フェーズ      | 実行される条件                    |
| --------- | -------------------------- |
| `try`     | 常に実行（ここで例外が出るかもしれない）       |
| `except`  | `try` 内で例外が発生した場合のみ        |
| `else`    | `try` が**正常に終了**した場合（例外なし） |
| `finally` | 必ず実行（例外が出ても出なくても）          |

---

### 例：正常系と異常系を完全に分離

```python
def load_data(path: str) -> str:
    try:
        with open(path, "r", encoding="utf-8") as f:
            content = f.read()
    except FileNotFoundError:
        print("ファイルが見つかりません")
    else:
        print("ファイル読み込み成功")
        return content
    finally:
        print("リソースの後処理完了")
```

* 正常な読み込み：`try` → `else` → `finally`
* 例外あり：`try` → `except` → `finally`
* 例外あっても `finally` は**常に実行される**

---

### ネストされた例外処理構造

```python
try:
    do_something()
    try:
        risky_subtask()
    except SubtaskError:
        handle_subtask_error()
except Exception as e:
    handle_general_error(e)
finally:
    clean_up()
```

* 内側と外側で**異なる粒度の例外ハンドリング**が可能
* 例：メイン処理の継続可否を分ける、再試行の実装 など

---

### finally に注意すべきこと

#### ✅：リソース解放・ログ出力・一時ファイル削除など

#### ❌：return や raise を含めない方が安全

```python
def sample():
    try:
        return 1
    finally:
        return 2  # ← return が上書きされる（1ではなく2になる）
```

* `finally` 内の `return` は**例外も正常終了も強制的に打ち消す**

---

### else節の意義と誤用防止


#### 良い使い方：

```python
try:
    result = compute()
except ValueError:
    print("無効な入力です")
else:
    print("結果:", result)
```

* **例外がないときだけの後処理**を `else` に書くことで、**ロジックが明瞭**になる

#### 曖昧な書き方（避ける）：

```python
try:
    result = compute()
    print("成功")  # ← try内に書くと、エラーでも実行される可能性あり
except:
    ...
```

---

### 例外の再送出と復旧判断

```python
try:
    validate()
except ValueError as e:
    log_error(e)
    raise  # 呼び出し元に再度通知
else:
    continue_processing()
finally:
    cleanup()
```

* 例外処理中でも `raise` により**処理責任を上位に委ねる設計**が可能

---

### テンプレート化された構造

実務で安定する例外処理構成：

```python
try:
    risky()
except SpecificError:
    handle_specific()
except Exception as e:
    log_generic(e)
    raise
else:
    confirm_success()
finally:
    cleanup()
```

* 特化処理 → 汎用処理 → 成功時処理 → 必須後処理という**整理された構造**

---

### まとめ

* `try-except-else-finally` は**明示的に責務を分離**できる強力な構文
* `else` は「例外がなかった場合だけの後処理」に活用
* `finally` には副作用（return上書き）を起こす構文を避ける
* **ネスト・再送出・ログ出力との組み合わせ**により、現場で信頼されるエラーハンドリングが構築できる

---
## 16章: exceptの注意点
---

### `except:` の正体と落とし穴

```python
try:
    risky()
except:
    print("何かのエラーが発生")
```

* これは **`except BaseException:` と等価**
* `KeyboardInterrupt`, `SystemExit` など、**「捕捉すべきでない例外」まで握ってしまう**

#### 例：

```python
try:
    while True:
        pass
except:
    print("終了不能")  # Ctrl+C に反応しない場合も
```

---

### 捕捉すべき例外は `Exception` 以下に限定すべき

```python
try:
    risky()
except Exception:
    print("通常の例外を捕捉")
```

| 階層              | 説明                                       |
| --------------- | ---------------------------------------- |
| `BaseException` | すべての例外の最上位、**システム制御も含む**                 |
| `Exception`     | 通常のアプリケーションで発生する例外の親クラス                  |
| サブクラス           | `ValueError`, `TypeError`, `KeyError` など |

---

### 「書きすぎ」のパターンと問題点

#### パターン①：すべての処理を `try` に入れる

```python
try:
    open("file.txt")
    process()
    write()
except Exception:
    print("何かのエラー")
```

* **どこでエラーが起きたか分からない**
* **スコープが広すぎてデバッグ困難**

#### 改善例：

```python
try:
    f = open("file.txt")
except FileNotFoundError:
    handle_file_error()

try:
    process()
except ProcessingError:
    handle_process_error()
```

---

### パターン②：複数の例外を個別に分けすぎる

```python
try:
    func()
except FileNotFoundError:
    ...
except KeyError:
    ...
except IndexError:
    ...
except ValueError:
    ...
# → すべて同じ処理なら except (A, B, C): にまとめる
```

#### 改善案：

```python
try:
    func()
except (KeyError, IndexError, ValueError):
    handle_data_error()
```

---

### パターン③：ログ出力もせず黙って握りつぶす

```python
try:
    do_thing()
except Exception:
    pass  # ← 絶対にNG：デバッグ不能
```

* **問題の存在自体に気づけない**
* 「バグなのに何となく動く」という**最悪の状態**

#### 改善案：

```python
except Exception as e:
    logging.error("例外発生: %s", e)
```


---

### 明示的に書くことの意義

#### ❌ `except:`

* **安全性**：低（BaseExceptionを握る）
* **推奨度**：非推奨
* **理由**：`KeyboardInterrupt` や `SystemExit` まで握ってしまい、強制終了できなくなる危険がある



#### ⭕ `except Exception:`

* **安全性**：高（通常の例外のみ対象）
* **推奨度**：OK
* **理由**：意図しないシステム例外を除き、アプリケーション例外だけを捕捉できる



#### ⭕ `except ValueError:`

* **安全性**：高
* **推奨度**：良い
* **理由**：想定される特定の例外のみを対象とするため、意図的な設計が明確になる



#### ⭕ `except (A, B, C):`

* **安全性**：高
* **推奨度**：良い
* **理由**：処理が共通な複数の例外を効率的にまとめられる



#### ⭕ `except Exception as e:`

* **安全性**：高
* **推奨度**：最良
* **理由**：例外オブジェクトをログ出力や再送出に活用でき、運用性と保守性に優れる


---

### 設計としての考え方

* 原則：「**分かっている例外だけ捕捉する**」
* 分からないものは **ログ出力の上で再送出（raise）** する
* 「握るか・投げるか」は**責務の境界設計**と直結する（→ 24章参照）

---

### まとめ

* `except:` は危険。**必ず例外の種類を明示すること**
* `Exception` 以下を基本とし、`BaseException` は原則握らない
* `try` 節の範囲は**必要最小限に限定する**
* **ログなし except pass は最悪**。最低でもエラー内容は記録すべき

---
## 17章: 複数例外

---

### `except (A, B):` 構文とは？

Pythonでは、複数の例外型を**タプルでまとめて1つの except 節で処理**できます：

```python
try:
    process()
except (ValueError, TypeError):
    print("入力エラーです")
```

* `ValueError` でも `TypeError` でも、同じ処理を適用
* **処理内容が同じなら this is DRY（Don't Repeat Yourself）**

---

### なぜ分けずにまとめるのか？

| 書き方             | 問題点               |
| --------------- | ----------------- |
| 複数の except を並べる | 同じ処理を何度も書くことになる   |
| まとめることで処理の重複を回避 | 処理の意味が明確、保守が容易になる |

例（悪い）：

```python
try:
    func()
except ValueError:
    handle()
except TypeError:
    handle()
```

例（良い）：

```python
try:
    func()
except (ValueError, TypeError):
    handle()
```

---

### 例外オブジェクトを取り出す場合

```python
try:
    func()
except (KeyError, IndexError) as e:
    print(f"アクセスエラー: {type(e).__name__} → {e}")
```

* `as e` はタプルまとめのときも使用可能
* 実際に**どの例外が発生したかを動的に判別**可能

---

### タプルでまとめられる条件と注意点

| 項目           | 内容                                                  |
| ------------ | --------------------------------------------------- |
| 内容           | **例外クラス（型）** だけをタプル内に記述する必要がある                      |
| リストや変数展開     | 不可（`except [A, B]` や `except ERRORS` は SyntaxError） |
| ネストした例外階層の混在 | OK（`(LookupError, ValueError)` など）                  |
| 処理内容が異なる場合   | 分けるべき（無理にまとめると可読性・正確性が下がる）                          |

---

### 実用例：入力検証やデータ構造操作

#### 入力の妥当性検証でよく使う：

```python
try:
    x = int(input("数字を入力: "))
except (ValueError, TypeError):
    print("数値として不正です")
```

#### dictやlist操作時の同時捕捉：

```python
try:
    value = d[key]
except (KeyError, IndexError):
    value = "デフォルト"
```

---

### タプルによる例外処理を使うべき場面

* **同一処理で十分な場合**
* **複数の例外が予想されるが、全て回避不能ではない**
* **例外ごとの区別より、共通の復旧手段を優先する**

---

### 分けるべき場面（逆にまとめない方が良い）

```python
try:
    risky()
except ValueError:
    handle_value()
except TypeError:
    handle_type()
```

* 各例外で**対処方法が異なる**場合は個別処理が正解
* 複数の処理を1つにまとめると、**意図が不明確になる恐れ**

---

### まとめ

* `(A, B)` の構文で**複数例外を1つのexceptで捕捉可能**
* **処理が共通な場合のみまとめるべき**
* `as e` を使えば**例外オブジェクトの詳細**も取得できる
* **構文の簡潔さと意味の明確さ**の両立が設計の鍵

---
## 18章: リトライ処理とエラー回避の設計

---

### なぜリトライ処理が必要か？

現実のプログラムでは、次のような**一時的なエラー**が頻繁に発生します：

* ネットワーク切断・タイムアウト
* 外部APIやDBの一時的な障害
* ファイルの排他ロック状態
* 遅延によるデータ未整備

このようなケースでは、**エラーが出たからといって即終了せず、一定回数再試行（リトライ）すること**が現実的かつ安定性の高い設計となります。

---

### 最小構成のリトライ処理

```python
import time

for i in range(3):  # 最大3回試す
    try:
        result = risky_operation()
        break  # 成功したら抜ける
    except TemporaryError:
        print(f"再試行中... ({i + 1}回目)")
        time.sleep(1)
else:
    raise RuntimeError("3回試しても失敗しました")
```

* `else:` は**すべて失敗したときだけ実行される**
* `time.sleep()` で間隔を空けるのが一般的

---

### 応用：指数バックオフ付きリトライ

```python
import time

for i in range(5):
    try:
        return api_call()
    except APITimeoutError:
        wait = 2 ** i
        print(f"{wait}秒待って再試行")
        time.sleep(wait)
```

* 通信系で使われる「指数バックオフ（exponential backoff）」手法
* 回数を重ねるほど**待機時間を長くして負荷を抑制**

---

### 共通関数化して再利用する

```python
def retry(func, attempts=3, delay=1, exceptions=(Exception,)):
    for i in range(attempts):
        try:
            return func()
        except exceptions as e:
            print(f"{i+1}回目失敗: {e}")
            time.sleep(delay)
    raise RuntimeError("全リトライ失敗")
```

#### 使用例：

```python
retry(lambda: fetch_data("https://example.com"), attempts=5, delay=2)
```

* **クロージャやlambdaで柔軟な再試行設計が可能**
* `exceptions` に複数の型を指定して**特定の例外だけ再試行対象に**

---

### 注意：致命的な例外はリトライすべきでない

| 例外タイプ                                  | リトライ対象とすべきか | 理由                        |
| -------------------------------------- | ----------- | ------------------------- |
| `ConnectionError`, `TimeoutError`      | ⭕           | 一時的な不安定状態の可能性がある          |
| `ValueError`, `TypeError`              | ❌           | プログラム側のロジックミス／繰り返しても成功しない |
| `PermissionError`, `FileNotFoundError` | △           | 状況によりけり（削除／排他ロック／タイミング等）  |

---

### ループ × try のパターン設計

| 構成                       | 用途例                         |
| ------------------------ | --------------------------- |
| `for` ループ + `try-except` | 回数制限付きの試行                   |
| `while True` + `break`   | 条件付き／継続判定付きリトライ             |
| `try` 内に `for` を含む       | 毎回の処理内容が異なる試行（例：複数URLを順に試す） |

---

### 実務的な追加項目

* **成功時のログ出力**：何回目で成功したかを記録
* **例外発生時の詳細出力**：後で原因分析しやすい
* **タイムアウト制御**：リトライ処理自体が永久に続かないように時間制限を設ける

---

### まとめ

* リトライ処理は**一時的なエラーの復旧**に効果的
* `for` + `try-except` + `else` + `sleep` で構成するのが基本形
* **致命的なエラーと一時的なエラーは区別**して扱う
* 共通関数や指数バックオフ設計で**保守性と柔軟性を両立**できる

---
## 19章: バグの原因を特定するための再現コード作成

---

### なぜ「再現コード」が重要か？

エラー報告やバグ調査では、\*\*そのバグが確実に再現する最小限のコード（再現コード）\*\*を用意することが、原因特定と修正を加速させます。

* 複雑な処理のままでは、**何が原因か分からない**
* 再現性のある簡潔なコードであれば、**デバッグ・検証・報告が圧倒的にスムーズ**

---

### 再現コードの原則：**最小・独立・確実**

| 特性  | 内容                                   |
| --- | ------------------------------------ |
| 最小化 | 関係ないコードは一切排除。エラー再現に**必要な処理だけ残す**     |
| 独立性 | 外部モジュール・データ・環境に依存せず、**単体で実行可能**であること |
| 再現性 | **実行すれば必ず同じエラーが出る**ことを確認する           |

---

### 再現コード化のステップ

#### ① 再現箇所の切り出し

* 問題のある処理を**最小単位に切り取る**
* 関連のない関数やクラスは削除

#### ② 変数や引数の固定化

* `sys.argv`, `input()` などの外部依存は値を直書きにする
* APIレスポンスはモックや定数に置き換える

#### ③ 外部依存を除去

* DB、ファイル、API呼び出しを**固定の辞書やリストに置換**
* 例：

```python
# 本番コード（不安定）
data = fetch_from_api("url")

# 再現コード（安定）
data = {"name": "Pikachu", "type": "Electric"}
```

#### ④ 再現コードが本当に再現するか確認

* 複数回実行して**エラーが出続けることを確認**
* 出力ログやTracebackが**常に同じであること**をチェック

---

### 再現コードの構成例（典型）

```python
def transform(data: dict) -> str:
    return data["type"].upper()

data = {"name": "Pikachu"}
print(transform(data))
```

```
KeyError: 'type'
```

→ 一目で原因（キー不足）が分かる。**報告にも使いやすい**

---

### 再現コードの応用：**自動テストの材料に**

* 再現コードはそのまま `pytest` などの**失敗テストケースに転用**可能：

```python
import pytest

def transform(data: dict) -> str:
    return data["type"].upper()

def test_transform_missing_key():
    with pytest.raises(KeyError):
        transform({"name": "Pikachu"})
```

---

### 再現コードの共有時の注意点

| 項目      | 指針                              |
| ------- | ------------------------------- |
| 環境依存の排除 | OS・パス・ローカル設定に依存しないようにする         |
| 必要情報の明記 | Pythonバージョン・依存ライブラリのバージョンを併記する  |
| 転載の配慮   | 本番コードからの引用がある場合は機密情報を除去する       |
| 再現手順    | 「このコードをこの環境で実行するとこうなる」という説明をつける |

---

### 再現コードを持つことの価値

* **自分で理解が深まる**
* **他人に助けを求めやすくなる**
* **Issueやバグレポートに説得力が出る**
* **修正後に同様のバグを再発させないためのテスト資産にもなる**

---

### まとめ

* バグの特定には「再現性のあるミニマルコード」が最も効果的
* 不要な処理・外部依存を削ぎ落とし、**本質だけを残す**
* 再現コードは**個人のデバッグ・チーム連携・自動テスト**に幅広く活用可能

---
## 20章: エラー発生時の状態記録

---

### なぜ「状態の記録」が必要か？

バグ調査や障害対応において、**その時点でプログラム内部がどのような状態だったか**を把握することが極めて重要です。
特に、**一時的・再現困難な不具合**では、状態記録がなければ調査が行き詰まります。

---

### 状態記録とは？

エラー発生時に、以下のような**内部情報をログやファイルに残す**こと：

* 発生した例外（種類・内容・Traceback）
* 実行中の関数・引数
* 各種変数の値やデータ構造の中身
* 処理対象のファイル名・URL・タイムスタンプなど

---

### 最低限の記録：Tracebackの保存

```python
import traceback

try:
    process()
except Exception:
    with open("error.log", "a", encoding="utf-8") as f:
        f.write(traceback.format_exc())
```

* `traceback.format_exc()` は**Traceback全文を文字列で取得**
* ファイルへの保存により**再現性のないエラーも後から確認可能**

---

### 任意変数の記録（ログ or JSON）

#### ログ出力

```python
import logging

logger = logging.getLogger("debugger")
logger.setLevel(logging.INFO)

try:
    do_work(x=42, y="abc")
except Exception as e:
    logger.error("x=%s, y=%s", 42, "abc")
```

#### JSON保存

```python
import json

state = {
    "stage": "fetching",
    "url": "https://example.com",
    "params": {"q": "pikachu"},
}
with open("snapshot.json", "w", encoding="utf-8") as f:
    json.dump(state, f, indent=2, ensure_ascii=False)
```

---

### 状態ダンプの技法：pickle / pprint / repr

#### `pprint` で人間向けに整形表示

```python
from pprint import pprint

pprint(config)
```

#### `repr()` で安全にログ出力

```python
logger.debug(f"user = {repr(user_obj)}")
```

#### `pickle` による変数ダンプ（後で復元）

```python
import pickle

with open("state.pkl", "wb") as f:
    pickle.dump(data, f)
```

---

### 状態記録の対象とタイミング

| タイミング          | 記録内容例                       |
| -------------- | --------------------------- |
| 処理開始直後         | 引数、入力ファイル、コマンドライン引数         |
| エラー発生時（except） | 例外の種類・内容・Traceback、ローカル変数の値 |
| 最終処理直前         | 出力ファイル名、保存先、成功フラグ、統計値など     |

---

### 自動化された例：デコレータで状態記録

```python
def log_args(func):
    def wrapper(*args, **kwargs):
        print(f"[DEBUG] {func.__name__} args={args}, kwargs={kwargs}")
        return func(*args, **kwargs)
    return wrapper

@log_args
def calc(a, b): return a + b
```

---

### 記録データの保存先と形式

| 種類       | 保存先               | 形式          | 特徴                          |
| -------- | ----------------- | ----------- | --------------------------- |
| ログ       | `*.log`           | テキスト        | 人間が読む用、長期保存向き               |
| スナップショット | `*.json`, `*.pkl` | JSON/pickle | 状態構造をそのまま保存／復元可能（JSONは可搬性高） |
| デバッグ用    | stdout／stderr     | 任意          | 一時的な観察用途（CIログなど）            |

---

### まとめ

* エラー調査においては\*\*「そのときの状態を残す」ことが最優先\*\*
* Tracebackだけでなく、**変数・引数・構造体の中身も記録すべき**
* `json`, `pickle`, `logging`, `traceback` を**目的に応じて使い分ける**
* 状態記録は**バグ調査・再現確認・テスト強化**にも大きく貢献する

---
## 21章: テスト失敗時のエラー対処（pytest前提）

---

### 「なぜ失敗したかを即時特定できるか」

単体テストや統合テストが失敗したとき、**テストフレームワークが提供するエラー出力を正確に読み取り、速やかに原因を特定する力**が必要です。特に `pytest` は詳細な失敗情報を出力するため、**その構造を読み解く技術**が重要になります。

---

### 代表的な失敗パターン

| パターン             | 例                                 | 対応                              |
| ---------------- | --------------------------------- | ------------------------------- |
| `assert` 失敗      | `assert x == y` が False           | 実際の値と期待値を比較し、原因を精査              |
| 未定義変数・型エラー       | `NameError`, `TypeError`          | テスト本体にミスがある可能性                  |
| テスト用 fixture の失敗 | `fixture 'sample_user' not found` | fixture 定義や import の確認          |
| 例外が発生すべきなのに発生しない | `Failed: DID NOT RAISE`           | `with pytest.raises(...)` の対象確認 |

---

### `assert` 失敗時の出力例と分析

```python
def test_add():
    assert 1 + 2 == 4
```

出力：

```
>       assert 1 + 2 == 4
E       assert 3 == 4
```

* `E` 行に注目：**実際の値（3）と期待値（4）の差異が明示されている**
* テスト対象ではなく**テストコード側のロジックミス**の可能性が高い

---

### 例外発生を期待するテストの書き方

```python
import pytest

def divide(a, b):
    return a / b

def test_zero_division():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)
```

#### よくある失敗：

```python
with pytest.raises(ValueError):
    divide(1, 0)
```

```
Failed: DID NOT RAISE <class 'ValueError'>
```

* **例外の型が一致していない**（ZeroDivisionError を期待すべき）

---

### fixture の失敗例

```python
# test_example.py
def test_with_fixture(user):
    ...
```

出力：

```
fixture 'user' not found
```

* fixture の**名前ミス**または **スコープ外**であることが多い
* 解決法：`conftest.py` の定義確認、`@pytest.fixture` デコレータを確認

---

### `--maxfail` や `-x` を使った調査効率化

```bash
pytest -x      # 最初の1件で止める
pytest --maxfail=3  # 最大3件で停止
```

* **大量のテストが失敗するケース**では、**先に代表的な1件を確実に直す**のが鉄則

---

### `pytest.raises` の詳細オプション

```python
with pytest.raises(ValueError) as excinfo:
    raise ValueError("無効です")
assert "無効" in str(excinfo.value)
```

* `as excinfo` を使えば**例外オブジェクトの内容検証**も可能

---

### ログ出力と組み合わせたデバッグ

```python
import logging

logging.basicConfig(level=logging.DEBUG)

def test_log_debug():
    logging.debug("変数xの値: %s", 42)
    assert 42 == 43
```

* **失敗直前の状態をログに記録しておくことで、再現が容易に**

---

### まとめ

* `pytest` のエラー出力は**詳細かつ読みやすい**が、**見るべき行を素早く特定**することが重要
* `assert` 失敗 → 実値と期待値の差を見る
* `raises()` → 例外の型と文言を明確に確認
* `fixture` エラー → 名前・import・スコープを確認
* 必要に応じて `-x` や `--maxfail` で**調査の効率化**

---
## 22章: 外部API・DBなどでのエラー処理

---

### なぜ外部サービスでは例外処理が必須か？

ファイルや内部処理と異なり、**外部API・DB接続・ネットワーク通信は予測不能な要因**で失敗します：

* サーバーダウン・通信断・タイムアウト
* 認証エラー・アクセス制限
* 応答形式の不一致や予期せぬ空データ
* DBロック・接続切れ・クエリ失敗

これらは**自分のコードに責任がない場合も多いため、適切なリトライ・フォールバック・ログ記録が重要**です。

---

### HTTP API の典型的な例外構造（requests使用時）

```python
import requests

try:
    response = requests.get("https://api.example.com/data", timeout=5)
    response.raise_for_status()
    data = response.json()
except requests.exceptions.HTTPError as e:
    print(f"HTTPエラー: {e}")
except requests.exceptions.Timeout:
    print("タイムアウトしました")
except requests.exceptions.RequestException as e:
    print(f"通信エラー: {e}")
```

| 例外クラス                    | 内容                 |
| ------------------------ | ------------------ |
| `HTTPError`              | ステータスコード 4xx/5xx   |
| `Timeout`                | 接続・応答が一定時間で得られない   |
| `ConnectionError`        | 接続失敗（DNS, ネット遮断など） |
| `RequestException`（親クラス） | 通信関連のすべての例外を包括     |

---

### DB系（sqlite3, psycopg2など）の例外例

```python
import sqlite3

try:
    conn = sqlite3.connect("poke.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
except sqlite3.OperationalError as e:
    print(f"DB操作エラー: {e}")
except sqlite3.DatabaseError as e:
    print(f"DB接続エラー: {e}")
finally:
    conn.close()
```

| 例外クラス              | 内容               |
| ------------------ | ---------------- |
| `OperationalError` | SQL構文・接続失敗・ロックなど |
| `IntegrityError`   | 主キー重複・外部キー違反     |
| `ProgrammingError` | SQL文法エラー・無効な操作   |
| `DatabaseError`（親） | DB全般のエラーの親クラス    |

---

### よくある対応パターン

#### ✅ 失敗しても継続

```python
try:
    fetch()
except requests.RequestException:
    fallback_data = load_cache()
```

#### ✅ 状態記録と再送出

```python
try:
    update_db()
except sqlite3.IntegrityError as e:
    log_error(e)
    raise
```

#### ✅ リトライ処理

```python
for i in range(3):
    try:
        return api_call()
    except TimeoutError:
        time.sleep(2)
```

---

### 設計指針：API層は責任分界が明確

| 層         | 責務                    | 推奨される例外設計          |
| --------- | --------------------- | ------------------ |
| ライブラリ層    | エラーをラップして再送出          | カスタム例外で意味を明示する     |
| アプリケーション層 | 例外を捕捉してログや再試行・代替処理を行う | `try-except` で制御する |
| 呼び出し元UI層  | ユーザーに伝える（表示・ログ）       | ログ記録・メッセージ整形       |

---

### ログ・通知・検出の実用例

```python
except requests.HTTPError as e:
    logger.error("API呼び出し失敗: %s", e)
    notify_slack("サーバーからの応答が不正です")
```

---

### テストと例外の関係

* `requests-mock`, `pytest.raises`, `unittest.mock` を使って**外部要因を模擬**
* **「外部依存部は例外を出すことがある」前提でテストを組む**

---

### まとめ

* 外部APIやDBとの通信では**失敗前提の設計が不可欠**
* 例外の粒度を適切に分け、**ログ・リトライ・再送出を組み合わせる**
* `requests` / `sqlite3` などの**例外体系を把握し、構造的に設計**
* \*\*「責任のある場所でのみ握り、上に投げる判断」\*\*を常に意識すること

---
## 23章: 非同期処理における例外

---

### なぜ非同期処理は例外が難しいのか？

Python の `asyncio` を用いた非同期処理では、**複数の非同期タスクが並行して動作**するため、通常の `try-except` 構文だけではすべての例外を拾いきれないことがあります。

* **タスク単位で例外が独立して発生する**
* **非同期関数（`async def`）内とその外で例外の扱いが異なる**
* **`gather()` や `create_task()` を使うと例外が“隠れる”ことがある**

---

### 非同期関数の例外処理（基本形）

```python
import asyncio

async def fetch():
    raise ValueError("非同期タスクのエラー")

async def main():
    try:
        await fetch()
    except ValueError as e:
        print(f"捕捉: {e}")

asyncio.run(main())
```

* `await` によって呼び出された関数内の例外は**通常通り try-except で捕捉可能**

---

### `gather()` による例外の扱い方

```python
import asyncio

async def task1():
    raise ValueError("task1失敗")

async def task2():
    await asyncio.sleep(0.1)
    return "task2成功"

async def main():
    results = await asyncio.gather(task1(), task2(), return_exceptions=True)
    print(results)

asyncio.run(main())
```

#### `return_exceptions=True` の意味：

* 例外が発生しても **raise せず、戻り値としてリストに含める**
* 各タスクの成否を**1つのリストで扱える**
* 使わない場合、**最初にエラーが出た時点で `gather()` 全体が例外となる**

---

### `asyncio.create_task()` で例外が“無視される”問題

```python
async def background():
    raise RuntimeError("バックグラウンド失敗")

async def main():
    asyncio.create_task(background())
    await asyncio.sleep(0.5)

asyncio.run(main())
```

* この例では、**RuntimeError は画面に出るだけで捕捉されない**
* **create\_task() で作成されたタスクの例外は放置すると検出されないまま失敗する**

#### 対策：タスクを明示的に監視・回収する

```python
task = asyncio.create_task(background())

try:
    await task
except RuntimeError as e:
    print(f"捕捉成功: {e}")
```

---

### 非同期タスクの一括監視とログ出力

```python
tasks = [asyncio.create_task(t()) for t in funcs]
results = await asyncio.gather(*tasks, return_exceptions=True)

for r in results:
    if isinstance(r, Exception):
        logger.error("例外: %s", r)
```

* `gather()` と `return_exceptions=True` の組み合わせで、
  **例外と正常終了を同時に収集・分類**できる

---

### asyncioで使える例外系 API

| 関数・構文                                         | 説明                   |
| --------------------------------------------- | -------------------- |
| `asyncio.gather(..., return_exceptions=True)` | 例外を raise せず収集       |
| `asyncio.wait()`                              | 成否にかかわらず全タスク完了を待機    |
| `task.exception()`                            | 完了済みのタスクに発生した例外を取得可能 |
| `task.cancel()` + `CancelledError`            | タスクの明示的中断とキャンセル例外の捕捉 |

---

### 非同期処理の例外は「設計で守る」

* **バックグラウンドタスクは握る責任を持つ**
* `gather()` は**return\_exceptions の有無で設計方針が変わる**
* `await` のつけ忘れや `create_task()` の放置は**最悪のバグ温床**

---

### まとめ

* `async` では**通常の try-except が効かない場面が存在**
* `gather()` は `return_exceptions=True` を活用すべき
* `create_task()` の例外は明示的に `await` して監視
* 非同期処理では「**例外は発生する前提で構造的に制御する設計**」が必要

---
## 24章: エラー処理方針の決め方（例外を握る・投げる）

---

### なぜ「握る」か「投げる」かを決める必要があるのか？

例外処理においては、**その場で処理（握る）すべきか、呼び出し元へ渡す（投げる）べきかを明確に判断**しなければなりません。
これを曖昧にすると、以下のような問題が起こります：

* **ライブラリ側で例外を握り潰してしまい、利用者がバグに気付けない**
* **UI側に責任を丸投げしすぎて、設計責任が不明確になる**
* **ログ出力・状態記録・再試行などの対処漏れ**

---

### 判断基準：誰が例外の意味を理解できるか？

| ケース                 | 対応方針        | 理由                             |
| ------------------- | ----------- | ------------------------------ |
| 呼び出し元が状況を知っている場合    | **投げる**     | 利用者側でリトライ・ログ出力などの判断ができるため      |
| 呼び出し元が無知（共通関数・汎用処理） | **握る**      | 呼び出し元が意味を判断できない例外は、呼び出し側では処理不能 |
| 例外が内部処理に由来する場合      | **握る + ログ** | 関数内部で完結すべき問題は、そこで処理し再送出しない     |
| 例外が外部要因による場合        | **投げる**     | 呼び出し元が復旧方法を決める（再接続、代替処理など）     |

---

### 「握る」パターン（try-exceptで処理）

```python
def parse_int(text: str) -> int:
    try:
        return int(text)
    except ValueError:
        return 0  # デフォルト値で握る
```

* 関数内部で問題が完結し、呼び出し元は例外を意識しなくてよい
* このような処理は「**握る設計**」

---

### 「投げる」パターン（raiseで渡す）

```python
def fetch_data(url: str) -> dict:
    response = requests.get(url)
    response.raise_for_status()  # 呼び出し元に投げる
    return response.json()
```

* 通信失敗はライブラリ内部で判断できない
* 利用者側でリトライ・失敗通知・UI更新を設計する必要がある

---

### 握って再送出するパターン（ログ記録などを追加）

```python
def risky():
    try:
        process()
    except SpecificError as e:
        logger.error("処理中にエラー発生: %s", e)
        raise  # 再送出
```

* **その場でログなどを記録しつつ、制御責任は上位へ返す**
* **エラーの文脈を補足しながら、処理責任を委譲**する設計

---

### 握る／投げるの境界を設計で明確にする

#### 1. ライブラリ・ユーティリティ層は「投げる」

* 利用者側に判断材料を残すため、**エラーを握らない**
* 代わりに **明示的なドキュメントやカスタム例外**を用意する

#### 2. アプリケーション層は「握る or ログ付き再送出」

* UIへの通知や状態変化を伴う処理は**その場で例外を処理する責任がある**
* **ユーザーに見せるメッセージや挙動を制御する責任があるため**

---

### エラー方針の実務的設計例

| レイヤ             | 方針                  | 例外処理の役割                |
| --------------- | ------------------- | ---------------------- |
| ユーティリティ関数・小さな関数 | **投げる（握らない）**       | 処理失敗の判断を上に任せる          |
| ライブラリモジュール      | **投げる＋カスタム例外**      | 利用者が意味を判別しやすく          |
| アプリケーションサービス層   | **握る or ログ＋再送出**    | 状態の記録と通知、ログ整備          |
| UI層（CLI/GUI）    | **握る＋メッセージ提示・終了処理** | ユーザーへの対応・終了フローの制御責任がある |

---

### 「握るべきでないのに握ってしまった」設計の害

* **バグが“見えなくなる”**（例外が握り潰されてログにも出ない）
* **利用者が失敗を検知できない**（返り値がNoneなどだと誤動作する）
* **再現性が失われる**（問題がなかったように見えて再発）

---

### まとめ

* 「握る or 投げる」は**エラー処理設計の根幹**
* 基本方針：**知っている者が処理し、知らない者には投げる**
* 「握り潰し」は**絶対に避ける**（ログなし `except: pass` など）
* エラー処理は**責務分界の設計そのもの**。コードの信頼性を左右する重要項目

---
