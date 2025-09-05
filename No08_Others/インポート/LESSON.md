
# 📘 import01 - importの基礎

---

## ✅ 1.1 importとは

「別のpythonファイル」の定義を読み込み、メインのpythonファイルで使用する仕組み。

1. **再利用性**：一度書いたものをリサイクル。
2. **保守性**：一部がエラーを起こしても、一部の修正で済む。
3. **責任分離**：エラーが起きたとき、どの部分がエラーの原因か分かる。
4. **外部ライブラリ**：デフォルトでimportされないものをimportする。

---

## ✅ 1.2 4つの基本形

| 形式                                               | 説明               |
| ------------------------------------------------ | ---------------- |
| `import module`                                  | モジュール全体をインポート    |
| `from module import function`                    | 関数・クラス単位で直接インポート |
| `import module as module_XXX`                    | モジュールに別名をつける     |
| `from module import function_XXX as function_X3` | 関数に別名をつける        |

---

## ✅ 1.3 import の使用例

### ファイル構成（同一ディレクトリ内）

```plaintext
project/
├── main.py
└── greetings.py
```

### greetings.py

```python
def morning():
    print("こんにちは")
    
def night():
    print("こんばんは")
```

### main.py

```python
# モジュール全体をインポート
import greetings
greetings.morning()
```

```python
# 関数だけimport
from greetings import morning
from greetings import night

morning()
night()
```

```python
# モジュールのエイリアス
import greetings as g

g.morning()
g.night()
```

```python
# 関数のエイリアス
from greetings import morning as m
from greetings import night as n

m()
n()
```

### 出力（全て同じ）

```bash
こんにちは
こんばんは
```

---



# 📘 import02 - import の探索パスと `sys.path` の正体

---

## ✅ 2.1 `sys.path` とは何か？

```python
import sys
print(sys.path)
```

これを実行すると、リストが出力される。これは **Pythonがimport文でモジュールを探す順番のリスト** である。

### 🔹 本質：

> `sys.path` は、Pythonがモジュールを見つけるために「順番に見に行くディレクトリの一覧」を保持している。

* importするとき、Pythonは `sys.path` にあるパスを**上から順番に**探していき、最初に見つかったモジュールを使う。
* 逆に言えば、`sys.path` にそのモジュールのあるディレクトリが無ければ、`ModuleNotFoundError` になる。

---

## ✅ 2.2 `sys.path` に最初から入っているもの

以下のような要素が初期状態で入っている：

1. **スクリプトを実行したディレクトリ（またはそのディレクトリへの空文字列）**
2. **標準ライブラリの場所（例：`/usr/lib/python3.x/`）**
3. **site-packages の場所（pipで入れたライブラリ）**

---

## ✅ 2.3 例：sys.path の中身を確認

```python
import sys
for p in sys.path:
    print(p)
```

例えば次のような出力が得られることがある：

```text
''
'/home/user/myproject'
'/usr/lib/python3.12'
'/home/user/.local/lib/python3.12/site-packages'
```

* `''`（空文字列）は「スクリプトを実行した場所」を意味する。
* 下の方は標準ライブラリ・外部ライブラリの場所。

---

## ✅ 2.4 モジュールが見つからない場合

次のようなエラーになる：

```text
ModuleNotFoundError: No module named 'greetings'
```

この場合、`sys.path` に greetings.py のある場所が含まれていない可能性が高い。

---

## ✅ 2.5 `sys.path` の使い方と追加方法

### 🔸 一時的にパスを追加する

```python
import sys
from pathlib import Path

# このファイルの一つ上の階層を追加
sys.path.append(str(Path(__file__).resolve().parent.parent))
```

注意：

* これは**一時的な対応策**であり、本来はプロジェクト構成自体を見直すべき。
* ただし、試験的にモジュールを動かしたいときには有効。

---

## ✅ 2.6 `PYTHONPATH` とは何か？

環境変数 `PYTHONPATH` を使うと、`sys.path` に**事前に**パスを追加できる：

```bash
$ PYTHONPATH=./src python main.py
```

これは `src/` を import 対象に含める例。ただし環境に依存しやすいため、**常用は非推奨**。

---

## ✅ 2.7 実行場所が import に与える影響

```bash
# 例1
$ python main.py
```

```bash
# 例2
$ cd src && python main.py
```

→ どちらで実行したかによって `sys.path` の最初の要素が変わり、import 成否が変わる。**同じコードでも、どこから実行したかで動かなくなることがある。**

---

## ✅ 2.8 `-m` オプションの有効活用

```bash
$ python -m src.main
```

これを使うと、**プロジェクトルートが自動的に `sys.path` に追加される**ため、import の安定性が増す。

推奨：パッケージ化されたコードは `-m` で実行することを前提に設計すべき。

---



# 📘 import03 - 絶対インポートと相対インポート

---

## ✅ 3.1 インポートの2つの形式

Python には大きく分けて、2つのインポート形式がある。

### 🔹 絶対インポート（推奨）

```python
from src.util.io_json import read_json
```

* ルート（プロジェクトの基準ディレクトリ）からのパスで指定
* **常に明示的で、他人にも分かりやすい**
* ファイルの位置を移動しても壊れにくい（構成が適切であれば）

---

### 🔹 相対インポート（省略的）

```python
from ..util.io_json import read_json
```

* 自分の位置（相対位置）からの距離で指定
* `..` は1階層上、`...` は2階層上を意味する
* **パッケージ化されている（＝上位ディレクトリに`__init__.py`がある）ことが前提**

---

## ✅ 3.2 相対インポートが失敗する理由

相対インポートは、**モジュールが「パッケージの一部として」実行されていること**が条件。

以下のように直接 `.py` を実行すると：

```bash
$ python src/util/io_json.py
```

この場合 `__package__` が空になるため、次のような相対 import はエラーになる：

```python
from ..config import filepath  # ← ValueError / ImportError
```

### ✔ 対策：

* 相対インポートを使うなら **必ず `-m` を使うこと**
* つまり `python -m src.util.io_json` のように、**モジュールとして実行**すれば動作する

---

## ✅ 3.3 相対インポートの構文まとめ

| 構文                         | 意味                               |
| -------------------------- | -------------------------------- |
| `from . import x`          | 同じ階層から `x` を import              |
| `from .. import x`         | 一階層上から `x` を import              |
| `from ..foo import x`      | 一階層上の `foo` モジュールから `x` を import |
| `from ...bar.baz import x` | 二階層上の `bar.baz` から import        |

⚠ `..` の深さがディレクトリ階層に対応していないとエラーになる。

---

## ✅ 3.4 推奨される使い分け

| 規模                   | 推奨される形式            | 理由         |
| -------------------- | ------------------ | ---------- |
| 小規模スクリプト（単一ディレクトリ）   | 相対インポートでも可         | 柔軟に動かせる    |
| 中〜大規模プロジェクト          | 絶対インポートを推奨         | 明確で、移動に強い  |
| パッケージ構成で CLI 実行されるもの | `-m` 実行 + 相対 or 絶対 | 両方動くが管理に注意 |

---

## ✅ 3.5 現場でのベストプラクティス

* **常にプロジェクトルートからの絶対インポートを使うこと**
* `src/` 構成を導入し、VSCode・pytest・CLI全てで import が壊れない構成を設計する（`import09〜10` で解説）
* 相対インポートはテスト用モジュール内部など、**特殊な閉じた関係**でのみ使う

---

## ✅ まとめ

| 観点     | 絶対インポート      | 相対インポート   |
| ------ | ------------ | --------- |
| 明確さ    | 高い           | 低い        |
| 柔軟性    | 高い（構成が正しければ） | 実行方法に依存   |
| 実務での使用 | 基本はこちら       | 使い所が限定される |

---




# 📘 import04 - モジュールとは何か

---

## ✅ 4.1 定義

Pythonにおける「モジュール」とは：

> `.py` 拡張子を持つ1つのPythonファイルであり、
> `import` によって他の場所から再利用可能なコード単位のこと。

---

## ✅ 4.2 例

```text
project/
├── greetings.py
└── main.py
```

### greetings.py

```python
def say_hello():
    return "こんにちは"
```

### main.py

```python
import greetings
print(greetings.say_hello())
```

このとき、`greetings.py` はモジュールである。
`main.py` はモジュールを利用する側（実行スクリプト）である。

---

## ✅ 4.3 目的と役割

* 関数・クラス・定数などを機能ごとに分離する
* 再利用性・可読性・保守性の向上
* 他のモジュールから読み込んで使用する前提の設計

---

## ✅ 4.4 実行時に与えられる属性

モジュールには自動で次のような属性が与えられる：

| 属性名           | 説明                                  |
| ------------- | ----------------------------------- |
| `__name__`    | 実行方法に応じて `"__main__"` または モジュール名になる |
| `__file__`    | ファイルの絶対パス                           |
| `__package__` | パッケージ名（なければ `None` または `""`）        |

### 確認例：

```python
print(__name__)
print(__file__)
print(__package__)
```

#### 実行結果例（直接実行時）

```bash
$ python greetings.py
__main__
/path/to/greetings.py
None
```

#### import 経由での実行

```bash
$ python main.py
greetings
/path/to/greetings.py
（パッケージがあれば名前、なければ None）
```

---

## ✅ 4.5 スクリプトとモジュールの兼用

```python
def say_hello():
    return "こんにちは"

if __name__ == "__main__":
    print(say_hello())
```

* この構造により、直接実行したときだけ `print()` される
* import されたときは `print()` が走らない

---

## ✅ 4.6 命名と設計上の注意点

* ファイル名（＝モジュール名）は小文字 + アンダースコアが基本（PEP8）
* 組み込みモジュールや有名ライブラリと重ならないようにする（例：`string.py` は避ける）
* 1ファイル1責務（目的）にすることで可読性・再利用性が高まる

---

## ✅ まとめ

| 観点 | 内容                                            |
| -- | --------------------------------------------- |
| 定義 | `.py` ファイル1つがモジュール                            |
| 目的 | コードの再利用・分離                                    |
| 特徴 | `__name__`, `__file__`, `__package__` などの属性あり |
| 注意 | 命名、責務の分離、直接実行とimportの両立構造の使い分け                |

---



# 📘 import05 - パッケージとは何か

---

## ✅ 5.1 定義

Pythonにおける「パッケージ」とは：

> **`__init__.py` を含むディレクトリ**
> 複数のモジュール（`.py` ファイル）をまとめる階層単位

* 単なるディレクトリではなく、**Pythonに「import可能な単位」として認識される**
* モジュールの集合体として使うことで、階層的な整理が可能になる

---

## ✅ 5.2 具体例

```text
project/
├── mypackage/
│   ├── __init__.py
│   ├── greetings.py
│   └── farewell.py
└── main.py
```

### greetings.py

```python
def say_hello():
    return "こんにちは"
```

### farewell.py

```python
def say_goodbye():
    return "さようなら"
```

### main.py

```python
from mypackage.greetings import say_hello
from mypackage.farewell import say_goodbye

print(say_hello())
print(say_goodbye())
```

このとき `mypackage` はパッケージであり、`greetings` や `farewell` はその中のモジュール。

---

## ✅ 5.3 `__init__.py` の役割

* **そのディレクトリをパッケージとみなすための印**
* Python 3.3 以降は省略可能だが、**省略すべきではない**

  * `__init__.py` がないと、相対 import や `-m` 実行が不安定になる
* 空でもよいが、以下の用途にも使える：

  * 初期化処理
  * 内部インポートの集約（簡易API化）

### 例：**init**.py に書く

```python
from .greetings import say_hello
from .farewell import say_goodbye
```

これにより、以下のように呼び出せる：

```python
from mypackage import say_hello, say_goodbye
```

---

## ✅ 5.4 階層構造の活用

パッケージは**再帰的に入れ子にできる**：

```text
project/
└── mypackage/
    ├── __init__.py
    ├── greetings/
    │   ├── __init__.py
    │   └── formal.py
    └── farewell.py
```

呼び出し例：

```python
from mypackage.greetings.formal import say_polite_hello
```

---

## ✅ 5.5 命名と設計の指針

* ディレクトリ名（＝パッケージ名）はモジュール同様、**小文字 + アンダースコア**
* `mypackage/__init__.py` は**空でも必ず置く**
* パッケージ階層が深くなりすぎないよう注意する
* インポート経路が循環しないよう、設計段階で分割を検討

---

## ✅ まとめ

| 観点     | 内容                                |
| ------ | --------------------------------- |
| 定義     | `__init__.py` を含むディレクトリ           |
| 役割     | 複数モジュールの構造的な整理                    |
| 必須要素   | `__init__.py`（空でもよい）              |
| 実務上の推奨 | 省略不可。全階層に明示的に置くこと                 |
| インポート例 | `from package.module import func` |

---




# 📘 import06 - ライブラリとは何か

---

## ✅ 6.1 「ライブラリ」の定義は何か？

Python公式には**明確な技術的定義は存在しない**。
しかし実務では次のように理解されている：

> **再利用可能なコード群をまとめた論理的・配布的な単位**

* 複数のパッケージやモジュールを含む
* 他者に提供されることを前提とした再利用可能なソフトウェア
* 多くは PyPI 経由でインストール可能な形を取る

---

## ✅ 6.2 典型例

| 名前         | 内容                   |
| ---------- | -------------------- |
| `requests` | HTTP通信の高レベルラッパー      |
| `numpy`    | 配列処理・線形代数を高速に処理      |
| `pandas`   | 表形式データの分析処理          |
| `flask`    | 軽量Webアプリケーションフレームワーク |

いずれも：

* 複数の `.py` ファイルやディレクトリ（＝モジュールやパッケージ）で構成されている
* 組織的に設計されており、明確な外部API（import経路）が用意されている

---

## ✅ 6.3 実体としての構成

たとえば `requests` の内部構成：

```text
requests/
├── __init__.py
├── api.py
├── models.py
├── sessions.py
├── utils.py
└── ...
```

* これらを含めて「requestsライブラリ」と呼ぶ
* **ライブラリ = パッケージ構造を含む、外部配布前提の設計物**

---

## ✅ 6.4 自作コードとライブラリの違い

| 観点       | 自作モジュール    | ライブラリ                      |
| -------- | ---------- | -------------------------- |
| 配布前提     | なし         | ある（pipなど）                  |
| 公開範囲     | 自分 or チーム内 | 不特定多数（または広範）               |
| 設計指針     | 実行主導       | import主導・公開API設計           |
| ディレクトリ構成 | 任意         | 基本的にパッケージ構成（`__init__.py`） |

---

## ✅ 6.5 自作プロジェクトがライブラリ化される条件

* パッケージ構成で整理されており、
* 外部から import 可能なモジュール階層が安定していて、
* ユーザーが `from yourlib.submod import x` のように使えるよう設計されていれば、

その自作プロジェクトはライブラリと見なせる。

---

## ✅ 6.6 ライブラリとフレームワークの違い（参考）

* ライブラリ：**自分が呼び出すもの**（例：`requests.get()`）
* フレームワーク：**相手に呼ばれる前提の構造**（例：Flaskルーティング）

---

## ✅ まとめ

| 観点      | 内容                                                |
| ------- | ------------------------------------------------- |
| ライブラリとは | モジュール・パッケージを含む再利用可能な配布単位                          |
| 定義の厳密さ  | 曖昧だが実務では通用している用語                                  |
| 構成要素    | 複数の `.py` ファイル（≒モジュール群）                           |
| 実務上の使い方 | `import ライブラリ名`、または `from ライブラリ.サブモジュール import x` |

---




# 📘 import07 - 主要概念の違いと誤解の整理

（GitHub公開・クローン利用を想定）

---

## ✅ 7.1 用語の混同を正す

| 用語    | 定義（Pythonにおける扱い）                    |
| ----- | ----------------------------------- |
| モジュール | `.py` ファイル1つ                        |
| パッケージ | `__init__.py` を含むディレクトリ             |
| ライブラリ | モジュールやパッケージの集まり。配布・再利用を想定した構造（抽象概念） |

Pythonにおける「ライブラリ」は形式的な区切りがなく、**構造（モジュール＋パッケージ）と運用（配布・再利用設計）を兼ね備えたもの**である。

---

## ✅ 7.2 実務での使い分け視点

### ✔ モジュール（単独ファイル）

* 学習・ツール用途、CLIスクリプトに多い
* import での再利用は限定的（構造化されていないため）

### ✔ パッケージ（再利用設計された階層）

* 他ファイルから `import a.b.c` として明示的に利用可能
* `__init__.py` を置くことで import 解決が安定する

### ✔ ライブラリ（GitHub公開を想定）

* 複数のパッケージ／モジュールから構成され、
* 他者が `git clone` → `import` して使えるようになっているもの

---

## ✅ 7.3 自作プロジェクトを GitHub に公開して再利用させるために必要な構造

### 例：正しいディレクトリ構成（`mytool` というプロジェクトを想定）

```text
mytool/
├── src/
│   └── mytool/
│       ├── __init__.py
│       ├── core.py
│       └── util.py
├── tests/
│   └── test_core.py
├── pyproject.toml
├── README.md
└── .gitignore
```

* `src/` ディレクトリでルートを切るのは業界標準構成の一つ
* 実際のライブラリ本体は `src/mytool/` 以下にまとめる
* 外部の利用者が `from mytool.core import func` のように使えるようにする

---

## ✅ 7.4 ユーザーが clone して使う場合の import 問題

### ❌ よくある失敗例

```bash
$ git clone https://github.com/user/mytool.git
$ cd mytool
$ python src/mytool/core.py
```

```text
ModuleNotFoundError: No module named 'mytool'
```

→ 原因：`src/` ディレクトリが `sys.path` に含まれていない
→ Pythonは `src/` 以下の `mytool` を探索できていない

---

## ✅ 7.5 解決策（3通り）

### 方法1：`-m` を使ってモジュールとして実行（最善）

```bash
$ python -m src.mytool.core
```

* Pythonが `src/` をルートとみなして `mytool` を正しく import できる
* クローンユーザーに推奨すべき手法

---

### 方法2：`PYTHONPATH` を使う（非推奨）

```bash
$ PYTHONPATH=src python src/mytool/core.py
```

* 環境依存しやすく、長期運用や複数環境では破綻しやすい

---

### 方法3：`sys.path` をコード上で追加（短期対応）

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parent.parent))
```

* 動作はするが、本来は**構造を見直すべき**ケース

---

## ✅ 7.6 依存しないための設計原則

* **必ずすべてのパッケージディレクトリに `__init__.py` を置く**
* import は**絶対インポート**に統一する（例：`from mytool.core import x`）
* README に実行例を書くときは必ず `-m` を用いた形式にする

  * ❌ `python src/mytool/core.py`
  * ✅ `python -m src.mytool.core`

---

## ✅ 7.7 誤解されやすい点の整理

| 誤解                        | 正確な理解                         |
| ------------------------- | ----------------------------- |
| モジュール＝ライブラリ               | モジュールは1ファイル、ライブラリは設計単位        |
| `import` はファイルパスに従う       | `import` はモジュールパスに従う（`.` 区切り） |
| `__init__.py` は不要         | 必須ではないが、実務では安定性のため必須          |
| `python file.py` で実行すればよい | `-m` を使わないと import 解決が壊れる     |

---

## ✅ 7.8 clone 利用前提のベストプラクティス

| 項目       | 指針                             |
| -------- | ------------------------------ |
| ディレクトリ構成 | `src/yourlib/` の構造に分離する        |
| インポート方法  | 絶対インポートに統一                     |
| 実行方法     | `python -m src.yourlib.module` |
| ドキュメント   | `README.md` に import / 実行方法を明記 |
| パス追加     | `sys.path` 依存は避け、構造で解決すること     |

---

## ✅ まとめ

* ライブラリとは、構造化され再利用設計されたコード集合である
* モジュール（ファイル）・パッケージ（ディレクトリ）から構成される
* GitHub上に公開する場合でも、import構造とパス設計を誤ると即時に破綻する
* `src/` 構成と `-m` 実行の原則に従うことで、clone → import の再現性を担保できる

---



# 📘 import08 - 公開プロジェクトにおけるimport構造の最適化

（GitHubでのclone利用を前提とした設計方針）

---

## ✅ 8.1 公開時に考慮すべき構造上の課題

PythonプロジェクトをGitHubに公開する際には、他のユーザーが `git clone` 後に即座にコードを実行・利用できる構造になっているかどうかを重視すべきです。とくに、以下のような手順でコードが利用されることを想定する必要があります。

```bash
git clone https://github.com/yourname/yourproject.git
cd yourproject
python scripts/example.py
```

このような使われ方において、import構造に不備があると `ModuleNotFoundError` を引き起こし、第三者にとって再現性が低いプロジェクトと判断される可能性があります。よって、**importの安定性と実行環境の前提**を明確に設計しておくことが求められます。

---

## ✅ 8.2 srcディレクトリ構成の意図と利点

業界では `src/` ディレクトリを導入する構成が一般的となっています。これは、**ライブラリ本体と周辺ファイル（tests, scripts など）を明確に分離**するためです。以下はその構成例です。

```text
yourproject/
├── src/
│   └── yourlib/
│       ├── __init__.py
│       ├── core.py
│       └── util.py
├── scripts/
│   └── example.py
├── tests/
│   └── test_core.py
├── pyproject.toml
├── README.md
└── .gitignore
```

この構成を採用することで、次のような利点があります：

* `src/` 以下のみを import 対象とできるため、ビルドや配布時の対象を制御しやすい
* テスト・ユーティリティ・補助スクリプトなどを論理的に分離できる
* `src/yourlib/` の外部利用を想定した import パス設計が可能になる

---

## ✅ 8.3 scriptsからのimportが失敗する理由

クローンしたばかりのユーザーが以下のようにファイルを実行すると、しばしば import に失敗します。

```bash
python scripts/example.py
```

```text
ModuleNotFoundError: No module named 'yourlib'
```

このエラーは、Pythonが `sys.path` に `src/` を含んでいないために、`yourlib` パッケージを見つけられないことが原因です。Pythonは、スクリプトのあるディレクトリを基準にパスを解決するため、ルート構成やモジュール設計が不適切だと、こうしたエラーが発生します。

---

## ✅ 8.4 推奨される実行方法と構造の調整

### 方法1：`-m` を用いたモジュール実行（最も安全）

```bash
cd yourproject
python -m src.yourlib.core
```

この形式では、Pythonが `yourproject/` を基点としたモジュール解決を行い、`src/` 以下のパッケージが正しく認識されます。プロジェクトの README に記載すべき実行形式も、この方式に統一することが望まれます。

---

## ✅ 8.5 sys.path操作やPYTHONPATHに頼る構成の問題点

### 方法2：環境変数 `PYTHONPATH` に `src/` を指定（非推奨）

```bash
PYTHONPATH=src python scripts/example.py
```

この方法は一時的には機能しますが、OSやシェル、CI/CDの設定に依存しやすく、実行環境が変わると動作が破綻するリスクがあります。

### 方法3：`sys.path` をスクリプト内で操作（暫定的な対応）

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parents[1]))
```

この対応も、構造が適切に設計されていない場合の応急処置にすぎません。あくまで短期的な回避策であり、根本的な解決策ではありません。

---

## ✅ 8.6 公開ライブラリとして必要な要素

ライブラリとして他者に使われることを前提にした設計では、以下の点を意識してください：

| 項目       | 推奨事項                                                       |
| -------- | ---------------------------------------------------------- |
| ディレクトリ構成 | `src/yourlib/` に集約し、論理的な名前空間を維持する                          |
| import方法 | 相対importは避け、すべて絶対import（`from yourlib.module import x`）に統一 |
| 実行方法     | `-m` を使ったモジュール実行形式でREADMEに記載                               |
| initファイル | すべてのパッケージディレクトリに `__init__.py` を配置                         |
| パス設定     | `PYTHONPATH` や `sys.path` への依存を避け、構造設計で対応                  |
| 再現性      | クローン後に即 `pytest` や `python -m ...` が動作する状態にしておく            |

---

## ✅ 8.7 まとめ：GitHub公開とimport設計の指針

* 公開プロジェクトは **再現性** を最優先に設計すべきです
* `src/` ディレクトリを基点とした構造を採用し、ルートからの絶対importを徹底することが重要です
* `scripts/` からの直接実行ではなく、`python -m` を基本とする運用をドキュメントで明示してください
* importエラーを「実行者の責任」とせず、「構造で回避」する姿勢が必要です

---




# 📘 import09 - 絶対・相対インポート


---

## ✅ 9.1 絶対インポートとは何か

絶対インポートとは、**プロジェクトルートを基準にパッケージ階層を明示するimport方法**です。具体的には、以下のような形式になります。

```python
from yourlib.core import some_function
```

ここでの `yourlib` は、たとえば `src/yourlib/` にあるルートパッケージを指します。Pythonは `sys.path` に含まれるルートからモジュールを探索します。

絶対インポートは、**ファイルの位置に依存せず、プロジェクト全体の構造に対して一貫性を持たせることが可能**であるため、以下の点で有利です。

* importの読みやすさ（どこから来たかが一目瞭然）
* 外部ユーザーやCI/CD環境でも同じ構文で利用可能
* `-m` 実行との親和性が高い

---

## ✅ 9.2 相対インポートとは何か

相対インポートとは、**現在のモジュールの場所を基準として、相対的にimportを行う方法**です。ドット（`.`）やドット2つ（`..`）を用います。

```python
from .util import helper_function         # 同一パッケージ内
from ..core import base_class             # 1階層上の別モジュール
```

相対インポートは、内部モジュール同士の関係が明確なときには有効です。ただし、以下の制約があります。

* スクリプトとして直接実行（`python file.py`）すると失敗する
* `-m` を用いた実行でのみ機能が保証される
* 依存方向が明示されず、構造を変更すると破綻しやすい

したがって、**スクリプトの直接実行と相性が悪く、再利用性にも劣る**ため、原則として避けるべきです。

---

## ✅ 9.3 実行形式による違いとimport失敗の原因

Pythonでは、スクリプトの実行方法によって import の解決方法が変化します。

| 実行形式                       | import解決基準        | 相対import | 絶対import |
| -------------------------- | ----------------- | -------- | -------- |
| `python yourlib/module.py` | `module.py`の親フォルダ | ❌ 壊れる    | ❌ 壊れる    |
| `python -m yourlib.module` | プロジェクトルート         | ✅ 動作     | ✅ 動作     |

このように、**スクリプト直接実行では相対・絶対どちらも不安定**になることが多いため、基本的には `-m` を使う実行方法に統一することが重要です。

---

## ✅ 9.4 実務での使い分け方針（原則）

| 利用場面        | 推奨されるimport手法          |
| ----------- | ---------------------- |
| 公開ライブラリ開発   | 絶対import（構造が安定するため）    |
| テスト・補助スクリプト | 絶対import（pytestと相性良好）  |
| 小規模な個人スクリプト | 相対importも容認可能          |
| スクリプト直接実行   | import方式ではなく実行方法を見直すべき |

---

## ✅ 9.5 相対インポートを選ぶべきでない理由（詳細）

* `.`, `..` の組み合わせが複雑になると読みづらい
* モジュールの移動や分割に弱く、importが壊れやすい
* 他者がcloneして使う際、import失敗の原因になる
* IDEや補完機能との相性が悪くなるケースがある

これらの理由から、**相対importは原則として避け、絶対importに統一することが望ましい**です。

---

## ✅ 9.6 絶対インポート統一の運用ルール

以下のような開発運用を取り入れることで、import構造を安定させることができます。

* `src/` ディレクトリ構成を採用し、ビルド・配布対象を明確化する
* 全モジュールは `yourlib` 以下に集約する
* `pytest`, `mypy`, `black` などのツールを、`src/` をルートとした形で設定する
* `README.md` で import 構文や実行方法を明示する

---

## ✅ 9.7 まとめ

* 絶対インポートは、構造の可読性と保守性に優れるため、公開前提のプロジェクトでは必須である
* 相対インポートは限定的にのみ許容され、開発中の一時的用途にとどめるべきである
* importに失敗する多くの原因は、実行方法（`python file.py`）とimport設計の不整合にある
* `python -m yourlib.module` 形式での実行を基本とし、構造と実行形式の整合性を保つことで、安定した運用が可能になる

---



# 📘 import10 - `__name__ == "__main__"` 
（スクリプト実行の制御・テスト・CLIへの応用）

---

## ✅ 10.1 `__name__` とは何か

Pythonでは、すべてのモジュールに対して `__name__` という特殊変数が自動的に定義されます。
この変数は、次のように2通りの値を取ります。

| 実行状況                  | `__name__` の値              |
| --------------------- | -------------------------- |
| スクリプトとして直接実行された場合     | `"__main__"`               |
| 他のファイルから import された場合 | モジュール名（例：`yourlib.module`） |

この性質を利用することで、**「直接実行されたときだけ実行される処理」** を記述できます。

---

## ✅ 10.2 基本的な使用例

以下は典型的な使用例です。

```python
def main():
    print("このモジュールは直接実行されました")

if __name__ == "__main__":
    main()
```

このコードは、`python this_file.py` で実行されたときのみ `main()` を実行します。
一方で、他のモジュールから `import` された場合は `main()` が実行されません。

---

## ✅ 10.3 利用する意義

この構文を使うことで、次のような利点が得られます。

* **スクリプトとしての実行**と**モジュールとしての利用**を分離できる
* テストコードから import したときに副作用（printなど）を防げる
* CLIツール化する際の入口関数を定義できる
* `python -m package.module` 形式と組み合わせて構造化された実行が可能

---

## ✅ 10.4 `-m` 実行との併用

`__main__` を使う際に重要となるのが、`-m` を使ったモジュール実行です。

```bash
python -m src.yourlib.module
```

このように実行された場合も、実行されるモジュール内の `__name__` は `"__main__"` となります。
つまり `-m` 形式は **import構造を保ちつつ `__main__` を起動できる唯一の方法**です。

---

## ✅ 10.5 CLIのエントリーポイントとして使う

`__main__` を利用して、コマンドライン引数を受け取る CLI スクリプトの入口を定義することも可能です。

```python
import sys

def main():
    print("引数:", sys.argv)

if __name__ == "__main__":
    main()
```

これにより、Pythonファイルを直接 CLI 実行可能にすることができます。

```bash
python script.py arg1 arg2
```

---

## ✅ 10.6 テストとの分離・副作用の抑制

`if __name__ == "__main__"` によって、ファイル内のコード実行を明示的に制御できるため、pytest などでテストするときにも余計な出力や処理が発生しません。

```python
# test_target.py から import されても print が走らない
if __name__ == "__main__":
    print("このコードは直接実行されたときだけ表示されます")
```

これは **テスト時の安定性確保・再利用性の向上** に直結します。

---

## ✅ 10.7 ディレクトリ構成と `__main__.py`

パッケージ全体を直接実行可能にしたい場合は、パッケージの直下に `__main__.py` を設置します。

```text
yourlib/
└── __main__.py
```

これにより、以下のように **パッケージ名ごと実行**することができます。

```bash
python -m yourlib
```

この形式はCLIツールのエントリーポイントとしてよく使われ、`argparse` や `click` と組み合わせると強力なインターフェースを構築できます。

---

## ✅ 10.8 実務での設計指針

| 利用目的      | 設計指針                                       |
| --------- | ------------------------------------------ |
| スクリプト単体実行 | `if __name__ == "__main__"` で明示的に起動関数を定義   |
| ライブラリ設計   | モジュールは import 専用とし、副作用を抑える                 |
| CLI対応     | `main()` を持つ構造とし、`argparse` / `click` 等と連携 |
| サブコマンド形式  | `__main__.py` に CLI 全体の入口処理を記述             |

---

## ✅ まとめ

* `__name__ == "__main__"` は、直接実行時だけコードを起動させるための構文です
* スクリプト実行とモジュール利用を明確に分けることができ、保守性と再利用性が高まります
* `-m` 形式と組み合わせることで、構造を壊さずにスクリプトを実行できます
* CLI設計やテスト時の副作用抑制において、必須の構文であると言えます

---




# 📘 import11 - `__main__` とCLIライブラリ

（`argparse`・`click` を用いた構造化CLIの構築）

---

## ✅ 11.1 なぜCLI構造と組み合わせるのか

Pythonのスクリプトに `if __name__ == "__main__"` を使うことで、直接実行時のみ動作するエントリーポイントを定義できます。
この構文と CLI ライブラリ（例：`argparse` や `click`）を組み合わせることで、**再利用可能かつ実行制御可能なツール**を構築できます。

CLI対応を行う主な理由は以下の通りです：

* 引数による条件分岐やファイル指定など、**外部からの制御**が可能になる
* テストやimport時には副作用を起こさない構造を維持できる
* `-m` 実行形式との組み合わせで、**構造を保ったまま実行可能**

---

## ✅ 11.2 最小構成のCLIスクリプト（argparse使用）

```python
import argparse

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--name", help="ユーザー名")
    args = parser.parse_args()
    print(f"こんにちは、{args.name} さん")

if __name__ == "__main__":
    main()
```

```bash
$ python script.py --name 君
こんにちは、君 さん
```

この構造は `if __name__ == "__main__"` により、他ファイルから import されたときには `main()` が実行されないようになっています。

---

## ✅ 11.3 `-m` 実行への対応

CLIツールを構造化し、`python -m yourlib.module` 形式で実行したい場合、次のようにします。

```text
yourproject/
└── src/
    └── yourlib/
        ├── __init__.py
        └── cli.py  ← ここにmain関数を定義
```

```python
# src/yourlib/cli.py

def main():
    print("CLIモジュールが起動されました")

if __name__ == "__main__":
    main()
```

```bash
python -m src.yourlib.cli
```

この方法であれば、構造を壊さずCLI処理を実行できます。

---

## ✅ 11.4 複数のサブコマンドに対応する設計

`argparse` では `add_subparsers()` を使うことで、次のような形式にできます。

```bash
python -m src.yourlib.cli fetch
python -m src.yourlib.cli clean
```

```python
def main():
    parser = argparse.ArgumentParser()
    subparsers = parser.add_subparsers(dest="command")

    fetch_parser = subparsers.add_parser("fetch")
    clean_parser = subparsers.add_parser("clean")

    args = parser.parse_args()
    if args.command == "fetch":
        print("データ取得処理を開始")
    elif args.command == "clean":
        print("整形処理を実行")
```

---

## ✅ 11.5 `click` を使った構造化（推奨）

`click` を使うと、より明確かつPythonicにCLIを記述できます。

```python
import click

@click.group()
def cli():
    pass

@cli.command()
def fetch():
    click.echo("データ取得処理")

@cli.command()
def clean():
    click.echo("整形処理")

if __name__ == "__main__":
    cli()
```

```bash
python -m src.yourlib.cli fetch
python -m src.yourlib.cli clean
```

---

## ✅ 11.6 **main**.py との統合

以下のように `__main__.py` を配置すれば、パッケージ全体をコマンドとして扱うことができます。

```text
yourlib/
├── __init__.py
├── __main__.py
└── cli.py
```

```python
# yourlib/__main__.py
from yourlib.cli import cli

if __name__ == "__main__":
    cli()
```

```bash
python -m yourlib fetch
```

この形式は `click` との親和性が高く、CLIツールとしての配布や再利用に向いています。

---

## ✅ 11.7 テストやimport時の安全性

CLI構造を `main()` や `cli()` に閉じ込めておくことで、pytest や他モジュールからの import 時に副作用が起きません。これにより、**再利用性・保守性・テスト容易性**が大きく向上します。

---

## ✅ まとめ

* `__main__` と CLI ライブラリを組み合わせることで、安全かつ再利用可能なツールを作成できます
* `-m` 形式と `click` や `argparse` の併用により、構造を壊さずにCLIを構築できます
* `__main__.py` を導入することで、パッケージごとCLIとして実行可能になります
* CLI設計は import構造と実行設計の交差点であり、安易なスクリプト直接実行に頼るべきではありません

---



# 📘 import12 - PYTHONPATH

---

## ✅ 12.1 `PYTHONPATH` とは何か

`PYTHONPATH` は、Pythonがモジュールを検索する際のパスを追加で指定するための環境変数です。
通常、Pythonは以下の順序でモジュールを探します：

1. スクリプトのあるディレクトリ
2. 標準ライブラリのディレクトリ
3. `site-packages` 等のインストール済みパッケージ
4. `PYTHONPATH` に指定されたパス（任意）

たとえば、次のようにすれば、`src/` を検索対象に追加できます。

```bash
PYTHONPATH=src python scripts/example.py
```

---

## ✅ 12.2 なぜ `PYTHONPATH` に頼る設計が危険か

`PYTHONPATH` は一見便利ですが、以下のような問題点があります。

* **環境依存性**が高く、OSやシェル、エディタ設定に左右される
* `.env` や VSCode の launch.json、CI設定など、個別に管理が必要
* 他者がcloneしても同じ挙動が保証されず、**再現性が崩壊する**
* 他の方法（`-m` や構造設計）で代替可能なケースが多い

したがって、**長期運用やチーム開発においては非推奨**です。

---

## ✅ 12.3 `.env` による間接指定（VSCodeやdotenv使用時）

ローカル開発環境では、`.env` ファイルを使って `PYTHONPATH` を間接的に設定することがあります。

```
# .env
PYTHONPATH=src
```

このファイルを VSCode の拡張機能（Python環境）や `python-dotenv` で読み込むことで、環境変数を一時的に反映できます。

```python
# scripts/example.py
from dotenv import load_dotenv
load_dotenv()
```

ただしこの方法も、**コード外で依存関係が発生する**という意味で、安全とは言えません。

---

## ✅ 12.4 恒久的な対応：構造と実行方法の見直し

環境設定によるimport制御は避け、構造と実行方法によって解決するのが根本的な対策です。

| 問題                | 推奨される設計対応                          |
| ----------------- | ---------------------------------- |
| importエラーが起きる     | `src/`構成を導入し、パッケージ分離               |
| `scripts/`から実行したい | `python -m src.yourlib.module` に統一 |
| 他者のclone環境で失敗     | `README.md`に統一実行例を記述               |

このように、**パス設定に依存しない構造と運用**がベストプラクティスです。

---

## ✅ 12.5 どうしても `sys.path` や `PYTHONPATH` を使う場合

限定的・短期的にパスを調整したい場合は、次のような記述を一時的に加えることがあります。

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parents[1]))
```

また、`PYTHONPATH` に関しては `Makefile` や `tox.ini` の中で明示的に設定することで、環境依存を少し緩和できます。ただし、これはあくまで**構造的な問題を回避できないときの妥協策**であるべきです。

---

## ✅ 12.6 テスト環境における対応（pytestなど）

テスト時にも import 問題は発生します。以下のいずれかで対応することが多いです。

* ルート直下で `pytest` を実行し、`src/` を自動的に含める
* `pytest.ini` または `pyproject.toml` に以下を記述：

```toml
[tool.pytest.ini_options]
pythonpath = "src"
```

これにより、テストランナーが `src/` を自動でパスに含めるようになります。

---

## ✅ まとめ

* `PYTHONPATH` によるパス解決は一時的には有効ですが、再現性・可搬性の観点から非推奨です
* `.env` や `launch.json` も開発者依存の設定であり、本質的な解決とは言えません
* 根本的には、構造（`src/`採用）と実行形式（`-m`使用）で解決すべきです
* テスト・CLI・本番実行を含めた**一貫性のあるimport設計**が、安定運用には不可欠です

---



# 📘 import13 - トラブル事例

（循環import・再帰import・実行形式不一致などの対処）

---

## ✅ 13.1 よくある import トラブルの分類

Pythonのimportは柔軟な一方で、以下のようなパターンで問題が発生しやすいです。

| 種別        | 主な原因例                               |
| --------- | ----------------------------------- |
| 循環import  | 相互にimportしあってしまう構造                  |
| 再帰import  | 間接的な呼び出しが循環構造になり、stack overflowに近づく |
| 実行形式の不一致  | `-m` を使わない直接実行などでパスが崩れる             |
| モジュール名の衝突 | `json.py` や `test.py` のような名前による衝突   |
| import失敗  | sys.path未設定・構造不備・実行場所のズレ            |

---

## ✅ 13.2 循環import（circular import）

### 例：core.pyとutil.pyが互いに依存

```python
# core.py
from util import helper
def run(): ...
```

```python
# util.py
from core import run
def helper(): ...
```

このように **互いにimportし合う構造**は、モジュールの初期化中に `NameError` や `ImportError` を引き起こすことがあります。

### 解決策：

* **依存方向を一方通行に制限**する（util → core のみにする）
* import文を **関数内に遅延移動**する：

```python
def helper():
    from core import run
    run()
```

* 共通部分を **別モジュールに抽出**し、両者から利用する（`common.py` など）

---

## ✅ 13.3 実行形式の不一致

以下のように `python path/to/file.py` の形式で直接実行すると、絶対importが機能せず失敗します。

```bash
$ python src/yourlib/module.py
ModuleNotFoundError: No module named 'yourlib'
```

### 解決策：

* 必ず **ルートで `-m` を使う実行形式**に統一する：

```bash
$ python -m src.yourlib.module
```

* `README.md` にこの形式を明記し、プロジェクトの基本運用とすること

---

## ✅ 13.4 モジュール名の衝突

標準ライブラリや既存パッケージと同名の `.py` ファイルを作ると、**import解決が崩れます**。

例：自作ファイル `json.py`

```python
import json  # ← 自分自身のjson.pyがimportされる
```

### 解決策：

* `json.py`, `time.py`, `email.py` など、**標準ライブラリと同名を避ける**
* 衝突が起きたら、`print(json.__file__)` などでimport先を確認する

---

## ✅ 13.5 相対importと絶対importの混在

同じプロジェクト内で、以下のように記述方法が混在すると構造変更時に破綻します。

```python
# A.py
from .B import func1  # 相対import

# C.py
from yourlib.B import func1  # 絶対import
```

### 解決策：

* **絶対importに統一**すること（第9章・第10章参照）
* 相対importは限定的用途（同一ディレクトリの一時利用）にとどめる

---

## ✅ 13.6 デバッグ時に有効な方法

| 方法                                   | 説明                           |
| ------------------------------------ | ---------------------------- |
| `print(__name__)`                    | 実行中のモジュールが `__main__` か確認できる |
| `print(sys.path)`                    | 現在のモジュール探索パスを一覧できる           |
| `importlib.util.find_spec("module")` | import解決の可否を確認できる            |
| `python -i file.py`                  | 対話モードで import の流れを確認可能       |
| `PYTHONPATH=.`                       | 一時的に現在ディレクトリを明示的にパスに含める      |

---

## ✅ 13.7 pytest実行時の import 問題

pytest の実行方法によっても import 問題が発生します。

### ❌ 失敗しやすい例

```bash
cd tests/
pytest test_module.py
```

→ `src/` がパスに入らず import 失敗

### ✅ 推奨例

```bash
cd yourproject/
pytest
```

または、`pytest.ini` や `pyproject.toml` に以下を記述：

```toml
[tool.pytest.ini_options]
pythonpath = "src"
```

---

## ✅ まとめ

* importトラブルの多くは構造不備か実行形式の不一致に起因します
* 循環importや衝突は **構造の分離** や **依存方向の明確化** により回避可能です
* 実行時は `-m` を基本とし、再現性のあるimport解決を前提に設計すべきです
* 一時的なトラブルには `print(__name__)`, `sys.path`, `importlib` での調査が有効です

---




# 📘 import14 - 動的importと危険性

（importlib・import・reload・eval的実行の整理）

---

## ✅ 14.1 動的importとは何か

動的importとは、**実行時に文字列などを用いてimport処理を行う技法**です。
静的な`import`文とは異なり、ファイル名やユーザー入力に基づいてモジュールを読み込むことができます。

```python
import importlib

module = importlib.import_module("yourlib.module")
```

この手法は、**プラグイン構造やCLIオプションによる分岐**など、柔軟性が求められる場面で使われますが、誤用すると構造が破綻しやすくなります。

---

## ✅ 14.2 `importlib.import_module()` の基本

```python
import importlib

modname = "yourlib.util"
mod = importlib.import_module(modname)
mod.some_function()
```

この関数は、**ドット区切りのモジュールパス文字列を動的に解決**します。
内部的には `__import__` を呼び出していますが、通常はこちらの使用が推奨されます。

---

## ✅ 14.3 `__import__()` の低水準API（推奨されない）

```python
mod = __import__("yourlib.util")
```

`__import__()` はPythonの内部APIであり、正確な挙動は複雑です。
モジュール名がドット区切りの場合、**最上位モジュールしか返されない**ため扱いづらく、原則として `importlib.import_module()` を使用する方が安全です。

---

## ✅ 14.4 モジュールの再読み込み（reload）

モジュールを再読み込みしたい場合は、次のようにします。

```python
import importlib
import yourlib.module

importlib.reload(yourlib.module)
```

ただし、**状態を保持しているオブジェクトやクラスが再読み込み後も古い定義を保持する**可能性があるため、基本的には開発・デバッグ用途に限定すべきです。

---

## ✅ 14.5 eval的なimport処理の例

ユーザーの入力に応じて関数を動的に呼び出す構造を作ることも可能です。

```python
def run(name: str):
    module = importlib.import_module(f"commands.{name}")
    module.main()
```

これはCLIや設定ファイルから呼び出す構造として有効ですが、次のような**リスク**があります：

* モジュールが存在しない場合の例外処理が必要
* import時に副作用が起こる構成だと安全性が低い
* 実行順・構造が把握しづらく、**保守性が大幅に低下する**

---

## ✅ 14.6 想定される実務での用途

| 用途          | 利点                  | 注意点                     |
| ----------- | ------------------- | ----------------------- |
| プラグイン読み込み   | 柔軟な構成変更が可能          | import構造が不明瞭になる、補完が効かない |
| 設定ファイルによる実行 | ユーザー定義に応じて処理分岐が可能   | 実行前にモジュール名の妥当性検証が必要     |
| 自動テスト構築     | テスト対象を自動で読み込んで実行できる | 統一された構造・命名規約がないと破綻しやすい  |

---

## ✅ 14.7 危険な構文：`exec` と `eval`

```python
exec("import yourlib.util")
eval("yourlib.util.some_function()")
```

これらは**コードとしての安全性が保証されない**ため、**外部入力や設定に基づいて絶対に使ってはならない構文**です。
どうしても必要な場合は、`importlib` による明示的な制御へ置き換えてください。

---

## ✅ 14.8 静的解析との相性の悪さ

動的importを多用すると、以下のような実害があります。

* mypyやpyrightなどの型チェッカーが解析不能になる
* auto-completion（補完）が効かなくなる
* VSCodeなどでの「定義へ移動」「リファクタ」が不可能になる

これにより、**大規模開発や保守では致命的な設計負債**となる可能性が高いです。

---

## ✅ 14.9 設計指針としての判断基準

| 条件                        | 動的importの可否     |
| ------------------------- | --------------- |
| 実行時の柔軟なプラグイン切り替えが必要な場合    | 可（構造を明示し、限定用途に） |
| テスト対象を一覧から自動読み込みする必要がある場合 | 可（命名規約と例外処理を整備） |
| モジュール構造が複雑、依存が深い場合        | 不可（構造を分割・明示すべき） |
| 補完や型検査を優先したい場合            | 不可（静的importが必須） |

---

## ✅ まとめ

* `importlib.import_module()` による動的importは、**制限付きで有効**
* `__import__` や `exec`・`eval` は原則として避ける
* 再読み込みは開発中限定であり、本番では使用しない方が安全
* 補完・静的解析・保守性を損なうため、**動的importは慎重に限定用途で運用**すべきである

---


# 📘 import15 - パッケージ化

（共通処理の分離・構造的分割・依存方向の整理）

---

## ✅ 15.1 なぜパッケージ化が重要か

小規模なスクリプトでは、単一ファイルでも十分に管理可能です。しかし、機能の拡張や再利用を考えると、以下のような課題が顕在化します：

* 同じ関数や定数の重複定義
* importパスの混乱や循環依存
* テストや実行コードの分離困難
* CLI・APIなど複数の実行系統への対応困難

これらを避けるためには、**明示的なパッケージ設計とimport整理が不可欠**です。

---

## ✅ 15.2 サブパッケージの基本設計方針

パッケージ化を行う際は、以下のような**役割単位での分離**が基本です。

```text
yourlib/
├── __init__.py
├── core/
│   ├── __init__.py
│   ├── logic.py
│   └── validator.py
├── util/
│   ├── __init__.py
│   └── io.py
├── config/
│   ├── __init__.py
│   └── settings.py
└── cli/
    ├── __init__.py
    └── main.py
```

このように、「業務処理」「補助機能」「設定」「CLI」などに明確に分けることで、**変更の局所化**と**import依存の制御**が容易になります。

---

## ✅ 15.3 共通機能を `util/` に切り出す理由

`util/` パッケージは、以下のような**汎用機能や共通処理**を格納するための場所です。

* ファイル入出力（JSON, CSV, YAML など）
* ログ設定・パス操作・日付変換
* 汎用的な正規化・変換関数

これにより、業務ロジックと共通機能を分離でき、**再利用性とテスト容易性**が向上します。

---

## ✅ 15.4 依存方向の明示と階層化の原則

理想的な構造では、**下位層（util, config）が上位層（core, cli）に依存しない**ように設計します。
これは**疎結合な構造を保ち、循環importを防止するための基本原則**です。

```text
NG例：
core/logic.py → util/io.py → core/logic.py
```

```text
OK例（依存方向が一方向）：
util/ → core/ → cli/
```

---

## ✅ 15.5 `__init__.py` によるインターフェース整理

サブパッケージの `__init__.py` にimportを集約することで、使用者にとっての**明示的な公開API**を定義できます。

```python
# util/__init__.py
from .io import read_json, write_json
```

これにより、使用側は以下のように簡潔に記述できます：

```python
from yourlib.util import read_json
```

---

## ✅ 15.6 importパスの安定化と構造設計

importは絶対パスに統一し、構造変更に強い形にするべきです。

```python
# 悪い例（依存が曖昧で壊れやすい）
from ..core import logic

# 良い例（構造が明示されている）
from yourlib.core import logic
```

また、テストやスクリプト実行時に `sys.path` を追加しないで済むよう、プロジェクト構成を `src/` 配下に統一するのが望ましいです（import12参照）。

---

## ✅ 15.7 まとめ：再利用性を担保する設計原則

* 機能や責務ごとにサブパッケージを分割し、階層構造を明確にする
* 共通処理は `util/` に隔離し、上位層への依存を禁止する
* `__init__.py` を使ってパッケージの公開APIを定義する
* importは絶対パスに統一し、構造変更に強い設計とする
* テスト・CLI・バッチ等の実行系からも再利用可能な構造を最初から想定する

---




# 📘 import16 - 副作用とモジュール初期化

（処理実行タイミングとその制御の設計指針）

---

## ✅ 16.1 Pythonにおけるimportの本質

Pythonでは、`import` は単なる名前解決ではなく、**モジュールの読み込みと初期化を同時に行う**命令です。
つまり、importされた瞬間に、そのファイル内のトップレベル（インデントなし）のコードがすべて実行されます。

```python
# yourlib/util/logger.py
print("Loggerが初期化されました")
```

```python
# 別モジュール
from yourlib.util import logger  # ← ここで print が実行される
```

このような挙動は**副作用**と呼ばれ、意図しないタイミングでコードが動作する原因となります。

---

## ✅ 16.2 モジュールの読み込みとキャッシュ

Pythonでは、同じモジュールを複数回importしても、2回目以降は**初期化処理を再実行しません**。
最初のimport時に一度だけ初期化され、その結果は `sys.modules` にキャッシュされます。

```python
import sys
print(sys.modules["yourlib.util.logger"])
```

これはパフォーマンス上有利ですが、**副作用が起きる場所・順序が不明確になる**原因でもあります。

---

## ✅ 16.3 import時の副作用が問題となる場面

| ケース                        | 影響                    |
| -------------------------- | --------------------- |
| ログ設定や設定ファイルの読み込みがトップレベルにある | importだけで設定が走ってしまう    |
| テストで一部のモジュールだけ使いたい         | import時に全体が実行される      |
| 多数のCLI・スクリプトで共有されるモジュール    | スクリプトの一部だけ使いたいが副作用が発生 |

---

## ✅ 16.4 副作用を抑制する設計方法

### ✔ 方法1：トップレベルには定数・関数定義のみを書く

```python
# 悪い例
print("設定ファイルを読み込みました")  # import時に実行される

# 良い例
def setup():
    print("設定ファイルを読み込みました")
```

---

### ✔ 方法2：初期化処理は関数内または main 節に移す

```python
def init_logger():
    print("Loggerを初期化しました")

if __name__ == "__main__":
    init_logger()
```

---

### ✔ 方法3：`main()` を持たせて明示的に起動させる

```python
def main():
    run()

if __name__ == "__main__":
    main()
```

このパターンは **importされたときには動作せず、スクリプトとして直接実行されたときのみ動作**するため、安全です。

---

## ✅ 16.5 テスト環境での副作用とimport順依存の問題

pytest等のテスト環境では、import順序によって設定や状態が変化する場合があります。

```python
# test_a.py で logger が初期化される
# test_b.py は logger が既に初期化された状態で始まる
```

このような状態共有があると、**テスト間に依存関係が発生し、再現性が失われる**可能性があります。

### 解決策：

* 初期化状態を毎回明示的に設定・破棄する
* import時には実行されない構造に保つ（設定は関数化）
* pytestの fixture や setup 関数で状態を管理する

---

## ✅ 16.6 副作用の有無を確認するチェックリスト

| チェック項目                             | 対応               |
| ---------------------------------- | ---------------- |
| トップレベルで関数外に処理（printやopenなど）がある     | ✕ 構造を見直すこと       |
| モジュールimportだけでデータベース接続が始まる         | ✕ 初期化関数に隔離すべき    |
| import先モジュールが`__main__`の前提で設計されている | ✕ モジュール化・再利用性がない |

---

## ✅ まとめ

* Pythonのimportは**初期化を伴う**ため、importされた瞬間に副作用が発生することがある
* モジュールは**定義だけを含むもの**とし、実行は関数または `__main__` に移すべきである
* テスト・再利用・分割実行などを考慮すると、**副作用のある構造は重大な設計リスク**となる
* import時の副作用は、静的import構造の利点を失わせるため、**可能な限り排除・制御**することが望ましい

---

# 📘 import17 - `__main__` の真の意味と運用

---

## ✅ 17.1 `__main__` とは何か

Pythonファイルを実行する際、実行されたファイルの `__name__` は `"__main__"` になる。
この挙動を利用して、**モジュールとしても使えるし、スクリプトとしても使える**コードを書くことが可能になる。

```python
# example.py
def run():
    print("処理実行")

if __name__ == "__main__":
    run()
```

* `python example.py` → `"__main__"` なので `run()` が実行される
* `import example` → `__name__ == "example"` なので `run()` は実行されない

---

## ✅ 17.2 `__main__` の意図と設計

この仕組みにより、以下の設計方針が可能になる：

| 用途                    | 実現方法                                |
| --------------------- | ----------------------------------- |
| モジュールとして他のファイルから関数を利用 | `def` 群を外部からインポート                   |
| スクリプトとして単体実行での動作確認    | `if __name__ == "__main__"` ブロックで確認 |

これは **テスト性・再利用性・保守性の3点で重要**。スクリプト起動時だけ副作用を発生させるべき、という設計原則を体現している。

---

## ✅ 17.3 CLIやClickとの接点

この構造は `click` を用いるCLIでも前提とされる。
`click` のエントリポイントは `main()` のような関数に対し、`if __name__ == "__main__"` で起動するのが一般的。

```python
import click

@click.command()
def main():
    click.echo("Hello CLI")

if __name__ == "__main__":
    main()
```

* CLIツールのエントリポイントは `main()` に集約されており、importしても副作用が起きないようにする。

---

## ✅ 17.4 `__main__.py` の役割（補足）

パッケージ単位で `python -m パッケージ名` と実行したとき、`__main__.py` が存在すればその中身が実行される。

```plaintext
mytool/
├── __init__.py
├── __main__.py
└── core.py
```

```bash
python -m mytool  # __main__.py が実行される
```

これは `click` や `argparse` を含んだCLIの配布などで、`pip install` 後に `python -m` で使わせたい場合に必要になる。

---

## ✅ 17.5 実務における注意点

* **副作用のあるコード**（ファイル書き込み、API実行など）は、必ず `__main__` ブロック内に隔離せよ。
* 逆に、**グローバルスコープで `print()` や `open()` を行う**のは、意図しない副作用を生む。
* `__name__` の挙動は `pytest` にも関係する。import時に処理が走ると、テストが壊れる。

---




# 📘 import18 - `__init__.py` の設計とパッケージ初期化の制御

---

## ✅ 18.1 `__init__.py` の存在意義

### ◉ パッケージ識別の役割

* Pythonは `__init__.py` を含むディレクトリを「**パッケージ**」とみなす。
* Python 3.3以降、**namespace package**（`__init__.py` 不要）が導入されたが、
  実務ではいまだに明示的に `__init__.py` を設けるのが一般的。
* **ディレクトリが意図されたPythonパッケージであることを明示**する目的がある。

### ◉ 実行タイミング

* `import ディレクトリ名` で初めて、そのディレクトリ内の `__init__.py` が実行される。
* これは **1プロセス内で一度限り**。
* import時に副作用（ファイル書込・HTTP通信など）を起こすと問題になる。

---

## ✅ 18.2 典型的なパターン

以下のように、パッケージ内で構造が深くても、`__init__.py` で**トップレベルAPIのエクスポート**を制御できる。

```plaintext
mypkg/
├── __init__.py
├── core/
│   ├── __init__.py
│   └── run.py
├── utils/
│   └── helper.py
```

### `core/run.py`

```python
def run_main():
    print("Running main")
```

### `__init__.py`（mypkg直下）

```python
from .core.run import run_main
```

→ 使用者側：

```python
from mypkg import run_main
```

このように **深いディレクトリ構造をユーザーに意識させず、浅いインポートで提供**するのが実務設計の基本。

---

## ✅ 18.3 **all** による公開範囲の制御

```python
# mypkg/__init__.py
from .core.run import run_main
from .utils.helper import some_helper

__all__ = ["run_main", "some_helper"]
```

* `from mypkg import *` のとき、`__all__` に含まれるものだけがインポートされる。
* 「**何を公開し、何を内部に留めるか**」を明示的に区切るための記法。

---

## ✅ 18.4 初期化コードはどう使うべきか

### 使用してよいもの

* ログ設定
* バージョン情報の定義
* サブモジュールのimport整理
* 定数の読み込み

### 使用すべきでないもの

* ファイルI/O（設定ファイル読込は例外的に可）
* ネットワーク通信
* クラスインスタンス生成
* 実行系のコード（`main()` など）

**副作用を生む処理は厳禁**。理由は明確で、「import しただけで何かが動く」のは再利用性・テスト性・予測性の全てを壊す。

---

## ✅ 18.5 階層パッケージと再エクスポート戦略

```plaintext
project/
├── service/
│   ├── __init__.py         ← APIの集約ポイント
│   └── core/
│       ├── __init__.py
│       └── main.py         ← 実装
```

* `service/core/main.py` に実装があり、
* `service/__init__.py` で `from .core.main import run` のように再エクスポート

使用者にとっては：

```python
from project.service import run  # 深さを意識させない
```

これは **public API と internal 実装の境界を作る**実装戦略であり、大規模コードベースで重要。

---

## ✅ 18.6 なぜ `__init__.py` をあえて空にするのか

* 再エクスポートをしない小規模構成では、`__init__.py` を空に保つことで**何も副作用を起こさないことを明示**できる。
* セキュリティ的にも、「意図しない実行コードが import により走らない」保証になる。

---

## ✅ 18.7 pytestとの関係

* pytestはテスト実行時に各モジュールをimportする。
* `__init__.py` に実行コードがあると、テスト前に意図しない副作用が発生する可能性がある。

**importは即時実行ではない**、という原則が破れるため、pytestを使うプロジェクトではとくに `__init__.py` の副作用管理が重要。

---

## ✅ 18.8 実務ルールまとめ

| ルール                 | 意図・理由             |
| ------------------- | ----------------- |
| `__init__.py` は常に設置 | パッケージとしての構成保証と明示性 |
| 副作用を起こす処理は禁止        | 再利用性・テスト性・安全性の担保  |
| 公開APIのみを再エクスポート     | 利用者の視点からのAPI設計    |
| `__all__` による公開制御   | 明示的なインターフェース管理    |
| importの深さと構造を切り離す   | 利用者には浅く、実装は深く構造化  |

---




# 📘 import19 - 複数プロジェクト間のユーティリティ共有

---

## ✅ 19.1 問題設定と動機

複数のプロジェクトに共通する処理（例：ロガー設定、ファイルIO、定数など）を1か所にまとめ、
各プロジェクトからimportして使いたい、という需要は現場で非常に多い。

例：

```plaintext
common-utils/
├── __init__.py
├── logger.py
└── io.py

project-a/
├── main.py
└── ...
```

### 典型的な誤解：

* `sys.path.append("../common-utils")` でなんとかなる、と思い込む
* Pythonのimportは**フォルダ構造さえ見えていれば動く**と思っている
* リンカやパス探索の仕組みを正確に理解していない

---

## ✅ 19.2 Pythonにおけるimportの検索順序（sys.path）

Pythonは `import` の際、以下の順序で検索を行う：

1. スクリプトのあるディレクトリ（カレントディレクトリ）
2. `PYTHONPATH` 環境変数で指定されたパス
3. 仮想環境の `site-packages` 等
4. `sys.path` に含まれるディレクトリ

したがって、**プロジェクト外のフォルダをimport対象とするには、明示的なsys.path操作またはPYTHONPATHの設定が必要**。

---

## ✅ 19.3 安定した共有のための3つの選択肢

### ✅ 選択肢①：pip install -e によるローカル開発インストール

```bash
cd common-utils
pip install -e .
```

* `setup.py` または `pyproject.toml` を用意し、editableインストール
* `project-a` 側からは通常のライブラリのように import可能

```python
from common_utils.logger import get_logger
```

→ **現実的で最も安全。CI/CDでも動作保証しやすい。**

---

### ✅ 選択肢②：パス操作による暫定的共有（非推奨）

```python
import sys
from pathlib import Path

sys.path.append(str(Path(__file__).resolve().parent.parent / "common-utils"))

from logger import get_logger
```

* 明示的にsys.pathに追加
* 実行時環境に依存しやすく、**保守性・再現性・可搬性が低下**
* pytest等で失敗しやすく、VSCode補完が効かないこともある

---

### ✅ 選択肢③：共通ユーティリティを各プロジェクトにコピー

* ソース管理で `utils/` フォルダを丸ごと複製
* importの問題は起きないが、**共通機能のバージョンがプロジェクト間でズレていく**という別の問題が生じる

---

## ✅ 19.4 仮想環境とPYTHONPATHの使い方

`.env` ファイルや `launch.json` に以下を定義することで、開発中に `common-utils` をimport可能にする手段もある：

```bash
PYTHONPATH=../common-utils
```

これは一時的には有効だが、**環境差異（OS・IDE・CI）でのトラブルを招きやすい**。

---

## ✅ 19.5 統一された構成案（モノレポ型）

```plaintext
root/
├── common_utils/        ← 再利用コード
│   ├── __init__.py
│   └── logger.py
├── project_a/
│   ├── main.py
├── project_b/
│   └── main.py
└── pyproject.toml       ← 全体統一で管理
```

この場合、共通部分を`pip install -e ./common_utils`で仮想環境に組み込む。

---

## ✅ 19.6 実務的な判断基準

| 条件                | 推奨方法                   |
| ----------------- | ---------------------- |
| 複数人チーム・CI/CD前提    | editable install（選択肢①） |
| 単独開発・一時的な検証       | sys.path追加（選択肢②）       |
| importの混乱を絶対に避けたい | utilsを直接埋め込み（選択肢③）     |
| monorepo構成で統一管理   | pyproject.tomlで全体統一    |

---

## ✅ 19.7 よくある失敗例と対策

| 失敗例                   | 原因                              | 対策                        |
| --------------------- | ------------------------------- | ------------------------- |
| `ModuleNotFoundError` | sys.pathにパスが通っていない              | `pip install -e`で対応       |
| テスト環境でimport失敗        | pytestがカレントディレクトリ依存で動いている       | ルートをプロジェクト直下に固定           |
| VSCodeで補完が効かない        | PYTHONPATHやsys.pathがIDEに伝わっていない | `.env`や`settings.json`で補正 |

---

## ✅ 19.8 実務設計のまとめ

* **1プロジェクト1仮想環境、1つの共通パッケージ**を原則とする
* importエラーが起きる設計は、**技術的には動いても実務では不適**
* 再利用と保守性を両立させるには、**editable install型が現実的解**

---




# 📘 import20 - 循環インポートの発生原因と回避策

---

## ✅ 20.1 循環インポートとは何か

**2つ以上のモジュールが、互いに依存し合ってimportしようとする構造**のこと。

```plaintext
a.py  →  b.py  →  a.py
↑        ↑        ↑
1回目    2回目   失敗（未定義）
```

Pythonは1回目のimportを途中まで進めるが、再び未完成のモジュールをimportしようとした時に、
**定義が完了しておらず失敗する（AttributeError, ImportErrorなど）**。

---

## ✅ 20.2 典型的な失敗例

### ファイル構成：

```plaintext
project/
├── models/
│   ├── user.py
│   └── post.py
```

### `user.py`

```python
from models.post import Post  # ← post.pyをimport

class User:
    def __init__(self, posts: list[Post]):
        self.posts = posts
```

### `post.py`

```python
from models.user import User  # ← user.pyをimport

class Post:
    def __init__(self, author: User):
        self.author = author
```

### 実行結果：

```bash
ImportError: cannot import name 'User' from 'models.user'
```

---

## ✅ 20.3 循環が発生する原因

| 原因               | 詳細                              |
| ---------------- | ------------------------------- |
| **相互参照**         | モデルや関数の定義に互いのクラスが必要になる          |
| **トップレベルimport** | 関数外・クラス外で直にimportすると、評価順で未定義となる |
| **広すぎる責務**       | モジュールが複数の概念を処理しており分割不十分         |

---

## ✅ 20.4 回避策①：**遅延インポート（関数内import）**

```python
def create_user(posts):
    from models.post import Post  # 関数内に移動
    return User(posts)
```

* Pythonは**import文の実行をその場所で行う**
* 関数内に置けば、**呼び出されるまでimportされない**

ただし、**関数が頻繁に呼ばれるとオーバーヘッドになる**ので注意。

---

## ✅ 20.5 回避策②：**型ヒントの forward reference（文字列化）**

```python
# post.py
from __future__ import annotations  # Python 3.7〜
from models.user import User

class Post:
    def __init__(self, author: "User"):  # ← 文字列にする
        self.author = author
```

* 型ヒントに限っては、**文字列として遅延評価される**
* `from __future__ import annotations` によりすべての型ヒントを遅延評価できる（Python 3.10以降はデフォルト可）

---

## ✅ 20.6 回避策③：**依存関係の再設計（責務分離）**

循環が発生する場合、モジュール設計が不適切な可能性がある。

```plaintext
models/
├── user.py
├── post.py
└── base.py  ← 共通の型や関係性のみを定義
```

* 共有される抽象型（例：`Author`, `Postable`）だけを `base.py` に切り出し、
* 他のモジュールはこれに依存することで循環を断ち切る

これは**ドメイン分離**の基本でもある。

---

## ✅ 20.7 回避策④：**局所的なimportに切り替え**

* 型ヒントのみが必要なら `from typing import TYPE_CHECKING` を使い、実行時はimportを省く

```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from models.user import User
```

これは `mypy` や `pyright` など型チェッカにのみ効き、実行時には無害となる。

---

## ✅ 20.8 実務設計の指針

| 原則                         | 意図                            |
| -------------------------- | ----------------------------- |
| モジュールは単方向に依存させる            | importの木構造がループしないように設計する      |
| 型ヒントの循環には forward ref を用いる | 実行エラーを回避できる                   |
| 中央に共通モジュールを置く              | 関係が発生するクラスは中心に統合（例：`base.py`） |
| importはトップレベル以外でも許容される     | 実務では柔軟に関数内・条件内importを使う       |

---

## ✅ 20.9 判別と検知

* 循環参照は**静的解析（例：pylint, pyflakes）では検知できないこともある**
* 実行時 `ImportError`, `AttributeError`, `NoneType has no attribute` などで気づくことが多い

---

## ✅ 20.10 総まとめ：避けるべき構造と対応指針

| 悪い構造                          | 修正例                                   |
| ----------------------------- | ------------------------------------- |
| `user.py ←→ post.py` 相互import | 共通部分を `base.py` に分離                   |
| トップレベルの型ヒント参照                 | forward ref で文字列化 or TYPE\_CHECKING使用 |
| 実行時にも関係するimport               | 関数内に移動して遅延import                      |

---



# 📘 import21 - pipでのプロジェクトインストール

---

## ✅ 21.1 問題設定

Pythonプロジェクトを他人が `pip install` で簡単にインストール・再利用できるようにするには、
**パッケージ化**と**メタデータの整備**が必要。

ただし以下を前提とする：

* `pyproject.toml` を使わずに `setup.py` のみで構成（レガシー互換性重視）
* `pip install -e .` によるローカル開発利用が主目的
* 再利用されるパッケージ名（例：`poke-pipeline` や `my-utils`）を意識した設計

---

## ✅ 21.2 最小構成例

```plaintext
myproject/
├── mylib/                ← import単位
│   ├── __init__.py
│   └── core.py
├── setup.py              ← メタ情報（インストールの核）
└── README.md             ← 任意だが推奨
```

### `mylib/__init__.py`

```python
from .core import hello
```

### `mylib/core.py`

```python
def hello():
    print("Hello from core!")
```

---

## ✅ 21.3 `setup.py` の最小例

```python
from setuptools import setup, find_packages

setup(
    name="mylib",
    version="0.1.0",
    packages=find_packages(),  # mylib/ を自動検出
    install_requires=[],       # 依存パッケージがあれば記載
    author="Your Name",
    description="My minimal reusable library",
)
```

* `name`: `pip install` 時のパッケージ名（他人と被るとPyPIで衝突）
* `packages=find_packages()` により、`__init__.py` を含むディレクトリを探索
* `install_requires`: 空でも構わない（依存がないなら）

---

## ✅ 21.4 開発者向けインストール（-e）

```bash
cd myproject
pip install -e .
```

これにより、

```python
from mylib import hello
```

が他のプロジェクトから使用可能になる。

### 実行確認用の `main.py` を外部に用意しておくとよい：

```python
# test_runner.py（ルート直下など）
from mylib import hello

hello()
```

---

## ✅ 21.5 setuptoolsの基本原理

* `pip install .` 実行時、`setup.py` が読み込まれ、パッケージが `site-packages` にコピーされる
* `pip install -e .` 実行時、**シンボリックリンク**が作られ、ソースを直接反映可能に
* `find_packages()` は `__init__.py` の有無でパッケージか否かを判定

---

## ✅ 21.6 よくあるミスと対策

| 症状                                             | 原因                         | 対策                           |
| ---------------------------------------------- | -------------------------- | ---------------------------- |
| `ModuleNotFoundError: No module named 'mylib'` | setup.pyの `packages` 設定が不正 | `find_packages()` を使う        |
| `pip install -e .` 後に反映されない                    | キャッシュ or リンクが切れている         | 一度 `pip uninstall` → 再度 `-e` |
| `setup.py not found`                           | 作業ディレクトリが誤っている             | `cd myproject` を確認           |

---

## ✅ 21.7 実務における拡張例

* テスト用には `tests/` ディレクトリを別途用意し、`setup.py` では除外
* CLI用途がある場合は `entry_points` に `console_scripts` を追加
* 長期的には `pyproject.toml` への移行が望ましいが、レガシー環境では `setup.py` が依然として主流

---

## ✅ 21.8 確実な設計のための原則

| 原則                                   | 理由                          |
| ------------------------------------ | --------------------------- |
| importされるモジュールは1つのディレクトリに集約          | `find_packages()` が期待通り動作する |
| `__init__.py` をすべての階層に含める            | パッケージと認識される条件               |
| 依存は `install_requires` で明示           | 他環境での動作保証                   |
| pipインストール不要な開発環境では `sys.path` 操作は避ける | 本番環境と乖離するため                 |

---


# 📘 import22 - `pyproject.toml` によるCLI

---

## ✅ 22.1 問題設定

現代的なPythonプロジェクトでは、`pyproject.toml` を使って
`setuptools` ベースで **CLI（コマンドラインツール）** を定義できる。

以下の目的を前提とする：

* ユーザーが `pip install .` 後に `mycli` のようなコマンドを直接使えるようにする
* 従来の `setup.py` ではなく、`pyproject.toml` を中心に構成する
* `setuptools` の公式仕様に準拠しつつ、最低限の例から実用構成までカバー

---

## ✅ 22.2 プロジェクト構成例

```plaintext
myproject/
├── src/
│   └── mylib/
│       ├── __init__.py
│       └── cli.py         ← main関数をここに定義
├── pyproject.toml         ← メタ情報 + CLI定義
└── README.md
```

---

## ✅ 22.3 `cli.py` の例

```python
# src/mylib/cli.py

def main():
    print("Hello from CLI tool!")

if __name__ == "__main__":
    main()
```

---

## ✅ 22.4 `pyproject.toml` の定義（最小）

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "mylib"
version = "0.1.0"
description = "A sample CLI tool"
authors = [
  { name="Your Name", email="your@email.com" }
]
dependencies = []

[project.scripts]
mycli = "mylib.cli:main"
```

ポイント：

* `[project.scripts]` により、`pip install .` 後に `mycli` が使える
* `"mylib.cli:main"` は `src/mylib/cli.py` 内の `main()` を指す
* `src/` に置く場合、`setuptools` 側の設定が必要（次項）

---

## ✅ 22.5 `setuptools` 側の設定（src構成対応）

```toml
[tool.setuptools]
package-dir = {"" = "src"}

[tool.setuptools.packages.find]
where = ["src"]
```

これにより `src/` 以下の `mylib` パッケージが正しく検出される。

---

## ✅ 22.6 インストールと動作確認

```bash
cd myproject
pip install .
mycli
# → Hello from CLI tool!
```

※ 開発用には `pip install -e .` を使えば即時反映される。

---

## ✅ 22.7 よくあるエラーと原因

| 症状                    | 原因                    | 対策                              |
| --------------------- | --------------------- | ------------------------------- |
| `command not found`   | `pip install .` していない | インストール後に再確認                     |
| `ModuleNotFoundError` | import指定が誤っている        | `"mylib.cli:main"` の構文を確認       |
| CLIが反映されない            | 古いバージョンが残っている         | `pip uninstall mylib` → 再インストール |

---

## ✅ 22.8 設計の原則と補足

| 原則                           | 理由                        |
| ---------------------------- | ------------------------- |
| CLIエントリポイントは `main()` 関数に統一  | テスト・再利用が容易                |
| `src/` 構成＋ `package-dir` を使う | importの明示化と分離             |
| `project.scripts` を使う        | setup.py非依存の構成（PEP621）に準拠 |

---


# 📘 import23 - 引数付きCLI設計
---

## ✅ 23.1 問題設定

CLIツールを設計する上で、次の2つは本質的に異なる：

| 要求                | 内容例                          |
| ----------------- | ---------------------------- |
| 実行可能性（コマンド化）      | `mycli` のように直接叩ける            |
| 引数受取とパース（インタフェース） | `mycli fetch --limit 100` など |

この章では、**引数を伴うCLI設計**を次の方針で扱う：

* **`pyproject.toml` + setuptools + entry\_points**
* **標準ライブラリ `argparse`** を基本とし、後半で **`click`** を導入
* シンプルな例から、コマンド分岐・ヘルプ生成まで扱う

---

## ✅ 23.2 プロジェクト構成

```plaintext
myproject/
├── src/
│   └── mylib/
│       ├── __init__.py
│       └── cli.py
├── pyproject.toml
└── README.md
```

---

## ✅ 23.3 `argparse` による基本構造

```python
# src/mylib/cli.py

import argparse

def main():
    parser = argparse.ArgumentParser(description="データ取得ツール")
    parser.add_argument("--limit", type=int, default=10, help="取得件数")
    parser.add_argument("--verbose", action="store_true", help="詳細出力")
    args = parser.parse_args()

    if args.verbose:
        print(f"[DEBUG] limit = {args.limit}")
    print(f"Fetching {args.limit} items...")
```

---

## ✅ 23.4 `pyproject.toml` の記述（再掲）

```toml
[project]
name = "mylib"
version = "0.1.0"
# ...

[project.scripts]
mycli = "mylib.cli:main"

[tool.setuptools]
package-dir = {"" = "src"}

[tool.setuptools.packages.find]
where = ["src"]
```

---

## ✅ 23.5 実行例

```bash
pip install -e .
mycli --limit 20 --verbose
```

出力：

```
[DEBUG] limit = 20
Fetching 20 items...
```

---

## ✅ 23.6 サブコマンド構成（`argparse`）

```python
def main():
    parser = argparse.ArgumentParser(prog="mycli")
    subparsers = parser.add_subparsers(dest="command")

    fetch_parser = subparsers.add_parser("fetch")
    fetch_parser.add_argument("--limit", type=int, default=5)

    args = parser.parse_args()

    if args.command == "fetch":
        print(f"Fetching {args.limit} items...")
    else:
        parser.print_help()
```

---

## ✅ 23.7 `click` による代替案

```python
import click

@click.group()
def main():
    pass

@main.command()
@click.option("--limit", default=10, help="取得件数")
@click.option("--verbose", is_flag=True, help="詳細出力")
def fetch(limit, verbose):
    if verbose:
        click.echo(f"[DEBUG] limit = {limit}")
    click.echo(f"Fetching {limit} items...")

if __name__ == "__main__":
    main()
```

### 補足：

* `click.group()` → サブコマンドを束ねる
* `@main.command()` → `mycli fetch` に対応

---

## ✅ 23.8 設計観点での比較表

| 観点        | argparse       | click            |
| --------- | -------------- | ---------------- |
| 標準ライブラリ   | ✅（追加依存なし）      | ❌（インストール必要）      |
| ヘルプ生成の簡便性 | △（構造が煩雑になりやすい） | ◎（デコレータで簡潔）      |
| サブコマンド設計  | ◯（明示的だが記述量が多い） | ◎（グループ設計が容易）     |
| 型変換/検証    | △（自前で定義）       | ◎（自動型変換＋バリデーション） |
| 教育用途・互換性  | ◎（標準のみで完結）     | △（導入コストと依存がある）   |

---

## ✅ 23.9 設計原則

| 原則                           | 意図                            |
| ---------------------------- | ----------------------------- |
| 引数の構造は **関数シグネチャに近づける**      | 関数単位での再利用やテストを容易にする           |
| CLIはエントリポイントと処理ロジックを分離せよ     | ユニットテスト・UI移植への耐性が高くなる         |
| clickの導入は**依存と教育コストを許容できる時** | 複雑なCLIや補助ツール群では強力だが、軽量環境には不向き |

---

次章（import24）では、
**複数のCLIツールをまとめて提供する構成**（例：`project.scripts` に複数コマンドを登録）
および `src/cli/` のように CLI 群を分離管理する設計に進む。


# 📘 import24 - 複数CLIツールの提供と構成管理

---

## ✅ 24.1 問題設定

プロジェクトによっては、**複数のCLIコマンドを個別に提供**したい場面がある。

例：

* `poke-fetch`: APIからのデータ取得
* `poke-clean`: JSON整形
* `poke-serve`: HTTPサーバ起動

これらをユーザーに **コマンド名単位で使わせる**には、次の設計が必要：

* `pyproject.toml` の `project.scripts` に複数のコマンドを登録
* CLI用の各モジュールを構造的に分離・管理

---

## ✅ 24.2 ディレクトリ構成（src配下にcli/ディレクトリ）

```plaintext
myproject/
├── src/
│   └── pokecli/
│       ├── __init__.py
│       ├── fetch.py       ← poke-fetch の本体
│       ├── clean.py       ← poke-clean の本体
│       └── serve.py       ← poke-serve の本体
├── pyproject.toml
└── README.md
```

---

## ✅ 24.3 各ファイルの例（簡略）

### `fetch.py`

```python
def main():
    print("Fetching data...")
```

### `clean.py`

```python
def main():
    print("Cleaning data...")
```

---

## ✅ 24.4 `pyproject.toml` の定義（複数CLI）

```toml
[project]
name = "pokecli"
version = "0.1.0"
# ...

[project.scripts]
poke-fetch = "pokecli.fetch:main"
poke-clean = "pokecli.clean:main"
poke-serve = "pokecli.serve:main"

[tool.setuptools]
package-dir = {"" = "src"}

[tool.setuptools.packages.find]
where = ["src"]
```

---

## ✅ 24.5 実行確認

```bash
pip install -e .
poke-fetch
# → Fetching data...

poke-clean
# → Cleaning data...
```

---

## ✅ 24.6 設計上の利点

| 設計                          | 意図                                  |
| --------------------------- | ----------------------------------- |
| CLI用のディレクトリを `pokecli/` に集約 | エントリポイントの見通しが良くなり、規模拡大に強い           |
| コマンド名とファイル名を一致              | `poke-fetch = fetch.py:main` の一貫性確保 |
| 複数のコマンドが単独で機能するよう設計         | `main()` 単位で設計し、テスト・再利用を可能にする       |

---

## ✅ 24.7 CLI設計の原則（複数エントリ向け）

| 原則                            | 説明                              |
| ----------------------------- | ------------------------------- |
| 1 CLI = 1 ファイル + 1 `main()`関数 | 構成の一貫性を保ち、疎結合を促す                |
| コマンド名 ≒ モジュール名 ≒ スクリプト名       | ユーザーと開発者双方の混乱を防ぐ                |
| `src/cli/` を使い、UI層として分離       | CLIはUIの一種であり、業務処理と混在させない設計が望ましい |

---

## ✅ 24.8 将来的な拡張と選択肢

* `click.group()` による **単一CLI内サブコマンド方式**（`poke-all` で統合）
* GUIやWebに移行したい場合も、`main()` が分離されていれば容易に再利用可
* CLI専用ライブラリを切り出す場合は、`pokecli-core` として別パッケージ化可能

---




# 📘 import25 - CLIにおける設定ファイル

---

## 🔰 なぜ設定ファイルが必要なのか？

PythonでCLI（コマンドラインツール）を作るとき、最初は `--limit 10` のように
引数でパラメータを指定するだけで済む場合が多いです。

しかし、実際の開発や運用が進んでくると、次のような課題が出てきます：

* 毎回長い引数を入力するのが面倒
* 環境ごと（例：本番用・開発用）に設定を変えたい
* 実行時のパラメータをバージョン管理（Git管理）したい
* 引数の組み合わせを人に共有したい

このようなときに、**設定ファイル（configファイル）を使って外から情報を与える**という方法が重要になります。CLIの「柔軟性」と「再現性」を担保するための基本的な設計方針です。

---

## 🧱 どんな形式の設定ファイルがあるのか？

Pythonでは主に3つの形式が使われます：

| 形式      | 特徴と使いどころ                                          |
| ------- | ------------------------------------------------- |
| `.json` | 最も簡単で高速。標準ライブラリで読める。ただしコメントが書けない。                 |
| `.toml` | 最近主流。`pyproject.toml` でも使われており、構造が分かりやすい。型情報も扱える。 |
| `.yaml` | 柔軟で多機能。ただし読み込みには外部ライブラリ（PyYAMLなど）が必要で、仕様が複雑。      |

現在の主流は `.toml` です。Python 3.11以降では `tomllib` というモジュールが標準で使えるようになりました。`.json` は依然として軽くて手軽ですが、長期的な運用では `.toml` を採用する方が安全です。

---

## 🧪 実際にどうやって設定ファイルを読むのか？

では、設定ファイルを読み込むCLIツールの例を考えてみます。
まずは以下のような `config.json` を用意します：

```json
{
  "limit": 30,
  "verbose": true
}
```

次に、これを読み込むPythonコードです：

```python
import json
from pathlib import Path
import argparse

def load_config(path: Path) -> dict:
    with path.open(encoding="utf-8") as f:
        return json.load(f)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--config", type=str, default="config/config.json")
    args = parser.parse_args()

    config = load_config(Path(args.config))
    print(f"読み込んだ設定: {config}")
    print(f"{config.get('limit')} 件のデータを取得します")
```

このプログラムを `poke-fetch` コマンドとして登録しておけば、

```bash
poke-fetch --config config/config.json
```

のように実行できます。ここで使っているのは `.json` ですが、`.toml` を使いたい場合は `tomllib` に置き換えるだけでOKです。

---

## 🔄 設定ファイルと引数の関係は？

CLIツールでは、「設定ファイルに書かれた値」と「引数で与えられた値」が**衝突**することがあります。
たとえば、

```bash
poke-fetch --limit 10 --config config.json
```

のように指定された場合、`config.json` に `limit = 30` が書かれていても、
引数で指定した `--limit 10` を優先させるのが普通です。

このような**優先順位の原則**があります：

```
引数 > 環境変数 > 設定ファイル > デフォルト値
```

実装としてはこう書きます：

```python
limit = args.limit if args.limit is not None else config.get("limit", 10)
```

---

## 🏗 設定ファイルの管理設計について

設定ファイルの運用には注意点もあります。

* **1つのファイルに全設定を詰め込まないこと**（スコープが混在して管理不能になる）
* **環境別に分けること**（例：`config/dev.toml`, `config/prod.toml`）
* **CLIツール側では「どのファイルを使うか」を引数で切り替えること**

さらに進んだ設計では、以下のように**2つの設定ファイルを読み込んでマージ**することもあります：

```python
base_config = load_toml("config/default.toml")
local_config = load_toml("config/local.toml")
final_config = {**base_config, **local_config}  # local優先で上書き
```

このようにすれば、**共通の設定**（default）と**環境ごとの差分**（local）を分けて管理できます。

---

## ✅ 最後に

設定ファイルを使ったCLIの設計は、実際の開発現場ではほぼ必須の知識です。
CLIが単純なうちは不要でも、規模が拡大したり他人と共有する必要が出てくると、
引数だけでは管理できなくなります。

設定ファイルの導入により、以下が実現されます：

* 再現性のある動作（＝同じファイルを読み込めば同じ出力になる）
* 引数の簡略化（何度も同じ引数を打つ必要がなくなる）
* チーム共有（設定ファイルをGitで共有できる）

---



# 📘 import26 - `pyproject.toml`の統合管理とツール設定の整合性

---

## ✅ 26.1 背景と問題設定

`pyproject.toml` は元々ビルド設定用（PEP 518/PEP 621）に導入されましたが、
現在では以下のような **多数のツールが同ファイル内で設定を共有**するようになっています：

* ビルド: `setuptools`, `poetry`
* 静的解析: `mypy`, `ruff`, `flake8`
* テスト: `pytest`
* ドキュメント: `sphinx`
* フォーマッタ: `black`, `isort`

ここで問題となるのは：

* **設定が肥大化して可読性が落ちる**
* **ツール間で同じ情報を重複して記述してしまう**
* **各ツールの設定フォーマットが微妙に異なる**

よって、**一貫した設計と配置ルール**が重要となります。

---

## ✅ 26.2 実際の設定例（`pyproject.toml`）

```toml
[tool.black]
line-length = 88
target-version = ["py311"]

[tool.isort]
profile = "black"

[tool.ruff]
line-length = 88
select = ["E", "F"]
ignore = ["E501"]

[tool.mypy]
python_version = "3.11"
strict = true

[tool.pytest.ini_options]
addopts = "-ra"
testpaths = ["tests"]
```

それぞれ `[tool.X]` セクションを用いて、ツール別に分離されています。

---

## ✅ 26.3 なぜ `.toml` に統合するのか？

昔は `setup.cfg`, `.flake8`, `pytest.ini` などファイルが分かれていました。
今は `.toml` に統一することで次の利点があります：

* **設定場所が一箇所に集まる（属人性が減る）**
* **VSCode や CLI ツールの自動補完が効きやすい**
* **CI などでもファイルが減り、管理が単純化する**

---

## ✅ 26.4 整合性の取り方と設計原則

各ツールが使用する設定がバラバラになっていると、次のような矛盾が生じます：

* `black` の `line-length` が 88、`ruff` が 100 → チェックが通らない
* `mypy` の `strict` を `true` にしたが、`tests/` だけ除外したい → 除外設定が必要

こうした矛盾を避けるため、**以下のような共通原則を文書化しておくことが望ましい**：

```toml
# 共通ポリシー
[tool.shared]
line-length = 88
python-version = "3.11"
```

※ `tool.shared` はダミー。実際には `pyproject.toml` 内では共有できないが、
**構成設計上の合意事項**として、チームやREADMEに明記すべきという考え方。

---

## ✅ 26.5 分割管理は可能か？

原則として、`pyproject.toml` は **分割不可（単一ファイル）** です。
YAMLのように `include:` で外部ファイルを読み込む仕組みは存在しません。

### 対策1：一部のツールにだけ別ファイルを使う

* `mypy.ini`, `.flake8`, `pytest.ini` は個別に残すことも可能
* `pyproject.toml` との**片方のみ使用される**よう、明示的に整理

### 対策2：コメントとセクション分割を徹底

```toml
# ========================
# Static Analysis
# ========================
[tool.ruff]
...

# ========================
# Formatting
# ========================
[tool.black]
...
```

このように、**大項目ごとに水平線コメント**で区切ると読みやすくなります。

---

## ✅ 26.6 チェックリスト：整合性の確認

| チェックポイント                        | 内容                                     |
| ------------------------------- | -------------------------------------- |
| `line-length` はすべてのツールで一致しているか  | `black`, `ruff`, `isort` など            |
| `python_version` の記述が統一されているか   | `mypy`, `black`, `pytest` など           |
| テスト対象や除外対象のパスが揃っているか            | `pytest`, `mypy`, `ruff` の `exclude` 等 |
| `strict = true` 設定が開発段階で致命的でないか | `CIだけstrict` など段階的導入が現実的               |

---

## ✅ 26.7 CI/CDとの連携設計

設定が統一されていれば、CI上でも以下のようにシンプルなコマンドで済む：

```bash
black . --check
ruff check .
mypy src/
pytest
```

`pyproject.toml` に全ツールの設定がまとまっていれば、各ツールは**自動でそれを読み取る**ため、
CIでも明示的に設定ファイルを渡す必要はなくなる。

---

## ✅ まとめと設計原則

| 原則                                 | 意図                           |
| ---------------------------------- | ---------------------------- |
| すべてのツールの設定を `pyproject.toml` に集約する | 読みやすく、一元管理が可能になる             |
| 整合性の取れた設定値を選定する（line-length等）      | 複数のチェックが矛盾なく通る               |
| コメント・区切り・順序を明示する                   | 読み手が混乱しない（特に共同開発では必須）        |
| `pyproject.toml` をREADMEで補足説明する    | どのツールがどの設定を見ているかが一目で分かるようにする |

---




# 📘 import27 - `importlib` による動的import

---

## ✅ 27.1 背景と問題設定

通常の `import` は静的です。たとえば：

```python
from mymodule import run
```

これは「どのモジュール・関数を使うか」がコード中に明示されています。

しかし、次のような要件では**動的にimportする必要**が出てきます：

* CLIで受け取った文字列から処理モジュールを選びたい
* プラグイン構造を構築し、あとから拡張できるようにしたい
* ループ内でモジュール名を切り替えて呼び出したい

---

## ✅ 27.2 具体例：モジュール名を文字列で指定

たとえば CLI で次のような引数を渡されたとします：

```bash
mycli --task fetch
```

この `"fetch"` を使って `pokecli.fetch` モジュールを動的に読み込みたい。

```python
import importlib

def load_module(name: str):
    return importlib.import_module(f"pokecli.{name}")

mod = load_module("fetch")
mod.main()  # fetch.py に定義された main() を呼び出す
```

---

## ✅ 27.3 `importlib.import_module()` の使い方

```python
import importlib

mod = importlib.import_module("mylib.submodule")
# = from mylib import submodule と同等
```

* 戻り値は「モジュールオブジェクト」
* 属性（関数やクラス）は `mod.attr` でアクセスする

---

## ✅ 27.4 関数・クラス単体を直接importする

モジュールでなく、特定の関数だけを読み込みたい場合：

```python
def load_attr(module_path: str, attr: str):
    mod = importlib.import_module(module_path)
    return getattr(mod, attr)

func = load_attr("pokecli.fetch", "main")
func()
```

CLIや設定ファイルで `"pokecli.fetch:main"` のような形式を扱いたい場合に有効です。

---

## ✅ 27.5 応用：`entry_points` の代替手段

たとえば、設定ファイルに以下のような形式で処理を定義する構成にしたいとします：

```toml
[task]
fetch = "pokecli.fetch:main"
clean = "pokecli.clean:run"
```

これを読み込んで処理に反映する：

```python
def import_from_string(path: str):
    module_path, func_name = path.split(":")
    mod = importlib.import_module(module_path)
    return getattr(mod, func_name)

func = import_from_string("pokecli.fetch:main")
func()
```

これにより、**モジュールや関数の指定を完全に外部化**できます。
entry\_points を使わない柔軟な拡張方式として、非常に実用的です。

---

## ✅ 27.6 動的importの注意点と制約

| 課題              | 内容                                            |
| --------------- | --------------------------------------------- |
| 実行時までimportされない | 補完・型チェックが効かない（静的解析に不利）                        |
| 書き間違いに気づきにくい    | `"pokecli.fehtch:main"` → import error に気づけない |
| サンドボックス構成では制限あり | importlibを封じられる場合がある（安全性のため）                  |
| 本番環境ではテストが必須    | import失敗時の挙動を検知する試験コードが必須となる                  |

---

## ✅ 27.7 設計原則

| 原則                             | 説明                                       |
| ------------------------------ | ---------------------------------------- |
| importlib は「構造を固定せずに呼び出す」ために使う | 名前や構成が将来変わる可能性がある場合に有効                   |
| import文字列は「明示的に定義された場所」から読む    | configファイルやdictで事前定義されていること              |
| 失敗時には例外補足して代替処理を用意する           | `try: importlib… except:` でフォールバックを定義すべき |
| 本体処理とUI層（CLIなど）は分離しておく         | 動的にimportしても副作用がない設計（関数呼び出し型）にすべき        |

---

## ✅ まとめ

`importlib` を使うことで、Pythonは以下のような柔軟な処理が可能になります：

* モジュール名・関数名を外部から与える
* CLIや設定ファイルと連携して処理を動的に切り替える
* プラグイン構造を自作できる（軽量な拡張構造）

ただし、静的解析が難しくなるため、**単体テストや補完の設計も含めて管理する必要**があります。

---


# 📘 import28 - プラグイン設計と拡張可能構造

---

## ✅ 28.1 問題設定と背景

Pythonで他人が書いたモジュールや拡張機能を**自動的に発見して使う**にはどうすればよいか？

* LLM APIをラップするCLIに、**ユーザー定義のプロンプトやアクション**を外部モジュールとして追加したい
* データ処理系CLIで、**別途インストールされた解析モジュール群**を動的に呼び出したい
* 本体が更新されなくても、**サードパーティが拡張可能な構造**にしたい（Flake8, pytest などが該当）

これに対応するPython標準の方法が、**`entry_points` を使ったプラグイン構造**です。

---

## ✅ 28.2 仕組みの全体像

1. **プラグイン側が** 自分のモジュールと関数を `pyproject.toml` に登録
2. **ホスト側（本体）が** `importlib.metadata.entry_points()` を使って全プラグインを列挙
3. 必要なものだけを `importlib` 経由で呼び出す

この構造は `pip install` によって発見・連携できるため、**インストールするだけで拡張が可能**になります。

---

## ✅ 28.3 プラグイン側（提供側）の記述

### ディレクトリ構成例

```plaintext
myplugin/
├── src/
│   └── myplugin/
│       └── plugin_module.py
├── pyproject.toml
```

### `pyproject.toml`

```toml
[project]
name = "myplugin"
version = "0.1.0"
dependencies = []

[project.entry-points."pokecli.plugins"]
fetch = "myplugin.plugin_module:fetch"
```

この設定により、`pokecli.plugins` という名前空間で `fetch` というプラグインが定義されます。

---

## ✅ 28.4 ホスト側（本体）の読み込み方法

```python
from importlib.metadata import entry_points

def load_plugins():
    eps = entry_points(group="pokecli.plugins")
    return {ep.name: ep.load() for ep in eps}

plugins = load_plugins()
plugins["fetch"]()  # myplugin.plugin_module.fetch() が呼ばれる
```

ここで `ep.name` は `"fetch"`、`ep.load()` は対応する関数です。
この構造によって、**他人が定義した処理をコード修正なしで呼び出す**ことができます。

---

## ✅ 28.5 応用設計：プラグインによるCLI拡張

ホスト側CLIを次のように設計します：

```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("command", choices=plugins.keys())
    args = parser.parse_args()
    plugins[args.command]()
```

この設計により、

```bash
pokecli fetch
pokecli analyze
```

のように、**本体コードを書き換えずに新しいコマンドが増やせる**構造になります。

---

## ✅ 28.6 entry\_points 設計上の注意点

| 注意点                        | 説明                         |
| -------------------------- | -------------------------- |
| エントリーポイント名は重複できない          | 複数のプラグインが同じ名前を使うと競合する      |
| 複数の entry\_points グループを使える | CLI用・拡張用など、目的別に名前空間を分けられる  |
| `ep.load()` 失敗時に例外が出る      | 検出前に `try/except` で確認すると堅牢 |
| `pip install -e .` 時も反映される | 開発中でも即時にCLIから利用可能になる       |

---

## ✅ 28.7 プラグイン構造の活用場面

| 用途                       | 利点                                    |
| ------------------------ | ------------------------------------- |
| LLMラッパーCLIに対する拡張プロンプト群   | 外部定義されたプロンプトをCLI経由で選択可能に              |
| データ解析ツールにおける解析モジュール群     | 新しい手法を追加してもCLI側を変更する必要がない             |
| コマンドラインアプリのコマンド増殖に対応した設計 | サブコマンド追加を本体コードから切り離して、pipインストールだけで有効化 |

---

## ✅ 設計原則まとめ

| 原則                                       | 意図                              |
| ---------------------------------------- | ------------------------------- |
| 拡張対象はエントリポイント経由でしか認識させない                 | importlib直呼び出しを避けることで本体側と独立性を保つ |
| グループ名は衝突しにくい名前空間にする（例：`pokecli.plugins`） | 他ツールやライブラリとの干渉を避ける              |
| インストールされたプラグインは毎回スキャンされる                 | キャッシュ・無効化等の戦略を将来的に導入可能にする設計を取る  |

---

## ✅ まとめ

* `entry_points` を使うことで、Pythonプロジェクトを**他者が拡張可能な構造**にできる
* `importlib.metadata.entry_points()` により、**本体コードの変更なしで機能追加**が可能
* CLI・LLMツール・解析パイプライン等において、**安全かつ標準的な拡張方式**として実用性が高い

---


了解しました。第29章では、**LLM APIを扱うCLIツールにおけるimport設計と拡張性**を解説します。
前章の `entry_points` を活用しつつ、**LLMのような外部APIクライアントを柔軟に構築する構造**を設計面から明確化します。

---

# 📘 import29 - LLM対応CLIにおける拡張設計とimport戦略

---

## ✅ 29.1 背景と問題設定

LLM（大規模言語モデル）をPythonで活用するCLIを作る際、次のようなニーズが想定されます：

* 複数のプロンプト・用途をCLIコマンドとして切り替えたい
* プロンプト定義・テンプレート・処理を外部に分離したい
* 本体コードを書き換えずに、新たなプロンプトを追加したい（≒プラグイン化）
* 開発者やチームごとにLLMへのリクエスト処理を柔軟に差し替えたい

このようなCLI設計では、**importをどう設計するか**が拡張性・保守性・信頼性に大きく影響します。

---

## ✅ 29.2 基本設計：本体とプロンプト処理を分離

```plaintext
llmcli/
├── src/
│   ├── llmcli/
│   │   ├── core.py           ← API呼び出しロジック
│   │   └── commands/
│   │       ├── __init__.py
│   │       ├── translate.py  ← 翻訳用
│   │       └── summarize.py  ← 要約用
├── pyproject.toml
```

ここでは、`commands/` 配下の各ファイルが「1つのプロンプト処理ユニット」に対応し、
`main()` を通じてCLI実行可能な状態とします。

---

## ✅ 29.3 コマンド登録の方法（pyproject.toml）

```toml
[project.entry-points."llmcli.commands"]
translate = "llmcli.commands.translate:main"
summarize = "llmcli.commands.summarize:main"
```

この形式により、CLIから以下のように使える：

```bash
llmcli translate --text "これはテストです"
llmcli summarize --file notes.md
```

---

## ✅ 29.4 プロンプト定義とmainの責務分離

`llmcli.commands.translate` のようなモジュールは次のような構造にすべきです：

```python
from llmcli.core import ask_llm

def make_prompt(text: str) -> str:
    return f"次の文を英訳してください：{text}"

def main():
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--text", required=True)
    args = parser.parse_args()

    prompt = make_prompt(args.text)
    response = ask_llm(prompt)
    print(response)
```

このように：

* `main()` → CLI引数処理のみ
* `make_prompt()` → プロンプト構築処理（可読性・再利用性を高める）
* `ask_llm()` → API呼び出しの抽象化（本体 `core.py` に集約）

と明確に役割分担する。

---

## ✅ 29.5 `ask_llm()` の中身（OpenAI API例）

```python
import openai

def ask_llm(prompt: str) -> str:
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
    )
    return response["choices"][0]["message"]["content"]
```

ここを他のAPI（Anthropic、Gemini、自己ホストLLMなど）に差し替えられるよう、
**core層をimport切替可能な構成にするのが肝要**。

---

## ✅ 29.6 拡張方法：新しいプロンプトを追加するには？

1. `llmcli/commands/mycustom.py` を作る
2. `main()` を実装（引数とプロンプト定義を含める）
3. `pyproject.toml` に登録：

```toml
[project.entry-points."llmcli.commands"]
mycustom = "llmcli.commands.mycustom:main"
```

4. `pip install -e .` で有効化

これでCLI上では：

```bash
llmcli mycustom --input foo
```

が即座に使えるようになる。

---

## ✅ 29.7 プラグイン提供側の設計例（第三者による拡張）

プラグインとして `llmcli_translate_advanced` のような別プロジェクトを作成：

```toml
[project.entry-points."llmcli.commands"]
advanced_translate = "llmcli_adv.main:run"
```

→ 本体は一切更新せずに新コマンドが使える。

---

## ✅ 29.8 設計原則とベストプラクティス

| 原則                             | 意図                               |
| ------------------------------ | -------------------------------- |
| CLIコマンドを最小のファイル単位に分割           | 修正範囲を限定し、プロンプト修正の影響を抑える          |
| `main()` は「UI」として限定する          | ロジックをUI層から切り離してテスト・再利用可能にする      |
| `ask_llm()` をcoreに集約しAPI切替に備える | モデル・プロバイダを変更しても構造全体は保たれる         |
| `entry_points` を活用して外部拡張を許容する  | LLM CLI を **ツールキット化** できるようにする構造 |

---

## ✅ まとめ

LLMを使ったCLIツールは単なるスクリプトに留まらず、

* **汎用化された呼び出し構造**
* **外部プロンプト・機能の追加**
* **プラグインとして他者に配布可能な設計**

を取り入れることで、「プロンプトを構成するための開発基盤」として進化します。

本章の設計方針は、**業務用LLMツールのコマンド体系**、**ローカルLLMの実験環境**、
**AIエージェントの構築基盤**にまで適用可能です。

---




# 📘 import30 - 公開・運用

---

## ✅ 30.1 問題設定

ここまでで、Pythonプロジェクトにおける import の基礎・応用・CLI・プラグイン構造を網羅してきました。
最終章では、それらを **他人が使用可能な形で公開し、長期的に運用するための条件** を整理します。

### 想定されるフェーズ：

1. **ローカル開発**（自分だけが使う構造）
2. **他者利用**（チーム、外部ユーザー、GitHub公開）
3. **継続運用・保守**（LLM/CLI/設定/importの整合性を維持）

---

## ✅ 30.2 公開時に問題となりやすいimport構造の失敗例

| 失敗例                                    | 問題点                                      |
| -------------------------------------- | ---------------------------------------- |
| `src` 構成を使わずルートに直置きしている                | `pip install .` 時に意図せぬファイルが import 対象になる |
| `sys.path.append` を多用                  | ローカルでは動くが、環境差異やパッケージ化で import 失敗         |
| `if __name__ == "__main__"` に処理が集中している | テスト・再利用・CLI登録が不可能になる（関数化すべき）             |
| import順が循環している                         | CLIからの起動や一部ツールで `ImportError` が発生        |
| CLIを`setup.py`から登録している                 | 現在の主流（pyproject.toml）に非対応。ツールとの連携が難しくなる  |

---

## ✅ 30.3 正しい構造の最終形（再掲・総合版）

```plaintext
myproject/
├── src/
│   └── mylib/
│       ├── __init__.py
│       ├── core.py
│       └── cli/
│           ├── __init__.py
│           ├── fetch.py
│           └── serve.py
├── tests/
│   └── test_core.py
├── config/
│   └── default.toml
├── pyproject.toml
├── README.md
└── LICENSE
```

---

## ✅ 30.4 pyproject.tomlの最終構成例

```toml
[project]
name = "mylib"
version = "1.0.0"
description = "Reusable Python CLI toolkit"
authors = [{ name = "Your Name" }]
readme = "README.md"
license = { text = "MIT" }

[project.scripts]
mylib-fetch = "mylib.cli.fetch:main"
mylib-serve = "mylib.cli.serve:main"

[tool.setuptools]
package-dir = {"" = "src"}

[tool.setuptools.packages.find]
where = ["src"]

[tool.black]
line-length = 88

[tool.pytest.ini_options]
testpaths = ["tests"]
```

---

## ✅ 30.5 公開前チェックリスト

| 項目                  | チェック内容                                                  |
| ------------------- | ------------------------------------------------------- |
| importパスの整理         | `src/` + `__init__.py` + `pyproject.toml` で明示的に構成されているか |
| CLI設計の分離            | CLI処理とロジック処理が分離されており、どちらか片方を変更しても壊れないか                  |
| 動的importの明示         | `importlib` による呼び出しは、想定範囲内に限定されているか                     |
| プラグイン設計の拡張性と制御      | `entry_points` が整理されており、悪意あるコードが混入しにくい構造か               |
| 開発・運用の差異吸収（-eと本番の差） | `pip install -e .` でも本番でも同じ構造で動作するよう設計されているか            |

---

## ✅ 30.6 運用フェーズにおけるimport戦略

| 状況                      | 推奨対応                                   |
| ----------------------- | -------------------------------------- |
| 本番環境では自動importを制限       | 明示的なロードのみに制限し、プラグイン等は別レイヤーに配置          |
| モジュール名の変更は内部のみにとどめる     | CLIのエントリポイント名や外部向けAPIのインタフェースは変更しない    |
| サードパーティ拡張を受け入れる場合は検証必須  | `entry_points` 登録前に明示的なセキュリティレビューを設ける  |
| import失敗は握りつぶさず明示的にログ出力 | CLIで捕捉できるようにし、環境依存問題や未登録プラグインの特定を容易にする |

---

## ✅ 30.7 最終設計指針の総括

| 原則                                                | 意図                                        |
| ------------------------------------------------- | ----------------------------------------- |
| import経路は **明示・静的** を基本とし、例外として `importlib` を使う   | 再現性と静的検査への耐性を確保                           |
| CLI・設定・本体処理は **ファイル階層と関数レベルで明確に分離**               | 各レイヤーの責務を区別し、単体テスト・再利用・修正を容易にする           |
| `pyproject.toml` は **パッケージ＋ツール設定の統合管理ポイント** として使う | 分散した設定ファイルを廃止し、整合性のある構成と保守を実現             |
| 公開前に **環境ごとのimport検証**（CIなど）を入れる                  | ローカルで動いていたものが `pip install` 後に壊れる問題を未然に防ぐ |

---

## ✅ まとめ

本章は、「import設計を支える最後の壁」として、実運用・公開時に必要な構造管理と設計の原則を扱いました。

これまでの章と組み合わせれば、以下が可能になります：

* 開発者自身がimportの動作と制御を正確に理解し、壊れない構造を作れる
* CLIや設定ファイル、プラグインの設計も含め、import設計が全体構造と整合する
* 社内外での再利用・配布・保守において、信頼性のあるプロジェクトとして通用する

---

