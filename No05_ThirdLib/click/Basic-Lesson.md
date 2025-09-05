## 1章: CLIとは何か

---

### CLIとGUIの違い

| 観点    | CLI（Command Line Interface） | GUI（Graphical User Interface） |
| ----- | --------------------------- | ----------------------------- |
| 操作方法  | コマンド入力                      | マウス・タッチ操作                     |
| 表現形式  | テキストベース                     | ビジュアルベース                      |
| 学習コスト | 高め（コマンド暗記が必要）               | 低め（直感的操作）                     |
| 自動化   | 容易（スクリプト化しやすい）              | 難しい（手動操作が基本）                  |
| 開発効率  | 高い（反復・バッチ処理に強い）             | UI構築に手間がかかる                   |
| リソース  | 少ない（描画なし）                   | 多い（グラフィック処理あり）                |

---

### REPLとの比較

| 観点   | CLI                               | REPL（Read-Eval-Print Loop）  |
| ---- | --------------------------------- | --------------------------- |
| 実行単位 | プログラム・ファイル・コマンド                   | 1行ずつの式                      |
| 状態保持 | 原則なし（ステートレス）                      | 状態あり（変数・定義が残る）              |
| 使用目的 | 実行・運用・ツール化                        | 試行錯誤・検証・学習                  |
| 代表例  | `curl`, `git`, `python myfile.py` | `python`, `ipython`, `node` |

---

### CLIのユースケースと設計思想

#### 主な用途

* 開発者向け操作ツール（例: Git, Docker, Poetry）
* サーバ運用（例: cron + CLI, log整形）
* バッチ処理・定期実行（例: データ取得・変換）
* API操作（例: AWS CLI, GCP CLI）

#### 設計方針

* **明示的な構成**：引数・オプションを明確に
* **自動化前提**：人間操作に頼らずバッチ連携しやすく
* **保守性重視**：UI変更の影響が少なく、長期利用に向く
* **拡張しやすさ**：処理追加や構造変更が容易な設計

---
## 2章: clickとargparseの比較

---

### 構文・柔軟性・保守性

| 比較項目  | argparse         | click                |
| ----- | ---------------- | -------------------- |
| 構文形式  | 明示的（パーサ構築）       | 宣言的（デコレータベース）        |
| 柔軟性   | 基本的な構文に限定        | ファイルパス, バリデーション, 色など |
| 保守性   | コード量が多くなりやすい     | 再利用・共通化がしやすい         |
| 拡張性   | 自作の仕組みで補完する必要がある | 多機能が標準で用意されている       |
| 自動ヘルプ | 明示的に定義           | `--help`が自動生成        |

---

### 初学者と実務での使い分け

| 観点    | argparse           | click             |
| ----- | ------------------ | ----------------- |
| 学習段階  | Python標準ライブラリを学ぶ目的 | 最初から実用性を求める場合     |
| スクリプト | 単機能で完結する用途         | 複数機能・複数ファイルのCLI構築 |
| 実務開発  | 拡張要件が少ない場合         | UI品質や保守性を重視する場面   |
| チーム開発 | 標準ライブラリで統一したいとき    | 分担や拡張を想定した設計      |

---

### clickを選ぶ理由

* デコレータ中心で**直感的な記述**が可能
* `@click.group()` によって**複数コマンドを構造化**できる
* `click.Path`, `click.Choice`, `click.secho` などの**補助機能が豊富**
* **自動でヘルプ生成・型変換・エラーハンドリング**がついてくる
* CLI全体を\*\*「UI」として整備\*\*でき、poke-pipelineのような保守案件に最適

---
## 3章: clickの基本構造

---

### 基本構文と主要デコレータ

```python
import click

@click.command()
@click.option('--name', prompt='Your name', help='表示する名前')
def hello(name):
    click.echo(f'Hello, {name}!')
```

| 要素                  | 説明                            |
| ------------------- | ----------------------------- |
| `@click.command()`  | 関数を1つのコマンドに変換                 |
| `@click.option()`   | オプション引数を追加（`--name`など）        |
| `@click.argument()` | 位置引数を追加（順番に渡す必須引数）            |
| `click.echo()`      | `print()`に似るが、エンコーディングとパイプに強い |

---

### デコレータ構造のポイント

* 複数の `@click.option()` は**上から順に適用される**
* 関数の引数名と一致していないとエラーになる
* 型変換・バリデーションは `@click.option(..., type=...)` で一括管理できる

---

### CLI関数の実行方法

#### 単体ファイルとして実行

```python
if __name__ == '__main__':
    hello()
```

```bash
$ python hello.py --name Pikachu
```

#### モジュールとして実行

```bash
$ python -m cli.hello --name Pikachu
```

* `-m` 実行は**補完が効きやすく、importも安定**
* 複数コマンドを持つ構成に進む場合、`@click.group()` へ発展させる

---
## 4章: コマンドの実行方法

---

### `python -m` 実行の利点

```bash
$ python -m cli.fetch --endpoint pokemon
```

| 実行形式               | 特徴                   |
| ------------------ | -------------------- |
| `python file.py`   | 単体スクリプトとして直接実行       |
| `python -m module` | パッケージとしてモジュール単位で実行可能 |

* `-m`を使うと**importと補完が安定**
* ルート配下の`cli/`ディレクトリを直接CLI化できる
* 実行対象が**サブコマンド形式に拡張しやすい**

---

### モジュール構造におけるメリット

* CLIをモジュールとして扱えば、他コードからも呼び出しやすい
* テスト・補助スクリプト・補完設定の組み込みが容易になる
* `__main__.py`に依存しない構造は**保守性が高い**

---

### `__main__` 直実行との違い

| 項目        | `python file.py` | `python -m module`   |
| --------- | ---------------- | -------------------- |
| 依存性       | ファイルパスに依存        | モジュールパスに依存（import前提） |
| import安定性 | 相対importが壊れやすい   | importエラーが起きにくい      |
| 大規模構成との相性 | 弱い               | 強い（複数ファイル化しやすい）      |

---

### poke-pipeline構成での位置づけ

* `cli/`以下をパッケージ化し、`python -m cli.fetch` で実行
* `main.py`から各コマンドファイルを**独立的に読み込む設計**
* 実行可能性と再利用性を両立する、**標準的な構成方式**

---
## 5章: オプションと引数
---

### 概念の比較

| 区分  | 引数（argument）      | オプション（option）           |
| --- | ----------------- | ----------------------- |
| 書き方 | `command <value>` | `command --key <value>` |
| 順序  | 決まっている            | 自由（順不同）                 |
| 必須性 | 原則必須              | デフォルト可・任意               |
| 表示  | `--help` に出ない     | `--help` に説明付きで表示される    |
| 用途  | 主対象（ファイル名など）      | 設定項目・制御フラグ（モードなど）       |

---

### clickにおける使い分け

```python
@click.command()
@click.argument('filename')
@click.option('--verbose', is_flag=True)
def load(filename, verbose):
    ...
```

* `filename`：必ず指定しなければならない（位置引数）
* `--verbose`：省略可能。指定時のみ真になる（オプション）

---

### 属性・型・UIとしての設計

| 項目       | click.option                      | click.argument  |
| -------- | --------------------------------- | --------------- |
| default  | `default="en"` のように指定可能           | 不可（常に値が必要）      |
| required | `required=True` で必須にできる           | 常に必須            |
| type     | `type=int`, `type=click.Path()` 等 | `type=`で同様に指定可能 |
| prompt   | 対話的入力を受け取る                        | 対話入力不可          |

---

### UI設計上の使い分け指針

* **argument**：ファイル名・IDなど、**主対象**であり省略不可な値
* **option**：設定やモード指定など、**補助的で柔軟な制御項目**

CLIをUIと見なしたとき、argument は「フォームの必須入力欄」、option は「オプションチェックボックス」に相当する。

---
## 6章: 型変換とバリデーション
---

### 基本的な型指定

```python
@click.option('--count', type=int)
@click.option('--path', type=click.Path(exists=True))
@click.option('--lang', type=click.Choice(['en', 'ja']))
```

| 型種別            | 説明                         |
| -------------- | -------------------------- |
| `int`, `float` | 数値変換                       |
| `str`          | 明示しなくても自動で文字列として扱われる       |
| `bool`         | `is_flag=True` によるフラグ形式に使う |
| `click.Path`   | ファイル・ディレクトリの存在チェックが可能      |
| `click.Choice` | 選択肢を限定（自動でヘルプ表示にも反映）       |

---

### promptによる対話入力

```python
@click.option('--username', prompt='Enter your name')
```

* 入力がなかった場合、対話的に質問する
* パスワード入力なども `hide_input=True` で対応可能
* CLIを**人間の対話UIとして使う**場合に有効

---

### エラーハンドリングの自動化

* 指定された型に変換できない場合、**自動でエラーを表示して中断**
* `click.BadParameter` や `click.ClickException` により、**美しいエラーメッセージ**が出力される

```bash
$ python app.py --count not_a_number
Error: Invalid value for '--count': 'not_a_number' is not a valid integer
```

---

### 型変換の活用例（poke-pipeline）

* `--id 25` → `int`型で処理し、ファイル名生成や条件分岐に利用
* `--endpoint pokemon` → `click.Choice([...])`で限定
* `--overwrite` → `is_flag=True` で上書き制御

---

### 設計指針まとめ

* 型指定により**安全性・明確性・自動補助**が向上する
* 複雑なUI制御なしで、**正確な入力と自動検証**を実現可能
* **人にもスクリプトにも優しいCLI**に近づける重要ポイント

---
## 7章: コマンドグループの設計

---

### `@click.group()` による構造化

```python
import click

@click.group()
def cli():
    pass

@cli.command()
def fetch():
    click.echo("Fetching...")

@cli.command()
def extract():
    click.echo("Extracting...")
```

* `@click.group()` で**親コマンド**を定義
* `@cli.command()` で**サブコマンド**を追加
* 各サブコマンドは独立した関数として定義でき、**拡張が容易**

```bash
$ python -m cli.main fetch
$ python -m cli.main extract
```

---

### コマンド群の系統整理

poke-pipelineでは以下の4系統で構成：

| コマンド        | 役割          |
| ----------- | ----------- |
| `fetch`     | JSONの取得     |
| `extract`   | 抽出と整形       |
| `transform` | データ構造の変換・加工 |
| `all`       | 上記3つを連続して実行 |

---

### グループ設計の利点

* CLI全体を**論理的に整理**できる
* ヘルプ表示で**全体構造が一目瞭然**
* 将来的な追加・削除も**影響範囲を限定**できる
* ユーザーにも開発者にも**見通しの良いCLI**を実現

---

### 実装における推奨構成

```text
cli/
├── main.py        # group定義
├── fetch.py       # 各コマンドの実体
├── extract.py
├── transform.py
├── all.py
```

* `main.py` に `cli()` グループを定義
* 各ファイルで `@cli.command()` を使って個別登録
* 構造化されたCLIは**実務レベルでの運用に適する**

---
## 8章: 複数ファイル構成の設計指針

---

### ディレクトリ分割の基本方針

```text
cli/
├── main.py        # コマンドグループの定義
├── fetch.py       # fetchコマンドの処理
├── extract.py     # extractコマンドの処理
├── transform.py   # transformコマンドの処理
├── all.py         # 一括実行コマンド
```

* 各ファイルが**1コマンドを担当**する構成に分離
* main.py は `@click.group()` とエントリポイントのみを保持
* **責務ごとに明確に分離**することで、拡張・保守・見通しが向上

---

### import設計と依存の整理

```python
# main.py
from cli.fetch import fetch
from cli.extract import extract
...
```

* 明示的に各コマンド関数をインポートして登録
* 相互依存を避ける構成にする（fetch.pyがextract.pyをimportしない）

---

### 依存を最小化するポイント

* **共通処理や定数は util/ や config/ に逃がす**
* 1ファイル＝1機能 に徹し、**モジュール内の循環importを防ぐ**
* 構造上の分離は**テストやLLM連携にも効果的**

---

### poke-pipelineにおける応用

* `cli/` は単なるスクリプト集ではなく、**ツール群としてのUI層**
* 下層（`src/`, `util/`, `config/`）との依存方向を**一方向に保つ**
* 実行系ロジックを`cli/`に集約することで、**実行制御・ログ制御を統一**可能に

---

### 設計原則のまとめ

* CLI層は**UI**と見なす
* **分割・独立・単一責任**を徹底
* **読みやすさ・保守性・LLM支援**を見越した構造にする

---
## 9章: 開発用・本番用コマンドの分離

---

### main\_for\_debug構成

```python
def main_for_debug():
    fetch.main(standalone_mode=False)
```

* `standalone_mode=False` により**エラー時も例外を投げる**（デバッグ向け）
* `if __name__ == '__main__'` ブロック内でのみ呼び出す
* **一時的な検証・対話テスト**に便利

---

### `--debug`, `--dry-run` オプション

```python
@click.option('--debug', is_flag=True, help='デバッグ出力を有効にする')
@click.option('--dry-run', is_flag=True, help='実行せずに内容を確認')
```

| オプション       | 用途                   |
| ----------- | -------------------- |
| `--debug`   | ログ出力の詳細化やスタックトレースの出力 |
| `--dry-run` | 実際の処理をスキップして確認だけ行う   |

* **破壊的操作の防止**や**挙動確認**に効果的
* 安全性とUXを高めるために、**実務でも標準装備が推奨される**

---

### 試験的引数・補助機能の封じ込め方

* 本番運用に不要なオプションや処理は**隠蔽・明示制御**する
* 例：

  * `--internal-cache-path`（内部処理の指定）
  * `--force`（強制実行系）

```python
if debug:
    click.echo("[debug] 実行前に状態を確認中")
```

---

### poke-pipelineでの活用例

* fetch系コマンドに `--dry-run`, `--delay` を追加し、**ループ実行中の挙動確認**に利用
* `main_for_debug()` を一時的に使って**ローカル実行・VSCode補完付きのデバッグ**が可能

---

### 設計指針まとめ

* **本番と開発で同じコードを使いつつ、安全に切り替える**
* デバッグ時の柔軟性を高めつつ、本番では無効化
* **試験用途と保守拡張を両立させるための設計層**

---
## 10章: 設定ファイルとclickの統合

---

### 環境変数との統合: `envvar`

```python
@click.option('--token', envvar='API_TOKEN', help='APIトークン')
```

* オプションが未指定なら、環境変数 `API_TOKEN` を参照
* `.env` ファイルで定義し、`python-dotenv` などで読み込むと便利
* **コマンドライン非表示で機密情報を扱える**メリットあり

---

### tomlやenvファイルの活用

#### `.env` の例

```env
API_TOKEN=abcdef123456
ENDPOINT=pokemon
```

#### tomlの例（`config.toml`）

```toml
[api]
token = "abcdef123456"
```

* `dotenv_values` や `toml.load` を使って、設定ファイル → dict に変換し、`callback`や`default`に渡す設計が可能
* **CLIと設定ファイルの併用**で柔軟性が大幅に向上

---

### 優先順位ルール（推奨）

1. コマンドラインでの指定（`--token=...`）
2. 環境変数（`API_TOKEN`）
3. 設定ファイル（`.env`, `.toml`）
4. オプションのdefault引数

* 明示的な指定が常に最優先
* **値の追跡が容易な設計にする**

---

### poke-pipelineでの活用方針

* `--endpoint`, `--lang` などは `.env` にも対応
* デフォルト値を環境ごとに切り替えたい場合に `.env.dev`, `.env.prod` などを使い分け
* CLIオプションと `.env` を両立させ、**本番用と開発用の動作を安全に分離**

---

### 設計指針まとめ

* **設定の受け口をCLIに集約し、選択肢を用意する**
* ユーザー・スクリプト・CIツールのどれからも使いやすい構成に
* **機密情報やデフォルト設定を外部に逃がすことで安全性と移植性が向上**

---
## 11章: 標準出力とロギング
---

### `click.echo()` vs `print()`

| 出力関数           | 特徴                                  |
| -------------- | ----------------------------------- |
| `click.echo()` | Unicode対応・出力先切替・stderr対応・自動flushあり  |
| `print()`      | 標準のPython関数。環境依存で文字化けやバッファ問題が発生しやすい |

```python
click.echo("成功しました")              # stdoutへ
click.echo("エラー発生", err=True)     # stderrへ
```

---

### ログ出力との併用

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

@click.command()
def main():
    logger.info("処理開始")
    click.echo("ユーザーに通知")
```

* `logger.info()` はログファイルや監視に適する
* `click.echo()` はユーザー向けの**即時通知**に適する
* CLIツールでは **「人向け：echo」「記録用：logger」** の分離が基本方針

---

### stdout/stderrの設計分離

| 種別     | 用途例               |
| ------ | ----------------- |
| stdout | 正常系の通知、処理結果の出力など  |
| stderr | エラー、警告、無視できない異常など |

```python
click.echo("保存完了", err=False)  # stdout
click.echo("保存失敗", err=True)   # stderr
```

* `err=True` を明示することで、パイプ処理やCIでの判別が容易になる

---

### poke-pipelineでの運用

* `click.echo()` → ユーザーへの実行通知（例: 「fetch開始」）
* `logger.info()` → 内部ログ記録（例: ファイル保存パス、件数など）
* `logger.warning()` / `click.echo(err=True)` → 異常系通知の二重出力

---

### 設計方針まとめ

* **人間用と機械用の出力を分離**する
* echoはUIとして、loggerは記録用として扱う
* stderrの活用により**失敗検出とパイプ処理の堅牢性**を確保できる

---
## 12章: 例外処理とexitコードの扱い

---

### `click.ClickException` による統一的な例外処理

```python
raise click.ClickException("エラー内容を表示")
```

* 自動で `stderr` に出力され、終了コードは `1` になる
* スタックトレースは出さず、**ユーザー向けの明快なエラー表示**に特化
* 実行中に例外を握り潰さず、**失敗を正しく伝えるCLI**を構築可能

---

### `abort()` による即時中断

```python
click.echo("中断します", err=True)
click.abort()
```

* `abort()` は `ClickException: Aborted!` を投げ、**強制終了**
* 確認プロンプトや条件チェックで「キャンセルされたとき」に使用される

---

### `sys.exit()` との使い分け

| 方法                     | 用途                   |
| ---------------------- | -------------------- |
| `click.ClickException` | 正常なCLIとしてのエラー通知      |
| `click.abort()`        | 明示的な中断（プロンプト失敗など）    |
| `sys.exit(code)`       | CLI外の例外や他モジュールの終了処理用 |

```python
import sys
if error_detected:
    sys.exit(2)  # 特定の異常を明示
```

* `0` → 正常終了
* `1以上` → 異常終了（CIやバッチで判定に使える）

---

### poke-pipelineでの活用例

* ファイル書き込み失敗 → `ClickException("保存に失敗しました")`
* `--dry-run` で確認後中断 → `click.abort()`
* 非CLI層での異常伝搬 → `sys.exit()` を使って明示終了コードを返す

---

### 設計指針まとめ

* **終了コードはCLIツールとしての信頼性を左右する**
* ユーザーには簡潔に、開発者には意味を持つ形で例外を扱う
* 正常終了 / エラー終了 / 中断 を**明示的に区別する実装が重要**

---
## 13章: コールバックと依存性注入

---

### callbackによる事前処理

```python
def validate_id(ctx, param, value):
    if value < 0:
        raise click.BadParameter("IDは0以上である必要があります")
    return value

@click.option('--id', callback=validate_id, type=int)
```

* `callback` 関数はオプションの**値を受け取って検証・変換**できる
* エラー時は `click.BadParameter` を投げて**CLI向けエラーメッセージを表示**

---

### ctx（context）を使った複合処理

```python
def validate(ctx, param, value):
    config = ctx.ensure_object(dict)
    config['id'] = value
    return value

@click.command()
@click.option('--id', callback=validate)
@click.pass_context
def main(ctx, id):
    click.echo(f"ctx経由のID: {ctx.obj['id']}")
```

* `ctx` はCLI実行時の**状態コンテナ**
* `ctx.obj` を使えば、**複数オプション間で値を共有**できる

---

### バリデーションの再利用

* `@click.option(..., callback=...)` によって、**同じ検証ロジックを複数のCLIで再利用可能**
* 引数を変えたいときは\*\*部分適用（functools.partial）\*\*なども利用できる

```python
from functools import partial
@click.option('--lang', callback=partial(validate_choice, choices=['en', 'ja']))
```

---

### poke-pipelineでの活用例

* `--id`, `--lang`, `--endpoint` などに対して事前検証を設け、**整合性のない実行を防止**
* 設定ファイルからの読み込みとCLI引数の整合性確認にも使用可能
* `ctx.obj` を使えば、**モジュールまたぎでオプション値を伝播**できる

---

### 設計指針まとめ

* バリデーションは**CLIの信頼性とUXを高める第一歩**
* callbackは「処理の入り口を守る門番」
* 複雑な処理を分離し、**CLIを薄く保つ設計**に貢献する

---
## 14章: 複数オプションの相関制御

---

### 代表的な相関パターン

| パターン         | 説明例                              |
| ------------ | -------------------------------- |
| 排他（AかBのどちらか） | `--id` か `--all` のどちらか一方のみ指定可能   |
| 両方必須         | `--start` と `--end` を両方指定しないとエラー |
| 片方省略不可       | `--id` を指定したら `--lang` も必要       |

---

### 実装例: 排他（XOR）

```python
@click.command()
@click.option('--id', type=int)
@click.option('--all', is_flag=True)
def main(id, all):
    if (id is None) == (not all):
        raise click.UsageError("`--id` か `--all` のどちらか一方を指定してください")
```

* `XOR`（一方だけ）制約を**明示的にifで定義**
* `UsageError` により、ユーザーに**ヘルプ系のエラー**として伝わる

---

### 実装例: 両方必須（AND）

```python
@click.option('--start')
@click.option('--end')
def main(start, end):
    if not start or not end:
        raise click.UsageError("`--start` と `--end` の両方が必要です")
```

---

### 実装例: 片方省略不可（依存関係）

```python
@click.option('--id')
@click.option('--lang')
def main(id, lang):
    if id and not lang:
        raise click.UsageError("`--id` を指定したら `--lang` も必要です")
```

---

### 設計上の注意点

* clickには**組み込みの相関制御機能はない**
* 明示的な `if` 条件と `UsageError` によって制御する
* **コールバックや`ctx`と組み合わせれば、柔軟な事前チェックが可能**

---

### poke-pipelineでの応用

* `--endpoint` 指定時に `--id` か `--all` を選ばせる構成に使用
* `--lang` が必要なAPIでは、`--id` との依存を定義
* 必要に応じて `ctx.obj` を用い、**他オプションとの関係性も検証**

---

### 設計指針まとめ

* CLIは**自由度が高い分、明示的な制約が必要**
* 相関チェックによって**誤操作を未然に防ぐ**
* 複雑な相関は `if` + `UsageError` + `ctx` による段階的制御が最適

---
## 15章: 共通オプションの再利用

---

### 課題：複数コマンドで同じオプションを定義したい

```python
@click.command()
@click.option('--lang', type=click.Choice(['en', 'ja']), default='en')
def fetch(lang):
    ...

@click.command()
@click.option('--lang', type=click.Choice(['en', 'ja']), default='en')
def extract(lang):
    ...
```

* **同じオプション定義が重複**する
* 変更・保守が面倒になりやすく、**DRY原則に反する**

---

### 解決策：共通デコレータの定義

```python
def common_lang_option(func):
    return click.option(
        '--lang',
        type=click.Choice(['en', 'ja']),
        default='en',
        show_default=True,
        help='使用言語（enまたはja）'
    )(func)
```

* 通常のデコレータと同様に関数として記述
* `@common_lang_option` として複数コマンドに適用可能

```python
@click.command()
@common_lang_option
def fetch(lang):
    ...
```

---

### 複数オプションのまとめ定義

```python
def common_options(func):
    func = click.option('--dry-run', is_flag=True)(func)
    func = click.option('--lang', type=click.Choice(['en', 'ja']), default='en')(func)
    return func
```

```python
@click.command()
@common_options
def fetch(lang, dry_run):
    ...
```

* **複数オプションを1つの関数でまとめて再利用可能**
* 順序がある場合は**下から順に適用されることに注意**

---

### poke-pipelineでの応用

* `--lang`, `--dry-run`, `--delay` などを**複数コマンドで共通化**
* `@common_fetch_options` や `@common_extract_options` などの**用途別定義**も可能
* 将来的に `cli.base_options` モジュールで一元管理する構成も検討可

---

### 設計指針まとめ

* **共通オプションは装飾関数化して使い回す**
* 重複を避けることで保守性と一貫性が向上
* 複雑化したCLIでも**構造が破綻しにくくなる**

---
## 16章: click.Pathとファイル操作

---

### `click.Path` の基本

```python
@click.option('--input', type=click.Path(exists=True, dir_okay=False))
@click.option('--output', type=click.Path(writable=True, dir_okay=False))
```

| 引数               | 説明                  |
| ---------------- | ------------------- |
| `exists=True`    | 存在確認（読み取り対象として想定）   |
| `writable=True`  | 書き込み可能かをチェック        |
| `dir_okay=False` | ディレクトリを拒否（ファイル専用）   |
| `file_okay=True` | ファイルを許容（デフォルトでTrue） |

* **ファイルの存在性や種類を自動で検証**
* `Path`型として処理できるので、`open()`等と連携が簡単

---

### 入出力ファイルのバリデーション例

```python
@click.command()
@click.option('--input', type=click.Path(exists=True))
@click.option('--output', type=click.Path())
def process(input, output):
    with open(input, 'r') as f:
        data = f.read()
    with open(output, 'w') as f:
        f.write(data)
```

* **読み込みファイルの存在性を強制**
* **出力ファイルの保存先を指定可能に**

---

### 相対パスと絶対パスの扱い

* `click.Path(resolve_path=True)` を指定すると、**絶対パスに変換して受け取れる**

```python
@click.option('--path', type=click.Path(resolve_path=True))
```

* スクリプト側でパスの解釈を行わずに済み、**パス指定ミスの防止になる**

---

### poke-pipelineでの活用例

* `--input` によるJSONファイル読み取り
* `--output` による変換後ファイルの保存先指定
* `--path` を `resolve_path=True` で受け取り、**ログ出力やデバッグの明確化にも利用**

---

### 設計指針まとめ

* `click.Path` によるパスチェックで**人為ミスを未然に防止**
* 入出力先の明示により、**CLIとしての信頼性と透明性が向上**
* 相対・絶対パスの混在にも**自動対応が可能な堅牢設計**

---
## 17章: 色付き出力とユーザー通知

---

### `click.secho()` の基本

```python
click.secho("成功", fg="green")
click.secho("警告", fg="yellow")
click.secho("エラー", fg="red", err=True)
```

| 引数     | 説明                     |
| ------ | ---------------------- |
| `fg`   | 前景色（文字色）               |
| `bg`   | 背景色                    |
| `bold` | 太字表示（`bold=True`）      |
| `err`  | 標準エラー出力に送る（`err=True`） |

* `print()` ではできない**CLI的なフィードバック**を提供
* 色は `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white` から選択

---

### `click.style()` で装飾だけ適用

```python
text = click.style("失敗", fg="red", bold=True)
click.echo(f"処理結果: {text}")
```

* `style()` は**文字列だけ生成**し、`echo()` や `logger` などに混ぜ込める
* テキストの中で一部だけ色付けしたいときに便利

---

### 成功・警告・失敗の標準出力例

```python
click.secho("[OK] データ取得成功", fg="green")
click.secho("[WARN] キャッシュを使用", fg="yellow")
click.secho("[ERROR] ファイル保存失敗", fg="red", err=True)
```

* こうした出力は、**バッチ処理やCIログに視認性を加える効果が高い**

---

### poke-pipelineでの活用例

* fetch 成功時に `green`、スキップ時に `yellow`、失敗時に `red`
* `--dry-run` モードでは警告色（`magenta` など）を用いることで**実行抑止を明示**
* エラー出力は `err=True` により stderr へ → **CIでの色分けログにも有効**

---

### 設計指針まとめ

* **色はCLIのUX向上に直結する**
* 成功/警告/失敗などの**状態の即時識別に不可欠**
* ログと標準出力を使い分けつつ、**視覚フィードバックを最大化する**

---
## 18章: 進捗表示とステータスバー

---

### `click.progressbar()` の基本構文

```python
with click.progressbar(length=100, label='処理中') as bar:
    for i in range(100):
        do_something()
        bar.update(1)
```

* `length=` は合計件数
* `label=` は表示されるメッセージ
* `.update(n)` で進捗を加算（1件ずつでも可）

---

### イテレータ対応の省略構文

```python
items = ['a', 'b', 'c']
with click.progressbar(items, label="変換中") as bar:
    for item in bar:
        process(item)
```

* イテラブルをそのまま渡すことで**簡潔に進捗表示**
* 件数が多い処理でも**ユーザーに待ち時間を可視化**できる

---

### ステータスメッセージ表示：`click.echo` + `end="\r"`

```python
click.echo("変換中...", nl=False)
# → 改行せずその行に上書き表示
```

* 動的な単行更新や**簡易なステータス出力**として有効
* 明示的に `\r`（復帰）を使えば、shellのような出力制御が可能

---

### poke-pipelineでの活用例

* `IDFetcher` の進捗に `progressbar` を適用
* `--verbose` 時に処理中のIDやファイル名も併記
* 大量fetch時でも**ユーザーが処理中か停止かを視認できる構成**

---

### 設計指針まとめ

* **長時間処理にはフィードバックが不可欠**
* 処理時間の見積りが困難な場面でも、**進捗表示がUXを支える**
* 非同期処理やスレッド実行にも応用可能な設計手段

---
## 19章: fetchコマンドの構築

---

### 目的と役割

* PokeAPI からデータを取得してローカル保存する処理を担当
* エンドポイント別に ID または URL を指定して**個別JSONをfetch**
* `public/raw/{endpoint}/{id}.json` に保存する設計

---

### 主なオプション設計

| オプション         | 説明                              |
| ------------- | ------------------------------- |
| `--endpoint`  | 対象エンドポイント（例: `pokemon`, `type`） |
| `--id`        | 単一IDを指定（整数）                     |
| `--lang`      | 言語（将来的に翻訳処理等で使用）                |
| `--overwrite` | ファイルが既に存在していても上書き保存するか（フラグ）     |

```bash
$ python -m cli.fetch --endpoint pokemon --id 25
```

---

### 構造化された保存処理

* 保存先は以下のように統一：

```text
public/raw/{endpoint}/{id}.json
```

* ファイル名がIDであることで、**重複管理・存在確認が容易**
* ディレクトリ構成を明確にすることで、**他工程（extract, transform）との連携がスムーズ**

---

### 実装ポイント

* `Path(...).exists()` によって保存前にファイルの存在を確認
* `--overwrite` が無効ならスキップ
* HTTPリクエスト処理に失敗した場合は `ClickException` により**明示的に失敗通知**

---

### fetchの将来的な拡張

* `--all` による全件取得（ParamFetcherによるIDリストからループ）
* `--delay` を使ったrate limit対策（スリープ処理）
* 多言語対応の切り替えなどにも対応可能（langを変数として保持）

---

### 設計指針まとめ

* CLIからの**柔軟な指定による取得処理**を提供
* データの一貫した保存形式とログ出力で**後続処理と接続**
* fetchは「poke-pipelineの入口」として最も重要な処理のひとつ

---
## 20章: extractコマンドの導入

---

### 目的と役割

* fetchで保存されたJSONから**必要な情報だけを抽出・整形**
* 加工後のデータを `public/extracted/{endpoint}/{id}.json` に保存
* flavor\_textやnameなど、**用途に応じたフィールド選択**を担当する処理

---

### 主な処理内容

| ステップ       | 説明                                                   |
| ---------- | ---------------------------------------------------- |
| ① ファイル読み込み | `public/raw/{endpoint}/{id}.json` をロード               |
| ② 抽出処理     | flavor\_text / names / type などの**キー抽出・構造変換**         |
| ③ 保存処理     | `public/extracted/{endpoint}/{id}.json` へ保存（上書き制御あり） |

---

### オプション例（想定）

| オプション         | 説明                            |
| ------------- | ----------------------------- |
| `--endpoint`  | 対象データ種別（例: `pokemon-species`） |
| `--id`        | 対象のID（fetchと同様）               |
| `--lang`      | 指定言語（flavor\_textなどの言語抽出に使用）  |
| `--overwrite` | 既存ファイルの上書き有無                  |

---

### flavorやlanguage対応の注意点

* flavor\_text\_entries などは**多言語混在** → `language.name == lang` の抽出が必要
* `names`, `genera`, `flavor_text_entries` などで **langによるフィルタ処理**が共通パターン

---

### 実装ポイント

* JSONのパースに失敗したら `ClickException` で明示
* キーが存在しないケースに対する `KeyError` ハンドリング
* 抽出失敗時は `--dry-run` モードで影響を抑えつつデバッグ可能に

---

### 設計指針まとめ

* extractは\*\*「生データ → 利用可能なデータ」への変換処理\*\*
* 再取得不要の前提で、保存構造と再実行性を確保
* flavor, name, language などの整形ルールを**明示的に定義し統一**

---
## 21章: transformコマンドと加工処理

---

### 目的と役割

* extract済みデータをさらに**構造変換・整形・統合**して、最終形式に仕上げる
* HTML描画やAPIレスポンス向けに最適化されたJSONに変換
* 出力は `public/transformed/{endpoint}/{id}.json` に保存される

---

### 主な処理内容

| ステップ    | 説明                                            |
| ------- | --------------------------------------------- |
| ① データ読込 | `public/extracted/{endpoint}/{id}.json` を読み込み |
| ② 加工処理  | キー名変更、構造の入れ替え、HTML向け整形など                      |
| ③ 出力処理  | `public/transformed/{endpoint}/{id}.json` に保存 |

---

### オプション設計（想定）

| オプション         | 説明                              |
| ------------- | ------------------------------- |
| `--endpoint`  | 対象データ種別                         |
| `--id`        | 単体変換対象ID                        |
| `--overwrite` | 出力ファイルの上書き許可フラグ                 |
| `--format`    | オプションで変換形式を切替（例: `html`, `api`） |

---

### 変換例（poke-species）

```json
{
  "id": 25,
  "name": "Pikachu",
  "flavor_text": "でんき　を　ためこむ　ほっぺたの　ふくろで...",
  "type": ["electric"]
}
```

* 多段ネストされていたJSONを、**使いやすい1階層に平坦化**
* **表示用途に最適なキー命名・順序・構造**に整えることが目的

---

### 実装ポイント

* `transform_{endpoint}.py` のようにモジュールを分離し、**変換ルールをファイル単位で管理**
* 共通処理（文字列整形、改行統一など）は `util/` に集約
* 例外処理・上書き判断もextractと共通で持たせる

---

### 設計指針まとめ

* transformは**poke-pipelineの「最終出力生成工程」**
* 汎用性の高い形式で出力し、フロントやAPIへの**受け渡しを前提に設計**
* 加工ルールは明示的に定義し、**一貫性と再現性**を担保する

---
## 22章: allコマンドの設計

---

### 目的と役割

* `fetch → extract → transform` の**3工程を一括実行**するコマンド
* スクリプト1行で全体処理を行えるようにすることで、**開発・検証・自動化が容易に**

```bash
$ python -m cli.all --endpoint pokemon --id 25
```

---

### 実行フローと順序制御

| ステップ      | 説明               |
| --------- | ---------------- |
| fetch     | PokeAPIからJSONを取得 |
| extract   | 必要な情報だけを抽出・整形    |
| transform | 最終的な表示・連携用形式に変換  |

* 各工程で `ClickException` を適切にcatchし、**処理を中断または継続**する制御が必要
* ログ出力・echoによる通知で**ユーザーに処理状況を提示**

---

### 主なオプション設計（最小構成）

| オプション         | 説明                          |
| ------------- | --------------------------- |
| `--endpoint`  | 対象エンドポイント（必須）               |
| `--id`        | 対象ID（個別処理）または `--all` で一括処理 |
| `--lang`      | 言語指定（flavor\_text等で使用）      |
| `--overwrite` | 中間ファイルや出力ファイルの上書き許可（任意）     |
| `--dry-run`   | 処理せずに対象やステップだけ確認（確認用）       |

---

### ステータス表示とログ連携

* 各工程の開始・終了を `click.echo()` で表示（色付きも有効）
* 処理時間・件数などのログを `logger.info()` で記録
* エラー時は `logger.error()` + `click.echo(err=True)` の併用で**ユーザーとログ両方に通知**

---

### 実装構成例（cli/all.py）

```python
@click.command()
@common_options  # endpoint, id, lang, overwrite, dry-runなど
def all(...):
    fetch_main(...)
    extract_main(...)
    transform_main(...)
```

* `fetch_main`, `extract_main`, `transform_main` は各CLIの内部関数として分離
* **共通関数として使い回せるように設計**

---

### 設計指針まとめ

* allは**poke-pipelineの自動化・統一実行インターフェース**
* スクリプト・CI・対話型操作すべてにおいて利便性を提供
* 状態やエラーの可視性を確保しつつ、**責務分離された構成を維持する**

---
## 23章: デバッグ用途の補助CLI

---

### main\_for\_debug構成の活用

```python
def main_for_debug():
    all.main(standalone_mode=False)
```

* `standalone_mode=False` によって、**例外がtry文で補足可能**になる
* 通常のCLI実行と同じコードパスを保ちつつ、**対話的なデバッグが可能**

---

### 補助オプションの設計例

| オプション       | 説明                        |
| ----------- | ------------------------- |
| `--dry-run` | 実際には保存せず、処理内容だけを出力（確認モード） |
| `--force`   | 通常は抑止される上書きや失敗処理を強制実行     |
| `--delay`   | fetch時にリクエスト間隔を追加（単位: 秒）  |

```python
@click.option('--delay', type=float, default=0.0, help='リクエスト間隔（秒）')
```

* **rate limit対策やリトライ制御の確認**に活用
* テスト時に**処理速度や副作用を抑えたい場合にも有効**

---

### 一時的CLIの封じ込め例

```python
@click.command(hidden=True)
def devtool():
    ...
```

* `hidden=True` にすることで `--help` に表示されない
* 一時的な内部コマンドや検証ツールを**意図的に隠す設計**

---

### テスト・開発モードでの分岐

```python
if dry_run:
    click.echo("[DRY RUN] 実行スキップ")
    return
```

* `--dry-run` や `--debug` によって、**本番と異なる挙動を安全に試行**できる
* フラグに応じた `logger.setLevel(logging.DEBUG)` の切替も可能

---

### poke-pipelineでの実用場面

* `main_for_debug()` により、`all`, `fetch`, `extract` それぞれの**動作確認が即座に可能**
* `--delay 0.5` により、APIへの影響を抑えて**慎重に全件取得**するテストが可能
* `--dry-run` は CI環境での**安全検証**にも有効

---

### 設計指針まとめ

* デバッグ用CLIは**本番ロジックを守りつつ試験可能にする安全弁**
* フラグ設計・main\_for\_debug・一時コマンドの使い分けで**柔軟な開発体験を提供**
* CLIの開発は**本番とデバッグの境界設計がカギ**

---
## 24章: CLI構成全体の評価と設計レビュー

---

### cliディレクトリ構成の最終確認

```text
cli/
├── __init__.py
├── main.py         # @click.group() 定義
├── fetch.py        # fetchコマンド
├── extract.py      # extractコマンド
├── transform.py    # transformコマンド
├── all.py          # 統合実行コマンド
├── debug.py        # 補助用・一時コマンド（任意）
```

* **1ファイル1責務**、明確な役割分担
* main.py を唯一のエントリポイントにし、`python -m cli.main` で全体を統括
* `common_options.py` のような**共通デコレータ集の分離**も構造安定に有効

---

### 保守性・再利用性・UI品質の観点からの評価

| 観点   | 設計対応例                               |
| ---- | ----------------------------------- |
| 保守性  | 共通オプション/構造化/コマンド分離/ログ出力の分離          |
| 再利用性 | main\_for\_debug・サブ関数分離・共通デコレータ導入   |
| UI品質 | `--help`の明快さ・エラーメッセージの整理・色付き出力・進捗表示 |

* clickの特性を活かし、**「CLIをUIとして捉える」設計が徹底されている**

---

### LLM/API・別プロジェクトへの転用余地

* CLI構成が疎結合・責務分離されているため、**プロジェクトをまたいだ再利用が容易**
* たとえば：

  * `fetch.py` を別API用に移植しやすい
  * `transform.py` を別形式への整形エンジンとして流用可能
  * `main.py` を別ツールの共通コマンドエントリとして流用可能

---

### 設計指針まとめ

* 本CLI構成は、**実務水準の保守性・再利用性・明瞭性を実現**
* 機能が増えても破綻せず、LLM補助による保守も容易
* 他プロジェクトやWeb連携に発展させる**土台として完成度が高い**

---
## 25章: オブジェクト指向とclickの設計原則

---

### クラスと関数の役割分担

| 構成要素        | 責務                       |
| ----------- | ------------------------ |
| **クラス**     | 実行ロジック・状態保持・処理の本体        |
| **click関数** | CLI引数の受け口・UIとしてのインターフェース |

* **クラス = 処理の本質**、**click = UIの入り口**
* ロジックはクラスに閉じ込め、click関数は**引数変換と呼び出しに特化**

---

### 最小例：CLIからクラスを呼ぶ

```python
class Greeter:
    def __init__(self, name: str):
        self.name = name

    def run(self):
        click.echo(f"Hello, {self.name}!")

@click.command()
@click.option('--name', default='World')
def greet(name):
    Greeter(name).run()
```

* click関数内では**引数を受け取ってインスタンス化 → メソッド呼び出しのみ**
* これにより：

  * ロジックの**テストが容易**
  * **非CLIからの呼び出しも可能**
  * **責務分離が明快**

---

### 利点

* クラスにまとめることで、**状態の持続や設定の注入**が容易になる
* clickの役割を「引数受け取りとUI」だけに絞れるため、**可読性と保守性が向上**
* LLM補助やコード分割時にも**迷いが生じにくい構造**

---

### poke-pipelineでの適用例（想定）

```python
class Fetcher:
    def __init__(self, endpoint: str, id_: int, overwrite: bool):
        ...

    def run(self):
        ...
```

```python
@click.command()
@click.option('--endpoint')
@click.option('--id', type=int)
@click.option('--overwrite', is_flag=True)
def fetch(endpoint, id, overwrite):
    Fetcher(endpoint, id, overwrite).run()
```

* Fetcherは**純粋な処理クラス**、fetch関数は**CLI引数の受け口**

---

### 設計指針まとめ

* **「click = UI」「クラス = ロジック」** が基本原則
* クラスを前提に設計すれば、CLI以外への転用（API / テスト / 自動化）も容易
* 次章では、この構造を前提とした **依存注入やデコレータ構成** に進む

---
## 26章: オブジェクト指向とclickの実装と注入

---

### クラスに外部設定を注入する方法

```python
from dataclasses import dataclass

@dataclass
class Config:
    endpoint: str
    lang: str
    overwrite: bool
```

* 設定を**1つのオブジェクトにまとめて**クラスへ渡すことで、**引数の増加に強い設計**が可能
* `dataclass` を使うことで初期化の省略記述と型明示も両立

---

### `click.option(..., callback=...)` による依存注入

```python
def inject_config(ctx, param, value):
    ctx.ensure_object(dict)
    ctx.obj['config'] = Config(endpoint=value, lang='ja', overwrite=False)
    return value

@click.command()
@click.option('--endpoint', callback=inject_config)
@click.pass_context
def fetch(ctx, endpoint):
    config = ctx.obj['config']
    Fetcher(config).run()
```

* `ctx.obj` を通じて**任意のクラス・設定・依存オブジェクトを伝播可能**
* 複数の `@click.option()` を組み合わせて構築した設定を**集約して渡せる**

---

### デコレータ形式で複合注入にも対応可能

```python
def inject_full_config(func):
    def wrapper(endpoint, lang, overwrite):
        config = Config(endpoint, lang, overwrite)
        return func(config)
    return click.pass_context(wrapper)
```

* あるいは Click 8 以降では `@click.pass_obj` を使うことで簡潔に取得可能

```python
@click.command()
@click.pass_obj
def fetch(config: Config):
    Fetcher(config).run()
```

---

### poke-pipelineでの応用設計

* `Config` を中間オブジェクトとして各フェッチャー・トランスフォーマーに注入
* オプション変更時も **CLI層の修正だけで済み、ロジック側は不変**
* LLM補助や設計変更にも強い「構成柔軟性」を得られる

---

### 設計指針まとめ

* クラス構成 × click連携は**責務分離と保守性向上に最適**
* `ctx.obj` や `@pass_obj` によって **注入・拡張性のある構造**が実現可能
* 全体設計に沿って **「UI層としてのclick」「ロジック層としてのクラス」** を明確に分離せよ

---
