---
title: Python_Foundation_
keyword:
created: 2025-08-20 00:57
modified: 2025-08-20 00:57
vault: technology
catergory: Language
language: Python
area: Foundation
identify:  
type: 
role: 
order: 
---

# 📍第2部：実装例

---

## 07章：引数のある関数への対応

---

### 解説

本章では、**引数を持つ関数に対するデコレーターの適用方法**について解説する。デコレーターの基本構造では、引数を持たない関数に対しては機能するが、**現実の関数のほとんどは引数を持つ**。このため、任意の引数に対応できる汎用的なデコレーターを設計することは、実務での利用において欠かせない技術である。

Pythonでは、関数の呼び出しに使われる引数を「位置引数（`*args`）」と「キーワード引数（`**kwargs`）」という2種類の構文によって抽象化できる。これにより、デコレーター側では関数ごとの引数構造を個別に記述することなく、**任意のシグネチャの関数を1つのデコレーターで処理できる**ようになる。

この章では、以下の3点を軸に解説する：

* `*args, **kwargs` による引数の一般化
* 呼び出し元と被装飾関数の整合性を保つ方法
* 引数を操作・検査するデザインパターン

---

### `*args, **kwargs` の役割と重要性

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # 前処理
        result = func(*args, **kwargs)
        # 後処理
        return result
    return wrapper
```

このように `wrapper()` 側が `*args, **kwargs` を受け取ることで、任意の数・種類の引数を関数に渡すことが可能となる。

| 構文         | 意味                           |
| ---------- | ---------------------------- |
| `*args`    | 任意の個数の位置引数をタプルとして受け取る        |
| `**kwargs` | 任意の個数のキーワード引数を辞書として受け取る      |
| 両方併用       | 位置／キーワードの混在や可変引数にも完全に対応可能になる |

この仕組みにより、あらゆる関数に対して1つのデコレーターで対応できるため、再利用性が極めて高まる。

---

### 例：デバッグ用デコレーター

```python
def debug(func):
    def wrapper(*args, **kwargs):
        print(f"[DEBUG] 呼び出し：{func.__name__}")
        print(f"[DEBUG] 引数: {args}, キーワード引数: {kwargs}")
        result = func(*args, **kwargs)
        print(f"[DEBUG] 戻り値: {result}")
        return result
    return wrapper

@debug
def multiply(x, y=1):
    return x * y

multiply(3, y=5)
```

出力例：

```
[DEBUG] 呼び出し：multiply
[DEBUG] 引数: (3,), キーワード引数: {'y': 5}
[DEBUG] 戻り値: 15
```

このように、元の関数に一切手を加えずに**周辺処理（ロギング）を挿入する**ことができる。

---

### 呼び出し元と整合性を保つ設計

デコレーターを設計する際に注意すべき点は、「引数の内容を変えずに委譲する」ことである。もし引数の型や数を意図的に変更する場合は、それが呼び出し側の期待と一致している必要がある。

誤ったパターン：

```python
def bad_decorator(func):
    def wrapper():
        return func()  # 引数を渡していない
    return wrapper

@bad_decorator
def greet(name):
    print(f"{name}さん、こんにちは")

greet("佐藤")  # TypeError: greet() missing 1 required positional argument
```

---

### 応用：引数の前処理・検証

ラップ関数で引数を受け取ることで、**関数呼び出し前に値を検査・変換・ログ出力**することも可能になる。

```python
def ensure_positive(func):
    def wrapper(*args, **kwargs):
        new_args = [max(0, a) for a in args]
        return func(*new_args, **kwargs)
    return wrapper

@ensure_positive
def subtract(x, y):
    return x - y

print(subtract(-10, 5))  # → 0 - 5 = -5
```

このように、**引数の事前加工を柔軟に実装できる**のも `*args, **kwargs` を使う利点の一つである。

---

### まとめ

* 実務で使う関数はほとんどが引数付きであり、デコレーターもそれに対応する必要がある
* `*args, **kwargs` を使えば、どんな関数にも装飾が可能になる
* ラップ関数は「受け取る」「委譲する」「操作する」という3段階の責任を持つ
* 引数の整合性を崩さずにラップする構造は、再利用性・信頼性を両立する重要な設計パターンである

このパターンをマスターすることで、実用的なデコレーターのほぼすべてを設計・実装できるようになる。以降の章で扱う戻り値処理や条件制御、ログの挿入なども、すべてこの基本構造の応用である。

---


## 08章：戻り値を操作するデコレーター

---

### 解説

デコレーターの目的は関数の\*\*「周囲に処理を挿入すること」**にあるが、その対象は引数だけでなく、**戻り値にも及ぶ**。本章では、関数の戻り値に対して検査・加工・記録などの操作を行うデコレーターを設計し、**呼び出し結果の制御**や**副作用との分離\*\*を実現する方法を解説する。

戻り値を操作することで、たとえば以下のような処理が可能になる：

* 値の検証（例：`None` でないか確認）
* 値の変換（例：HTMLエスケープやJSON化）
* ロギングやメトリクスの送信
* 戻り値のキャッシュ・フィルタリング

---

### 基本構造：戻り値を受け取って操作する

```python
def modify_return(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return f"[修正済] {result}"
    return wrapper

@modify_return
def greet(name):
    return f"{name}さん、こんにちは"

print(greet("佐藤"))  # → [修正済] 佐藤さん、こんにちは
```

ここでは、元の戻り値（文字列）にプレフィックスを追加して返している。関数本体のロジックを改変せずに、**最終出力だけを差し替える構造**となっている。

---

### 応用：Noneの扱い

関数の戻り値が `None` の場合、処理が意図通りに進行していないことがある。そういったケースに対し、**戻り値の補完や警告ログ**を挿入することができる。

```python
def ensure_not_none(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        if result is None:
            print("警告：戻り値が None です")
            return "デフォルト値"
        return result
    return wrapper

@ensure_not_none
def maybe_none():
    return None

print(maybe_none())  # → 警告：戻り値が None です / デフォルト値
```

---

### データ検査やフィルタリングにも応用可能

戻り値がリストや辞書などのコレクションである場合、その中身を検査・整形することで、**一貫した出力形式**を保つことができる。

```python
def filter_negatives(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return [x for x in result if x >= 0]
    return wrapper

@filter_negatives
def get_scores():
    return [10, -5, 0, 25, -8]

print(get_scores())  # → [10, 0, 25]
```

---

### 副作用との分離設計

戻り値を使って、関数の出力に基づく副作用を発生させることもできるが、その場合は**副作用ロジックと出力ロジックを明確に分離する**必要がある。

```python
def log_result(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"実行結果：{result}")
        return result
    return wrapper
```

このように、副作用（ログ出力）を挿入しつつ、関数の出力を変更しない設計にすることが、**可読性と安全性を両立する鍵**となる。

---

### テスト容易性と一貫性の担保

戻り値を加工するデコレーターは、関数の\*\*純粋性（pure function）\*\*を損なう可能性があるため、**以下の設計ガイドライン**が重要になる：

| 設計ポイント         | 説明                              |
| -------------- | ------------------------------- |
| 関数の出力に副作用を含めない | ログ出力・状態変化はあくまでラップ側で完結させる        |
| 加工対象が明示的であること  | 何をどう加工しているかがコードから明瞭に読み取れること     |
| 変更可能性の明記       | 戻り値が変わることがあるとドキュメント・型ヒントで明示すること |

---

### まとめ

* デコレーターは戻り値にも操作を加えることができる
* ログ、変換、検証、補完などさまざまな実用的応用が可能
* 戻り値を変更するか、変更せずに副作用だけ加えるかは設計の分岐点となる
* 関数本体に手を加えずに出力の振る舞いを変えられることは、保守性とテスト性を大きく高める

このパターンは、**返り値処理の標準化や出力の安全化**といった目的で多用され、あらゆる業務系Pythonアプリケーションで応用されている。


<hr>

## 08章：戻り値を操作するデコレーター

---

### 解説

デコレーターの目的は関数の\*\*「周囲に処理を挿入すること」**にあるが、その対象は引数だけでなく、**戻り値にも及ぶ**。本章では、関数の戻り値に対して検査・加工・記録などの操作を行うデコレーターを設計し、**呼び出し結果の制御**や**副作用との分離\*\*を実現する方法を解説する。

戻り値を操作することで、たとえば以下のような処理が可能になる：

* 値の検証（例：`None` でないか確認）
* 値の変換（例：HTMLエスケープやJSON化）
* ロギングやメトリクスの送信
* 戻り値のキャッシュ・フィルタリング

---

### 基本構造：戻り値を受け取って操作する

```python
def modify_return(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return f"[修正済] {result}"
    return wrapper

@modify_return
def greet(name):
    return f"{name}さん、こんにちは"

print(greet("佐藤"))  # → [修正済] 佐藤さん、こんにちは
```

ここでは、元の戻り値（文字列）にプレフィックスを追加して返している。関数本体のロジックを改変せずに、**最終出力だけを差し替える構造**となっている。

---

### 応用：Noneの扱い

関数の戻り値が `None` の場合、処理が意図通りに進行していないことがある。そういったケースに対し、**戻り値の補完や警告ログ**を挿入することができる。

```python
def ensure_not_none(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        if result is None:
            print("警告：戻り値が None です")
            return "デフォルト値"
        return result
    return wrapper

@ensure_not_none
def maybe_none():
    return None

print(maybe_none())  # → 警告：戻り値が None です / デフォルト値
```

---

### データ検査やフィルタリングにも応用可能

戻り値がリストや辞書などのコレクションである場合、その中身を検査・整形することで、**一貫した出力形式**を保つことができる。

```python
def filter_negatives(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return [x for x in result if x >= 0]
    return wrapper

@filter_negatives
def get_scores():
    return [10, -5, 0, 25, -8]

print(get_scores())  # → [10, 0, 25]
```

---

### 副作用との分離設計

戻り値を使って、関数の出力に基づく副作用を発生させることもできるが、その場合は**副作用ロジックと出力ロジックを明確に分離する**必要がある。

```python
def log_result(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"実行結果：{result}")
        return result
    return wrapper
```

このように、副作用（ログ出力）を挿入しつつ、関数の出力を変更しない設計にすることが、**可読性と安全性を両立する鍵**となる。

---

### テスト容易性と一貫性の担保

戻り値を加工するデコレーターは、関数の\*\*純粋性（pure function）\*\*を損なう可能性があるため、**以下の設計ガイドライン**が重要になる：

| 設計ポイント         | 説明                              |
| -------------- | ------------------------------- |
| 関数の出力に副作用を含めない | ログ出力・状態変化はあくまでラップ側で完結させる        |
| 加工対象が明示的であること  | 何をどう加工しているかがコードから明瞭に読み取れること     |
| 変更可能性の明記       | 戻り値が変わることがあるとドキュメント・型ヒントで明示すること |

---

### まとめ

* デコレーターは戻り値にも操作を加えることができる
* ログ、変換、検証、補完などさまざまな実用的応用が可能
* 戻り値を変更するか、変更せずに副作用だけ加えるかは設計の分岐点となる
* 関数本体に手を加えずに出力の振る舞いを変えられることは、保守性とテスト性を大きく高める

このパターンは、**返り値処理の標準化や出力の安全化**といった目的で多用され、あらゆる業務系Pythonアプリケーションで応用されている。

<hr>

## 09章：実行時間を測定するデコレーター

---

### 解説

パフォーマンスの最適化や処理負荷の診断には、**関数の実行時間を正確に測定する仕組み**が不可欠である。そこで本章では、関数の開始から終了までの時間を計測し、その結果をログや標準出力に出す**タイミング系デコレーター**の設計を解説する。

このようなデコレーターは、コードの可視化・検証・ベンチマーク・ボトルネック分析など、実務のあらゆる場面で応用できる。また、処理時間をロギングするだけでなく、**フォーマットや表示方法を統一できる点も利点**である。

---

### 基本構造：`time` モジュールを用いた計測

```python
import time

def measure_time(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        duration = end - start
        print(f"{func.__name__} took {duration:.4f} seconds.")
        return result
    return wrapper
```

このデコレーターは、`time.time()` を使って処理前後のタイムスタンプを取得し、その差分を秒単位で出力する。戻り値は変えずにそのまま返しているため、**関数の本質的な動作は変えずに、周囲に観測処理を追加している**構造である。

---

### 精度と単位の選択：`time` vs `perf_counter`

計測対象によっては、より高精度なタイマーが必要になる。たとえば `time.perf_counter()` は、OS依存の**最小精度の高いカウンタ**であり、短時間処理の計測に向いている。

```python
def precise_timer(func):
    def wrapper(*args, **kwargs):
        from time import perf_counter
        start = perf_counter()
        result = func(*args, **kwargs)
        end = perf_counter()
        print(f"{func.__name__}: {end - start:.6f}s")
        return result
    return wrapper
```

---

### 応用：時間を返り値として扱う

計測結果をログとして出すのではなく、**戻り値の一部として返す**ことで、上位処理が値を受け取って利用できる構造にすることもできる。

```python
def timed_with_result(func):
    def wrapper(*args, **kwargs):
        from time import perf_counter
        start = perf_counter()
        result = func(*args, **kwargs)
        duration = perf_counter() - start
        return result, duration
    return wrapper

@timed_with_result
def compute():
    sum(range(10**6))

res, dur = compute()
print(f"処理時間: {dur:.4f}秒")
```

このように、計測と本来の処理を**合成した戻り値として返す**ことで、より柔軟な連携が可能になる。

---

### 秒数のフォーマットと表示方法

秒数は小数点以下の桁数を調整したり、ミリ秒単位に変換したりすることで、**ユーザーにとって読みやすい形に加工**するのが実務上は望ましい。

```python
print(f"{duration * 1000:.2f}ms")  # ミリ秒
print(f"{duration:.3f}s")         # 秒（小数第3位）
```

場合によってはログファイルに記録することも想定し、**フォーマットと出力先の統一**が設計上の要件となる。

---

### 単体テストとの相性と注意点

実行時間を測定するデコレーターは、以下の点において注意が必要である：

| 観点           | 内容                                |
| ------------ | --------------------------------- |
| **テストの非決定性** | 処理時間は環境や負荷に依存するため、アサーションに使うべきではない |
| **副作用の抑制**   | テスト環境では標準出力への出力を抑える設計が望ましい        |
| **デバッグモード**  | 環境変数や引数でログ出力の有無を切り替える設計が有効        |

---

### ロギングモジュールとの統合（発展）

標準出力ではなく `logging` モジュールを用いることで、実行時間ログを**適切なログレベル・出力先に管理する**ことが可能になる。

```python
import logging

def time_logger(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        duration = time.time() - start
        logging.info(f"{func.__name__} took {duration:.3f}s")
        return result
    return wrapper
```

---

### まとめ

* 実行時間の測定は `time.time()` または `perf_counter()` で行う
* デコレーターによって関数の処理時間を外部から観測・記録できる
* ログ出力、戻り値への追加、ミリ秒変換など実務用途に応じた応用が可能
* 非決定的な処理であるため、テストや運用設計では環境依存性を考慮する
* ロギングモジュールと統合することで、プロジェクト全体の可観測性が向上する

このパターンは、パフォーマンス測定だけでなく、**ボトルネック発見や予測可能性の確保**にも有効な実務的デコレーターの代表格である。

<hr>

## 10章：ロギング・プリントを行うデコレーター

---

### 解説

本章では、関数の実行に合わせて**標準出力やログに情報を出力するデコレーター**を設計する。これは、関数の呼び出し・引数・戻り値などの情報を自動的に記録することで、**動作確認・デバッグ・監視**に役立てる目的である。

Pythonの `print()` を使えば手軽に標準出力できる一方で、`logging` モジュールを使えば**ログレベルや出力先を柔軟に制御**できる。本章では、両者の特徴を整理し、実務向けの設計に落とし込む方法を解説する。

---

### プリントベースのデコレーター（軽量）

```python
def print_debug(func):
    def wrapper(*args, **kwargs):
        print(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

適用例：

```python
@print_debug
def greet(name):
    return f"{name}さん、こんにちは"

greet("佐藤")
```

出力：

```
[CALL] greet args=('佐藤',) kwargs={}
[RETURN] greet -> 佐藤さん、こんにちは
```

このように、関数呼び出し前後で**情報を出力するだけの簡易な構成**であり、スクリプトや個人の学習用途に向いている。

---

### ロギングを使う構成（実務向け）

標準出力ではなく `logging` モジュールを使うことで、ログレベル（INFO/WARNING/ERRORなど）や出力先（ファイル、標準出力、syslogなど）を柔軟に設定できる。

```python
import logging

def log_debug(func):
    def wrapper(*args, **kwargs):
        logging.info(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

ログ設定（例）：

```python
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
```

---

### ロギングとプリントの違い

| 観点       | `print()` | `logging`                    |
| -------- | --------- | ---------------------------- |
| 出力先      | 標準出力のみ    | ファイル・標準出力・ストリーム等を選択可         |
| レベル制御    | 不可        | INFO, WARNING, ERROR などで制御可能 |
| 運用環境との連携 | 弱い        | 強い（構成ファイルや外部ツールと連携しやすい）      |
| フィルタリング  | 不可        | ログレベルやフィルタで対象を制限可能           |

**開発中は `print()`、運用や検証では `logging`** という使い分けが一般的である。

---

### 引数・戻り値の詳細な出力

関数の中で何が渡され、何が返ってきたかを正確に追跡するには、次のように書式を整えると可読性が高まる：

```python
def log_detail(func):
    def wrapper(*args, **kwargs):
        print(f"→ 実行関数: {func.__name__}")
        print(f"→ 引数: {args}")
        print(f"→ キーワード引数: {kwargs}")
        result = func(*args, **kwargs)
        print(f"→ 戻り値: {result}")
        return result
    return wrapper
```

これは関数の入出力がブラックボックスにならないよう、**自動的に観測可能にするための実用的手法**である。

---

### 本番環境との切り替え

以下のような設計により、**デバッグ用の出力を環境によって切り替える**ことも可能：

```python
import os

def conditional_log(func):
    def wrapper(*args, **kwargs):
        if os.getenv("DEBUG_MODE") == "1":
            print(f"[DEBUG] {func.__name__} called")
        return func(*args, **kwargs)
    return wrapper
```

これは、**ロジックそのものは変えずに、出力のみ制御したい場合に便利**である。

---

### 注意点と設計上の配慮

| 課題         | 解説                              |
| ---------- | ------------------------------- |
| 機密情報の誤出力   | ログにパスワードやAPIキーが出力されないようにする必要がある |
| デバッグと本番の混同 | 開発時の出力がそのまま本番に残らないよう、設定で制御可能にする |
| 可読性の確保     | フォーマットの統一、情報量の制御が重要（長すぎるログは逆効果） |

---

### まとめ

* ログやプリントは、関数の挙動を観測・検証するために有効なデコレーター用途の一つ
* `print()` は開発向き、`logging` は実務・本番向きの出力手段
* 引数や戻り値を明示的に出力することで、関数のトレース性が向上する
* ログ設計にはセキュリティ・環境切り替え・可読性の配慮が欠かせない

このパターンは、デバッグ支援や運用監視、API呼び出しのトレース、エラー再現などあらゆる場面で利用され、**デコレーターの中でも最も実務導入率の高い典型例**である。


<hr>

## 10章：ロギング・プリントを行うデコレーター

---

### 解説

本章では、関数の実行に合わせて**標準出力やログに情報を出力するデコレーター**を設計する。これは、関数の呼び出し・引数・戻り値などの情報を自動的に記録することで、**動作確認・デバッグ・監視**に役立てる目的である。

Pythonの `print()` を使えば手軽に標準出力できる一方で、`logging` モジュールを使えば**ログレベルや出力先を柔軟に制御**できる。本章では、両者の特徴を整理し、実務向けの設計に落とし込む方法を解説する。

---

### プリントベースのデコレーター（軽量）

```python
def print_debug(func):
    def wrapper(*args, **kwargs):
        print(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

適用例：

```python
@print_debug
def greet(name):
    return f"{name}さん、こんにちは"

greet("佐藤")
```

出力：

```
[CALL] greet args=('佐藤',) kwargs={}
[RETURN] greet -> 佐藤さん、こんにちは
```

このように、関数呼び出し前後で**情報を出力するだけの簡易な構成**であり、スクリプトや個人の学習用途に向いている。

---

### ロギングを使う構成（実務向け）

標準出力ではなく `logging` モジュールを使うことで、ログレベル（INFO/WARNING/ERRORなど）や出力先（ファイル、標準出力、syslogなど）を柔軟に設定できる。

```python
import logging

def log_debug(func):
    def wrapper(*args, **kwargs):
        logging.info(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

ログ設定（例）：

```python
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
```

---

### ロギングとプリントの違い

| 観点       | `print()` | `logging`                    |
| -------- | --------- | ---------------------------- |
| 出力先      | 標準出力のみ    | ファイル・標準出力・ストリーム等を選択可         |
| レベル制御    | 不可        | INFO, WARNING, ERROR などで制御可能 |
| 運用環境との連携 | 弱い        | 強い（構成ファイルや外部ツールと連携しやすい）      |
| フィルタリング  | 不可        | ログレベルやフィルタで対象を制限可能           |

**開発中は `print()`、運用や検証では `logging`** という使い分けが一般的である。

---

### 引数・戻り値の詳細な出力

関数の中で何が渡され、何が返ってきたかを正確に追跡するには、次のように書式を整えると可読性が高まる：

```python
def log_detail(func):
    def wrapper(*args, **kwargs):
        print(f"→ 実行関数: {func.__name__}")
        print(f"→ 引数: {args}")
        print(f"→ キーワード引数: {kwargs}")
        result = func(*args, **kwargs)
        print(f"→ 戻り値: {result}")
        return result
    return wrapper
```

これは関数の入出力がブラックボックスにならないよう、**自動的に観測可能にするための実用的手法**である。

---

### 本番環境との切り替え

以下のような設計により、**デバッグ用の出力を環境によって切り替える**ことも可能：

```python
import os

def conditional_log(func):
    def wrapper(*args, **kwargs):
        if os.getenv("DEBUG_MODE") == "1":
            print(f"[DEBUG] {func.__name__} called")
        return func(*args, **kwargs)
    return wrapper
```

これは、**ロジックそのものは変えずに、出力のみ制御したい場合に便利**である。

---

### 注意点と設計上の配慮

| 課題         | 解説                              |
| ---------- | ------------------------------- |
| 機密情報の誤出力   | ログにパスワードやAPIキーが出力されないようにする必要がある |
| デバッグと本番の混同 | 開発時の出力がそのまま本番に残らないよう、設定で制御可能にする |
| 可読性の確保     | フォーマットの統一、情報量の制御が重要（長すぎるログは逆効果） |

---

### まとめ

* ログやプリントは、関数の挙動を観測・検証するために有効なデコレーター用途の一つ
* `print()` は開発向き、`logging` は実務・本番向きの出力手段
* 引数や戻り値を明示的に出力することで、関数のトレース性が向上する
* ログ設計にはセキュリティ・環境切り替え・可読性の配慮が欠かせない

このパターンは、デバッグ支援や運用監視、API呼び出しのトレース、エラー再現などあらゆる場面で利用され、**デコレーターの中でも最も実務導入率の高い典型例**である。

<hr>

## 10章：ロギング・プリントを行うデコレーター

---

### 解説

本章では、関数の実行に合わせて**標準出力やログに情報を出力するデコレーター**を設計する。これは、関数の呼び出し・引数・戻り値などの情報を自動的に記録することで、**動作確認・デバッグ・監視**に役立てる目的である。

Pythonの `print()` を使えば手軽に標準出力できる一方で、`logging` モジュールを使えば**ログレベルや出力先を柔軟に制御**できる。本章では、両者の特徴を整理し、実務向けの設計に落とし込む方法を解説する。

---

### プリントベースのデコレーター（軽量）

```python
def print_debug(func):
    def wrapper(*args, **kwargs):
        print(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

適用例：

```python
@print_debug
def greet(name):
    return f"{name}さん、こんにちは"

greet("佐藤")
```

出力：

```
[CALL] greet args=('佐藤',) kwargs={}
[RETURN] greet -> 佐藤さん、こんにちは
```

このように、関数呼び出し前後で**情報を出力するだけの簡易な構成**であり、スクリプトや個人の学習用途に向いている。

---

### ロギングを使う構成（実務向け）

標準出力ではなく `logging` モジュールを使うことで、ログレベル（INFO/WARNING/ERRORなど）や出力先（ファイル、標準出力、syslogなど）を柔軟に設定できる。

```python
import logging

def log_debug(func):
    def wrapper(*args, **kwargs):
        logging.info(f"[CALL] {func.__name__} args={args} kwargs={kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"[RETURN] {func.__name__} -> {result}")
        return result
    return wrapper
```

ログ設定（例）：

```python
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
```

---

### ロギングとプリントの違い

| 観点       | `print()` | `logging`                    |
| -------- | --------- | ---------------------------- |
| 出力先      | 標準出力のみ    | ファイル・標準出力・ストリーム等を選択可         |
| レベル制御    | 不可        | INFO, WARNING, ERROR などで制御可能 |
| 運用環境との連携 | 弱い        | 強い（構成ファイルや外部ツールと連携しやすい）      |
| フィルタリング  | 不可        | ログレベルやフィルタで対象を制限可能           |

**開発中は `print()`、運用や検証では `logging`** という使い分けが一般的である。

---

### 引数・戻り値の詳細な出力

関数の中で何が渡され、何が返ってきたかを正確に追跡するには、次のように書式を整えると可読性が高まる：

```python
def log_detail(func):
    def wrapper(*args, **kwargs):
        print(f"→ 実行関数: {func.__name__}")
        print(f"→ 引数: {args}")
        print(f"→ キーワード引数: {kwargs}")
        result = func(*args, **kwargs)
        print(f"→ 戻り値: {result}")
        return result
    return wrapper
```

これは関数の入出力がブラックボックスにならないよう、**自動的に観測可能にするための実用的手法**である。

---

### 本番環境との切り替え

以下のような設計により、**デバッグ用の出力を環境によって切り替える**ことも可能：

```python
import os

def conditional_log(func):
    def wrapper(*args, **kwargs):
        if os.getenv("DEBUG_MODE") == "1":
            print(f"[DEBUG] {func.__name__} called")
        return func(*args, **kwargs)
    return wrapper
```

これは、**ロジックそのものは変えずに、出力のみ制御したい場合に便利**である。

---

### 注意点と設計上の配慮

| 課題         | 解説                              |
| ---------- | ------------------------------- |
| 機密情報の誤出力   | ログにパスワードやAPIキーが出力されないようにする必要がある |
| デバッグと本番の混同 | 開発時の出力がそのまま本番に残らないよう、設定で制御可能にする |
| 可読性の確保     | フォーマットの統一、情報量の制御が重要（長すぎるログは逆効果） |

---

### まとめ

* ログやプリントは、関数の挙動を観測・検証するために有効なデコレーター用途の一つ
* `print()` は開発向き、`logging` は実務・本番向きの出力手段
* 引数や戻り値を明示的に出力することで、関数のトレース性が向上する
* ログ設計にはセキュリティ・環境切り替え・可読性の配慮が欠かせない

このパターンは、デバッグ支援や運用監視、API呼び出しのトレース、エラー再現などあらゆる場面で利用され、**デコレーターの中でも最も実務導入率の高い典型例**である。
