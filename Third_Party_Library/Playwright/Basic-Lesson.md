# Pywright-Lesson

<hr>

## 1章: Playwrightとは何か

<hr>

### Playwrightの概要と特徴

| 項目       | 内容                                        |
| -- | -- |
| フレームワーク名 | Playwright（Microsoft製）                    |
| 主な用途     | E2Eテスト・UI操作の自動化・スクレイピング                   |
| 主な対応ブラウザ | Chromium（Chrome系）、Firefox、WebKit（Safari系） |
| 特徴       | マルチブラウザ対応、高速、非同期対応、強力なセレクタ、ヘッドレスモード対応     |
| 競合       | Selenium、Puppeteer                        |

### Seleniumとの違い

| 比較項目   | Playwright                | Selenium            |
|-  | - | - |
| 開発元    | Microsoft                 | Seleniumプロジェクト（OSS） |
| 対応ブラウザ | Chromium, Firefox, WebKit | 多様（IE含む）            |
| 実行速度   | 高速                        | 比較的低速               |
| API構造  | モダン（非同期／同期選択可）            | 古典的・やや複雑            |
| セレクタ指定 | 独自の `get_by_*()` APIが豊富   | CSS/xpath中心         |
| 期待値の扱い | 待機処理を組み込み済みで安定            | 明示的待機の追加が必要         |

### 主な用途

* フロントエンドアプリケーションのE2Eテスト
* 管理画面などの自動ログイン・操作確認
* JavaScriptレンダリング後のスクレイピング
* Cookieやセッションの検証・保存

### 使用言語・バインディング

| 言語         | サポート状況  |
| - | - |
| JavaScript | ◎（公式言語） |
| Python     | ◎       |
| C#         | ○       |
| Java       | ○       |

<hr>

## 2章: 環境構築



### Playwrightのインストール手順（Python）

```bash
$ pip install playwright
$ playwright install
```

| コマンド                     | 説明                                       |
|  | - |
| `pip install playwright` | Playwright本体のインストール（Pythonバインディング）       |
| `playwright install`     | ブラウザ実体（Chromium, Firefox, WebKit）のインストール |



### Playwright CLIの基本

| コマンド                    | 説明                         |
| -- | -- |
| `playwright codegen`    | 操作の録画・スクリプト自動生成ツール         |
| `playwright install`    | ブラウザのセットアップ                |
| `playwright open <url>` | URLを開いてデバッグ用ブラウザインターフェース起動 |
| `playwright --help`     | コマンド一覧の確認                  |



### `playwright codegen` の特徴

* 手動操作を記録してコードに変換（Python / JSなどを選択可）
* 自動でセレクタ推論・イベント記録
* そのままコピーしてスクリプトに組み込める



### 補足：プロジェクト構成の基本例

```
my-playwright-project/
├── tests/
│   └── test_example.py
├── requirements.txt
└── README.md
```

* 実行環境はvenv推奨
* テストランナー（pytestなど）との連携も視野に入れる構成が望ましい


## 3章: 基本的な使い方



### `sync_api` と `async_api` の違い

| 種類                   | 特徴                        | 使用例                                                 |
| -- | - |  |
| 同期API (`sync_api`)   | 書きやすい／シンプルな構文             | `from playwright.sync_api import sync_playwright`   |
| 非同期API (`async_api`) | 非同期処理に強く、複雑な操作も非同期で一括制御可能 | `from playwright.async_api import async_playwright` |

* 初学者や簡単なスクリプトでは同期APIで十分
* 複数のブラウザ操作・API連携などが絡む場合は非同期API推奨



### 最小限の同期スクリプト例

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()
```



### Headlessモードと非Headlessモード

| モード       | 説明                | 使用例                            |
|  | -- |  |
| Headless  | 画面を表示せずバックグラウンド実行 | `launch(headless=True)`（デフォルト） |
| 非Headless | ブラウザ画面を表示して動作確認可能 | `launch(headless=False)`       |

* デバッグや初期開発では `headless=False` が便利
* CIなど自動実行では `headless=True` が基本



### ページ遷移後の操作例（クリック・スクレイピング）

```python
page.click("text=ログイン")
page.fill("#username", "your_name")
page.fill("#password", "your_pass")
page.click("button[type=submit]")
```


## 4章: セレクタの指定方法



### Playwrightで使えるセレクタの種類

| 種別   | 特徴                            | 使用例                                     |
| -- | -- |  |
| CSSセレクタ  | 一般的なCSSの構文に準拠。柔軟だが待機は自前。 | `page.locator("div#main")`              |
| textセレクタ | 表示テキストに基づいて選択。i18n未対応の場面では強力。 | `page.get_by_text("ログイン")`              |
| roleセレクタ | アクセシビリティ対応。役割（role）とnameで選択。  | `page.get_by_role("button", name="送信")` |



### Playwright特有の`get_by_*()`API

| API名                   | 概要                       |
| - |  |
| `get_by_text()`        | 表示テキストから選択               |
| `get_by_role()`        | アクセシビリティrole属性とnameから選択  |
| `get_by_label()`       | `<label>` に対応するフォーム要素の取得 |
| `get_by_placeholder()` | プレースホルダーによる入力欄の選択        |



### `page.locator()` の利点

* CSS以外のセレクタも内包可能（textやroleも使える）
* **自動リトライ機構**によって、要素が出現するまで待機してくれる
* チェーン操作が可能（`.first()`, `.nth(1)`, `.click()` など）

```python
page.locator("button:has-text('送信')").click()
```



### 補足：待機つきの選択と操作

```python
submit = page.locator("button[type=submit]")
submit.wait_for()
submit.click()
```

* `locator.wait_for()` により要素が描画されるまで待機
* 明示的な `time.sleep()` より信頼性が高い


## 5章: 操作の基本



### ユーザー操作の主要メソッド

| 操作種別  | メソッド例                                  | 説明                  |
| -- | -- | - |
| クリック  | `page.click(selector)`                 | 要素をクリック             |
| 入力    | `page.fill(selector, text)`            | テキストフィールドに文字入力      |
| 選択    | `page.select_option(selector, value)`  | セレクトボックスの選択肢を指定     |
| スクロール | `locator.scroll_into_view_if_needed()` | 要素が画面外にあるとき自動でスクロール |



### スクリーンショットの取得

```python
page.screenshot(path="screenshot.png", full_page=True)
```

| 引数          | 説明                             |
| -- |  |
| `path`      | 保存先ファイル名                       |
| `full_page` | ページ全体か可視領域のみかを指定（デフォルト: False） |



### 要素の状態チェック

| メソッド                   | 説明                     |
| - | - |
| `locator.is_visible()` | 要素が可視状態かどうかを判定         |
| `locator.is_enabled()` | 有効状態か（disabledでないか）を判定 |
| `locator.count()`      | 該当する要素の個数を取得           |
| `locator.nth(i)`       | インデックス指定で要素取得（0始まり）    |

```python
if page.locator("text=ログイン").is_visible():
    page.click("text=ログイン")
```



### その他の操作例

```python
# チェックボックスON
page.check("input[type=checkbox]")

# フォーカスを移す
page.focus("#search")

# キー入力
page.keyboard.press("Enter")
```


## 6章: 待機の扱い



### 明示的待機 vs 暗黙的待機

| 種類    | 概要                             | 特徴              |
| -- |  |  |
| 明示的待機 | 開発者が待機メソッドを明示的に記述              | 高精度・制御しやすい      |
| 暗黙的待機 | Playwrightが内部的にリトライしながら待機処理を行う | コード簡潔・予期せぬ挙動の回避 |
| 固定待機  | `time.sleep()` 等を用いた待機         | 信頼性が低く非推奨       |



### 主な待機メソッド

| メソッド名                        | 用途                                        |
| - | -- |
| `locator.wait_for()`         | 要素の表示・状態変化を待つ                             |
| `page.wait_for_selector()`   | CSSセレクタに該当する要素の出現を待機                      |
| `page.wait_for_url()`        | 指定URLへの遷移完了を待機                            |
| `page.wait_for_load_state()` | ページのロード状態（`load`, `domcontentloaded`等）を待機 |

```python
page.wait_for_selector("h1.title")
```



### タイムアウト制御

| 方法                        | 説明                            |
| - | -- |
| `timeout` 引数              | 各待機APIの引数に `timeout=ms` を追加   |
| `set_default_timeout(ms)` | ページ全体のデフォルトタイムアウトを設定          |
| 例外 `TimeoutError`         | 待機に失敗すると発生する例外（try-exceptで補足） |

```python
try:
    page.wait_for_selector(".result", timeout=3000)
except TimeoutError:
    print("要素が見つかりませんでした")
```



### 補足：待機の推奨方針

* **固定待機は避ける**（例: `time.sleep(2)`）
* Playwrightは**自動的に待つ設計**になっているため、`locator`や`get_by_*`を活用することで待機を省略可能な場合も多い
* 状況によってはログ出力を併用するとデバッグが容易になる


## 7章: 複数ページ・タブ・iframeの操作



### 新しいタブ（ページ）の検出と操作

```python
with context.expect_page() as new_page_info:
    page.click("a[target=_blank]")  # 新しいタブを開く操作
new_page = new_page_info.value
new_page.goto("https://example.com")
```

| 用語              | 説明                         |
|  | -- |
| `context`       | ブラウザ内の1つのセッション（Cookie共有など） |
| `page`          | タブやウィンドウ1つを指す              |
| `expect_page()` | 新しいページ（タブ）が開かれるイベントを待つ     |



### iframeの操作

```python
frame = page.frame(name="iframe_name")
frame.locator("button#submit").click()
```

| 方法                     | 説明                             |
| - |  |
| `page.frame(name=...)` | name属性でiframeを取得               |
| `frame_locator()`      | `locator`からiframeをさらに操作できる形で取得 |



### frame\_locatorの例

```python
frame = page.frame_locator("iframe").locator("text=送信")
frame.click()
```

* iframe内部に対してもセレクタが使える
* frame名や構造に依存しないため、変更に強い



### contextの分離と用途

| 用途              | 説明                                       |
|  | - |
| セッション分離         | CookieやLocalStorageが他と独立                 |
| テストの並列実行        | 複数の `browser.new_context()` を生成して並列処理が可能 |
| ユーザー状態のシミュレーション | ログイン済み／未ログインなどの状態を複数contextで比較できる        |

```python
context = browser.new_context()
page = context.new_page()
```


## 8章: 認証とCookie管理



### ログイン状態の維持（storage\_state）

```python
# ログイン後の状態を保存
context = browser.new_context()
page = context.new_page()
page.goto("https://example.com/login")
# ... ログイン操作 ...
context.storage_state(path="state.json")
```

```python
# 保存した状態を再利用
context = browser.new_context(storage_state="state.json")
page = context.new_page()
page.goto("https://example.com/dashboard")
```

| 項目              | 説明                              |
|  | - |
| `storage_state` | Cookie・LocalStorageをJSON形式で保存可能 |
| `new_context()` | 認証済み状態を含むコンテキストを再生成するために使う      |



### Cookieの個別操作

```python
cookies = context.cookies()
context.add_cookies([{
    "name": "sessionid",
    "value": "abc123",
    "domain": "example.com",
    "path": "/",
    "httpOnly": True,
    "secure": True
}])
```

* `context.cookies()` で現在のCookie取得
* `context.add_cookies()` で明示的にCookieを挿入



### API認証トークンを使ったログインバイパス

```python
headers = {"Authorization": "Bearer <token>"}
page.goto("https://example.com/api/profile", headers=headers)
```

* REST APIと連携する場合、トークンを直接付与してログイン不要にできる
* ページ遷移前に `headers` を指定することで自動認証状態へ遷移可能



### 認証管理の活用例

| 用途                 | 手法                                        |
|  | -- |
| ユーザー認証付きのE2Eテスト    | `storage_state` を保存してテスト間で再利用             |
| 開発中にログイン操作を省略したい場合 | `context.add_cookies()` や `storage_state` |
| API検証やスモークテスト      | トークンを使って `headers` 経由で直接アクセス              |


## 9章: ネットワーク操作とモック



### APIレスポンスのモック（`route` の活用）

```python
def handle_route(route, request):
    if "api/data" in request.url:
        route.fulfill(status=200, content_type="application/json", body='{"key": "mocked"}')
    else:
        route.continue_()

page.route("**/api/data", handle_route)
page.goto("https://example.com")
```

| メソッド                | 説明                                |
| - |  |
| `page.route()`      | ネットワークリクエストをフックし、任意のレスポンスに書き換えられる |
| `route.fulfill()`   | 自作のレスポンスで応答する                     |
| `route.continue_()` | リクエストをそのまま進行させる                   |



### リクエスト／レスポンスのログ取得

```python
page.on("request", lambda request: print(">>", request.method, request.url))
page.on("response", lambda response: print("<<", response.status, response.url))
```

* `request.method` や `request.post_data()` で詳細確認可能
* `response.json()` でレスポンス内容の解析も可能（非同期）



### オフラインモードのシミュレーション

```python
context = browser.new_context()
context.set_offline(True)
page = context.new_page()
page.goto("https://example.com")
```

| 状態                  | 説明                       |
| - |  |
| `set_offline(True)` | ネットワーク接続を遮断状態に設定         |
| 使用例                 | オフライン時のエラーハンドリングテストなどに有効 |



### モックの活用用途

| 用途                 | 内容                           |
|  | - |
| 開発中のバックエンド未実装API対応 | モックで代替レスポンス返却                |
| 異常系の動作確認           | タイムアウトや500エラーなどを擬似的に再現可能     |
| E2Eテストの安定化         | テスト中の外部依存性を排除し、確実な結果取得が可能になる |


## 10章: スクレイピング応用



### テーブルデータの抽出例

```python
rows = page.locator("table tr")
for i in range(rows.count()):
    cells = rows.nth(i).locator("td")
    data = [cells.nth(j).inner_text() for j in range(cells.count())]
    print(data)
```

| 処理内容           | 説明                |
| -- | -- |
| `locator()`    | セレクタに一致する要素群を取得   |
| `count()`      | 一致した要素の数を取得       |
| `nth(i)`       | i番目の要素を取り出す（0始まり） |
| `inner_text()` | HTML内のテキスト内容を抽出   |



### 動的ロードされた要素への対応

```python
page.wait_for_selector("div.result")
results = page.locator("div.result")
```

* `wait_for_selector()` で要素が表示されるまで待機
* JavaScriptによる後読み込みにも確実に対応可能
* 明示的待機よりもPlaywrightの自動待機を活用する方が堅牢



### スクロールが必要な場面への対応

```python
while True:
    if page.locator("button.load-more").is_visible():
        page.click("button.load-more")
        page.wait_for_timeout(1000)
    else:
        break
```

* ページネーションや「もっと見る」ボタンも自動クリックで収集可能
* `scroll_into_view_if_needed()` を併用すると視認性が向上



### BeautifulSoupとの連携は非推奨

| 理由                     | 説明                                  |
| - | -- |
| JSレンダリング後のDOM構造が動的     | `page.content()` の結果が完全でない可能性がある    |
| Playwright自体に要素取得機能がある | `locator().inner_text()` などで十分かつ高機能 |
| 処理速度や堅牢性が落ちる可能性        | HTML文字列からの解析より、DOM操作ベースの方が信頼性が高い    |



### 補足：スクレイピング設計のポイント

* 取得対象を明確にし、セレクタは具体的に指定
* 冗長なsleepより、`wait_for_*`を積極活用
* JSONやCSVへの構造化保存も併用すると後工程が楽になる


## 11章: E2EテストとしてのPlaywright



### テストランナーの導入（pytest連携）

```bash
$ pip install pytest-playwright
```

```python
# tests/test_login.py
def test_login(page):
    page.goto("https://example.com/login")
    page.fill("#username", "user")
    page.fill("#password", "pass")
    page.click("text=ログイン")
    assert page.url == "https://example.com/dashboard"
```

| ツール                 | 説明                                     |
| - | -- |
| `pytest-playwright` | `page`フィクスチャを提供、Playwrightを簡単にテストに統合可能 |
| `pytest`            | Python定番のテストランナー                       |



### テスト用アカウントの操作検証例

* ダミーアカウントを使ったログイン検証
* 認証状態の維持（`storage_state.json`の再利用）
* 投稿→削除→確認といった一連の操作を1テストで完結

```python
def test_post_article(page):
    page.goto("https://example.com")
    page.click("text=記事投稿")
    page.fill("#title", "Playwrightとは")
    page.click("text=送信")
    assert page.locator("h1").inner_text() == "Playwrightとは"
```



### スクリーンショットと動画の自動記録

```bash
$ pytest --screenshot=on --video=on
```

| オプション             | 効果                             |
| -- |  |
| `--screenshot=on` | 失敗時に自動でスクリーンショットを保存            |
| `--video=on`      | テスト中の操作を動画として記録                |
| 保存先               | `test-results/` ディレクトリに自動保存される |



### テスト自動化のベストプラクティス

* **データを毎回初期化**（例：投稿の削除 → 再投稿）
* **ログイン状態を共有しない**（`storage_state`は読み取り専用）
* **UI要素の文言に依存しすぎない**（roleセレクタやid活用）
* **タイムアウトを明示的に設定**し、不安定さを回避


## 12章: CIとの統合（GitHub Actions等）



### GitHub Actionsでの基本構成例

```yaml
# .github/workflows/playwright.yml
name: Run Playwright Tests

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          playwright install

      - name: Run tests
        run: pytest --screenshot=on --video=on
```



### Headless実行とXvfb（GUI不要）

| 実行方式        | 説明                               |
| -- | -- |
| headlessモード | デフォルトで有効。GUI描画せず高速に実行            |
| xvfb環境      | GUI描画が必要な場合、仮想ディスプレイで対応（Linux限定） |

```bash
# xvfb使用の例（GitHub Actionsでは通常不要）
xvfb-run -a pytest
```



### Linux環境でのトラブル対処

| 問題           | 対応策                                      |
|  | - |
| `GLIBC`関連エラー | Ubuntuのバージョンを指定する（例: `ubuntu-22.04`）     |
| ブラウザが起動しない   | `playwright install` を忘れず実行              |
| WebKitが落ちる   | `apt install libwebkit2gtk-4.0-37` などを追加 |



### Playwright Test Reportの保存

```yaml
      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: playwright-results
          path: test-results/
```

| 用途         | 内容                       |
| - |  |
| アーティファクト保存 | スクリーンショット・動画・ログファイルなどの保存 |
| CIの可視化     | 実行環境で再現できない問題の原因特定に役立つ   |



### CI連携のポイント

* `pytest-playwright` の導入で**簡易な統合**が可能
* **Headlessモード**での実行を前提としたコードにしておく
* スクリーンショットやログの保存により**デバッグ性を担保**する


## 第13章：PlaywrightとCSR（Client Side Rendering）



### CSRにおけるJS描画完了の検出方法

CSR（Client Side Rendering）では、HTMLの初期ロード直後はほとんど何も描画されておらず、JavaScriptが後からDOMを構築する。そのため、Playwrightでの自動操作・スクレイピングでは「ページ読み込みが完了した」ことを正しく検出するために、単なる `page.goto()` や `page.wait_for_load_state("load")` だけでは不十分である。

以下の手法を組み合わせて、**JavaScriptによって生成される要素が確実に表示された状態**を検出する：

* `page.wait_for_selector("css=...")`
  特定の要素がDOM上に現れるまで待機。存在確認と同時に表示状態も検証できる。

* `locator.wait_for()`
  Locator APIを使うことで、複数条件（visible、attached、detachedなど）を柔軟に設定可能。

* `page.wait_for_load_state("networkidle")`
  ページ内のリソース取得が完全に停止した状態を待機するが、JS描画の保証はないため他と併用すべき。

* `page.evaluate("document.readyState")` での `complete` チェック
  初期HTMLの読み込みが完了しただけで、CSRでは描画完了を示さない。**使用非推奨**。



### `wait_for_*`と`load_state`の適切な使い分け

CSRにおける待機処理の選定は、検出したい対象に応じて次のように整理できる：

| 待機手法                            | 用途と注意点                                                                                 |
| - | -- |
| `page.wait_for_load_state()`    | ページ全体の読み込み状態（`load` / `domcontentloaded` / `networkidle`）を確認。<br>**CSRでは単体で使用すると不十分**。 |
| `page.wait_for_selector("...")` | 対象となる要素の存在を明示的に確認。**最も確実な方法**。タイムアウト設定も可能。                                             |
| `locator.wait_for(...)`         | Locator APIで取得した要素に対して柔軟に待機条件を指定できる（表示状態など）。                                           |

一般的には次の戦略が安全：

* **特定要素を待機** → `page.wait_for_selector()` や `locator.wait_for()`
* **全体的な読み込みを検出** → `page.wait_for_load_state("networkidle")`（ただし単独では不十分）



### CSRサイトのテスト・スクレイピング戦略

CSRでは初期HTMLが空に近いため、次のような対応が重要になる：

#### ✔ 要素ベースの安定した待機

* DOMの特定要素（例：`div.content`, `.card`, `[data-testid="loaded"]` など）をターゲットにする
* `page.wait_for_selector()` は**存在 + 表示**を同時に確認できる

#### ✔ セレクタの戦略的設計

* `get_by_role()`, `get_by_text()` などPlaywright独自のセマンティックセレクタで安定性向上
* CSS構造の変化に強く、**テストやスクレイピングの保守性**が上がる
* `page.locator("...").nth(i)` で動的リストへのアクセスにも対応

#### ✔ タイミングの確定

* DOM生成の**タイミングのぶれ**がある場合、明示的な `time.sleep()` は避け、**状態を確認する形**で待機する
* 条件付きループで `locator.count()` を見ながら要素出現を待つことも有効

#### ✔ Headless時の描画差異への対処

* Headlessではブラウザの描画タイミングが異なるため、**非Headlessでのデバッグが有効**
* `slow_mo` オプションを付けて描画を可視化しながら調整可能

```python
browser = playwright.chromium.launch(headless=False, slow_mo=100)
```

#### ✔ スクレイピングにおける注意点

* 初期読み込みでデータが見えない場合、スクレイピング前に**必要な要素を待つ処理を入れる**
* CSRで非同期ロードされるデータは、**XHR/Fetchの完了を確認**するか、**DOMを監視**して処理する


## 第15章：PlaywrightとSSG（Static Site Generation）



### ビルド後の静的ファイルへのアクセス

SSG（Static Site Generation）はビルド時にすべてのHTMLを生成するため、アクセス時にはサーバ側の動的処理が不要で、常に同一のHTMLレスポンスが返る。これにより、Playwrightを用いたテストでは**毎回同じ状態が得られる**ため、検証の再現性が非常に高い。

* ローカルで `npx serve out/` や `python -m http.server` を使い、HTTPサーバを立てて検証
* `page.goto("http://localhost:3000/")` の直後に `page.content()` を呼んでも完全なHTMLを取得できる
* `wait_for_load_state("load")` や `networkidle` は多くの場合不要。SSR同様、JS実行後の検証は補足として扱う
* PlaywrightはSSGとの相性が良く、デプロイ前の静的チェックに向く



### Playwrightによるリンク切れチェック・アクセシビリティ検証

SSGではリンク切れがビルド後に混入しても気付きにくいため、Playwrightで網羅的に確認することでサイト全体の健全性を保てる。

* `page.locator("a")` で全リンクを抽出し、`href` 属性を取得してHTTPリクエストで到達性を検査
* `.get_attribute("aria-label")` などアクセシビリティ属性を収集し、空欄や欠落を検出可能
* ページごとにスクリーンショットを撮り、ビジュアル異常も検知できる
* `page.on("response")` イベントを使い、404 / 500 エラーを含むレスポンス状況を収集

例：

```python
urls = await page.locator("a").evaluate_all("els => els.map(el => el.href)")
for url in urls:
    resp = await page.request.get(url)
    assert resp.status == 200
```



### 生成されたHTMLの構造検査とキャッシュ確認

静的に生成されたHTML構造を**定量的・定性的に確認**することは、品質保証の一環として重要。

* DOM構造を `locator.count()` や `inner_text()` で検査
* `assert "404 Not Found" not in page.content()` で意図しない文言の出現を監視
* `meta[name='robots']`, `link[rel='canonical']` などSEO要素の存在確認も自動化可能
* キャッシュやCDN経由での表示崩れを疑う場合、2回アクセスし `page.content()` を比較

```python
first = await page.content()
await page.reload()
second = await page.content()
assert first == second
```

* さらに、`expect_screenshot()` を用いて前回との差分検出も可能（GitHub Actionsとの組合せに有効）



## 第16章：PlaywrightとISR（Incremental Static Regeneration）



### データ差し替え検出とキャッシュ更新の可視化

ISR（Incremental Static Regeneration）は、事前に静的生成されたHTMLを一定期間キャッシュし、その後バックグラウンドで新しいHTMLに更新する仕組みである。ユーザーから見れば、初回は旧HTMLが表示され、再アクセス時に新しいデータが反映されるという挙動をとる。Playwrightはこの**キャッシュの入れ替わり**や**非同期的なDOM差分**を検出するのに適している。

* 初回 `page.goto()` 時に `page.content()` を保存
* ISR対象のデータ（例：記事内容、更新日時など）をトリガーして変更
* 再度同一ページにアクセスし、差し替えが発生しているかを比較

検証対象は以下のような項目：

* `.updated-at` などの更新時刻のテキスト
* 記事タイトルや内容のテキスト差分
* DOM構造そのものの変更（要素の追加・削除）

```python
old_text = await page.locator(".updated-at").text_content()
await page.reload()
new_text = await page.locator(".updated-at").text_content()
assert old_text != new_text
```

`page.screenshot()` を併用すれば、**キャッシュ切り替え前後の視覚的差異**を画像として確認できる。



### ページロード直後と再リクエスト後の差分比較

ISRは最初のアクセスでキャッシュを返し、裏で新しいHTMLを再生成するため、2回目以降のアクセスで差分が現れる。これをPlaywrightで再現・検証するには、2段階の比較が必要：

1. **初回取得：** `page.goto()` → `.content()` or `.text_content()` を記録
2. **更新トリガー後：** サーバ側のデータを変更（例：DB更新やCMS編集）
3. **再アクセス：** `page.reload()` → 同じ場所のデータを再取得
4. **差分比較：** 前後でテキストやDOMの差異を `assert` で確認

差分を検出することで、ISRの非同期再生成が正常に機能しているかを検証できる。差分検出に失敗した場合、ISRのビルドタイミングやトリガーの問題を示している可能性がある。

また、Playwrightの `expect_*` シリーズを使えば、**一定時間内に変化が起きること**を保証できる：

```python
await expect(page.locator(".updated-at")).not_to_have_text(old_text, timeout=10000)
```



### `wait_for_response()`による非同期再取得の検証

ISRでは、HTMLは静的に返されるが、裏で再生成を行うAPIが非同期で呼び出されている。これをPlaywrightで検出するには、`page.expect_response()` を使って**ISR再生成用のAPIエンドポイント**をモニタリングする。

```python
async with page.expect_response("**/__prerender*") as resp_info:
    await page.reload()
response = await resp_info.value
assert response.status == 200
```

このようなエンドポイントは Next.js で自動生成されており、ISR更新時には必ず呼び出される。これを捉えることで、バックグラウンド再生成が行われたことを検証できる。

また、`page.on("response")` を使えば、すべてのレスポンスをモニタリングし、特定のステータスコード（例：202 Accepted）や遅延応答を検出可能。



### その他の検証ポイント

* **タイムスタンプの一貫性検証：** 同じページを複数ブラウザで開いたとき、ISR更新が共有されているかを比較
* **ステールデータ検出：** 更新されるはずの要素が変わらないケースを検知し、再生成漏れを発見
* **CIとの統合：** ISRの再生成確認をCIパイプラインに組み込み、GitHub Actionsなどで自動監視



## 第17章：Playwrightとレンダリング遅延の分析



### `performance.timing`の取得と解析

Webページの表示における遅延の正確な把握には、ブラウザ内部のナビゲーションタイミングAPIを利用する。Playwrightは `page.evaluate()` を使って `performance.timing` や `performance.getEntriesByType("navigation")` を取得できるため、以下のような指標を定量的に収集・解析可能である。

* **navigationStart**：ページ遷移の開始
* **domContentLoadedEventEnd**：DOM構築完了
* **loadEventEnd**：全リソースの読み込み完了
* **firstPaint / firstContentfulPaint**：初描画／視覚コンテンツ描画のタイミング（Chromium系）

例：

```python
timing = await page.evaluate("JSON.stringify(performance.timing)")
timing_data = json.loads(timing)
dom_load = timing_data["domContentLoadedEventEnd"] - timing_data["navigationStart"]
```

こうした値をログとして蓄積すれば、デプロイ間でのパフォーマンス劣化を検知できる。



### ファーストビューとインタラクティブの定量比較

ユーザー体験の観点では「ページが表示された」だけでなく、「操作可能になった」タイミングが重要である。これを評価するには以下の複数指標を組み合わせる：

* **FCP（First Contentful Paint）**：最初のテキスト・画像などの描画時刻
* **LCP（Largest Contentful Paint）**：最大要素（例：ヒーロー画像や見出し）の描画時刻
* **TTI（Time To Interactive）**：全JS実行後に操作可能となる時刻

Chromiumベースのブラウザであれば、以下のように取得可能：

```python
metrics = await page.evaluate("""
() => {
  const [entry] = performance.getEntriesByType("navigation");
  return {
    fcp: performance.getEntriesByName("first-contentful-paint")[0]?.startTime,
    lcp: performance.getEntriesByType("largest-contentful-paint")[0]?.startTime,
    tti: entry.domInteractive
  };
}
""")
```

これにより、単なる読み込み完了ではなく「見える」「使える」の両面を定量的に計測できる。



### スクリーンショット記録との連携

レンダリング遅延を**視覚的に把握**するために、タイミングごとのスクリーンショットを取得して記録する手法が有効である。以下のように、ページ遷移後に一定間隔で `page.screenshot()` を呼び出し、描画の進行を可視化する：

```python
await page.goto(url)
await page.screenshot(path="step1_initial.png")
await page.wait_for_timeout(1000)
await page.screenshot(path="step2_midload.png")
await page.wait_for_load_state("networkidle")
await page.screenshot(path="step3_complete.png")
```

この連続画像により、FCP/LCP/TTIが体感的に確認でき、**非エンジニアへの説明資料**としても利用可能である。

また、CI環境ではスクリーンショットをHTMLレポートに自動埋め込みすることで、パフォーマンスの回帰を早期に検出できる。



### 開発・検証時の補助手法

* `slowMo` オプションを有効にしてステップごとの描画を目視確認

  ```python
  browser = await playwright.chromium.launch(headless=False, slow_mo=200)
  ```
* ネットワークを意図的に遅延させて実環境に近い挙動を再現（`page.route` + `delay`）
* LighthouseやWeb Vitalsとの併用でPlaywrightスクリプトの数値結果を検証・補完

レンダリング遅延の分析は、表示速度や体感品質の改善に直結する。Playwrightを用いれば、**ブラウザネイティブの指標を自動収集し、視覚的に補強するテスト**を構築できるため、プロダクトのUX向上やSEO改善に寄与するデータを継続的に取得可能となる。特にCSRやハイブリッドなページ構成では、遅延の影響が顕著であり、この章の分析は必須である。



## 第18章：レンダリング方式別の自動テスト設計指針



### CSR / SSR / SSG / ISRごとのE2Eテスト設計方針

現代的なWebアプリケーションでは、レンダリング方式（CSR / SSR / SSG / ISR）ごとにDOM構築や描画タイミングが大きく異なる。Playwrightを用いたE2Eテストでは、それぞれの方式に合わせた**待機・検証・構造化**が必要になる。以下は各方式における典型的な戦略である：

以下は、第18章における **レンダリング方式別のE2Eテスト設計指針**を、テーブルではなく**リスト形式**で再掲示したものです。



### CSR（Client Side Rendering）

* **初期HTML**：空。JSが描画を全て担当
* **セレクタの選び方**：

  * `page.get_by_role()`
  * `page.locator("css=...")`
* **待機方法**：

  * `locator.wait_for()`
  * `page.wait_for_selector()`
* **API監視**：

  * DOM構築がJS依存なので、XHR・fetch完了の監視が有効
* **テスト安定性**：低（JS完了タイミングに依存）



### SSR（Server Side Rendering）

* **初期HTML**：主要な構造・データが含まれている
* **セレクタの選び方**：

  * `page.title()`
  * `page.locator("text=...")`
* **待機方法**：

  * `page.wait_for_load_state("load")` だけでほぼ十分
* **API監視**：不要
* **テスト安定性**：高（HTMLが即座に安定）



### SSG（Static Site Generation）

* **初期HTML**：完全に静的で、ビルド時に生成済み
* **セレクタの選び方**：

  * `page.locator("meta")`
  * `page.locator("h1")`
  * `page.locator("link[rel='canonical']")`
* **待機方法**：基本不要

  * `page.content()` 取得のみで検証可能
* **API監視**：不要
* **テスト安定性**：非常に高（毎回同一のDOM構造）



### ISR（Incremental Static Regeneration）

* **初期HTML**：旧データ。バックグラウンドで再生成
* **セレクタの選び方**：

  * `.updated-at` のように明確な更新対象を指定
* **待機方法**：

  * `page.reload()` 後、`expect(locator).not_to_have_text(...)`
  * 差分を明示的に検証
* **API監視**：

  * `page.wait_for_response("**/__prerender*")` を使ってISR更新検出
* **テスト安定性**：中（非同期更新のため揺らぎがある）

それぞれの方式ごとに、**検証の焦点**と**リスク要因**が異なるため、テストスクリプトの設計を使い回さず、方式ごとに分岐・切替ができるよう構成すべきである。



### 各方式に最適なセレクタ・待機・確認手法




### CSR（Client Side Rendering）

* **セレクタ**

  * `page.get_by_role()`
  * `page.locator("css=...")`
* **待機手法**

  * `locator.wait_for()`
  * `page.wait_for_selector()`
* **API監視**

  * JavaScriptによるDOM構築が完了するまでのXHRやfetch完了を監視
* **テスト安定性**

  * 低め（JavaScriptの読み込み・実行順序に依存し、タイミングのズレが起こりやすい）



### SSR（Server Side Rendering）

* **セレクタ**

  * `page.locator("text=...")`
  * `page.title()`
* **待機手法**

  * `page.wait_for_load_state("load")` のみで安定検証可能
* **API監視**

  * 通常は不要（HTMLがサーバから完全に提供されるため）
* **テスト安定性**

  * 高い（初期DOMが完全に整っているため検証しやすい）



### SSG（Static Site Generation）

* **セレクタ**

  * `page.locator("meta")`
  * `page.locator("h1")`
  * `page.locator("link[rel='canonical']")`
* **待機手法**

  * 不要（`page.goto()` 直後に `page.content()` で即検証可能）
* **API監視**

  * 通常不要（全てビルド時に固定化）
* **テスト安定性**

  * 非常に高い（毎回同一の静的構造で変化がない）



### ISR（Incremental Static Regeneration）

* **セレクタ**

  * 差分対象の要素に `.id`, `.class` を付与し明示的に指定（例：`.updated-at`）
* **待機手法**

  * `page.reload()` の後に
    `expect(locator).not_to_have_text("旧値")` などで差分検出
* **API監視**

  * `page.wait_for_response("**/__prerender*")` を使用してISRの再生成トリガーを監視
* **テスト安定性**

  * 中（非同期でキャッシュ更新されるため、状態によってテスト結果に揺らぎが生じる）



### テスト構成における動的切替と分岐管理

複数のレンダリング方式が混在するプロジェクトでは、ページごとにレンダリング戦略が異なる可能性がある。そのため、テスト設計時には以下のような構成を推奨する：

1. 各ページまたは機能ごとに「レンダリング種別」のメタ情報を付与（YAML/JSONなど）
2. テスト時に `type=csr|ssr|ssg|isr` を参照し、適切な処理分岐を行う
3. 共通化可能なアサーションやヘルパー関数は `utils/render_helpers.py` 等にまとめて再利用性を高める

例：

```python
if render_type == "csr":
    await page.wait_for_selector(".main-card")
elif render_type == "ssg":
    assert "サイトへようこそ" in await page.content()
```

このような分岐設計により、**単一のテストフレームワーク内で多様なページ構造に柔軟に対応**できる。



### テスト効果の最大化に向けたヒント

* CSR対象では `setTimeout`, `requestAnimationFrame` の影響も想定し、**明示的に待機**
* ISR対象では **連続テスト間に差が生じうる**ため、**状態キャッシュの破棄**（例：`revalidate`）も考慮
* SSG対象ではスナップショットテストやリグレッションチェックが有効。GitHub Actionsとの統合に最適
* SSR対象ではSEO要素・OGP要素の自動抽出と検証が品質管理に直結



### サンプル構成例（ファイル単位分割）

```
tests/
├─ pages/
│  ├─ test_home_ssg.py
│  ├─ test_blog_ssr.py
│  ├─ test_dashboard_csr.py
│  └─ test_article_isr.py
├─ utils/
│  └─ render_helpers.py
└─ data/
   └─ render_types.json
```

* 各テストモジュールは対応方式に合わせた構造・待機戦略を採用
* `render_helpers.py` には `wait_for_ready()`, `assert_dom_stable()` などを実装し共通利用


レンダリング方式の違いは、E2Eテストの信頼性と効率に直結する。方式を無視した画一的なテストは誤検出や検出漏れの原因となるため、**構造を理解し、方式ごとに最適化されたテスト戦略を設計すること**が、モダンWeb開発のE2Eにおける基本方針である。Playwrightはこれを支える柔軟性と精密な制御力を備えており、開発・CI運用の両面で強力な武器となる。



## 第19章：スクレイピング戦略の分類と選択指針



### 静的HTML vs 動的DOM の判別方法

スクレイピングでは、対象サイトが**静的HTMLか動的DOMか**を早期に見極めることが成功の鍵となる。以下のような観察と技術を用いて分類を行う：

* **ページソース（Ctrl+U）で目的のデータが見えるか？**
  → 見えるなら静的HTML（SSGやSSR）
  → 見えない場合、CSRやJS動的描画の可能性が高い

* **開発者ツールでNetworkタブのXHRを観察**
  → データがAPI経由で取得されている場合、JSによる後処理が入る

* **初期表示が早く、操作によって内容が変わるか**
  → 無限スクロール・タブ切り替えなどがある場合、動的レンダリング要素あり

* **Playwrightで `page.content()` を取得し、データが含まれているか確認**
  → データがあれば静的／SSR、なければ動的描画対象

この初期分類により、以後の**セレクタ設計・待機戦略・取得方法**を大きく変えることができる。



### Playwrightを選ぶべきケースと非推奨ケース


#### Playwrightが有効なケース（選ぶべき）

* JSによって構築されたDOMを扱う必要がある（例：SPAやCSRページ）
* クリック・スクロール・フィルタ選択など**操作を伴うフロー**を含む
* アニメーション・インタラクションが多く、**ページ遷移が複雑**
* 一覧 → 詳細ページ → 戻る、のような**ステップ付きスクレイピング**
* スクリーンショットやログイン状態の保持が必要
* DOMが毎回変化するが、ロールやテキストで安定的に指定できる

#### 非推奨なケース（避けるべき）

* 完全に静的なHTMLを定期取得するだけ（`requests` + `BeautifulSoup`で十分）
* スケーラブルに高速取得したい（Playwrightは重く、多並列に向かない）
* 大量ページのクロール（並列化する場合、別プロセス管理やCloud処理が必要）
* APIが公開されており、JSONで同等の情報が得られる場合（直接叩くべき）



### 複数ページ／IDリスト型／無限スクロール型の分類

#### 1. 複数ページ型（ページネーションあり）

* URL末尾が `?page=2`, `?offset=20` などで切り替わる
* `next` ボタンや `ul.pagination` などが存在
* Playwrightでは `while True:` ループで `next` が出現しなくなるまでクリック

```python
while await page.locator(".next-button").is_visible():
    await page.click(".next-button")
    await page.wait_for_selector(".item")
```

#### 2. IDリスト型（一覧から個別詳細へ）

* 一覧ページに複数のリンクやボタンがあり、個別ページに遷移して詳細情報を抽出
* 各アイテムのリンクを `.get_attribute("href")` で事前取得してループ処理
* Playwrightでは `context.expect_page()` を使って**新タブ開閉も処理可能**

```python
for link in links:
    async with page.expect_navigation():
        await page.goto(link)
    extract_data()
```

#### 3. 無限スクロール型（lazy loading）

* `スクロール` に応じてコンテンツが非同期に追加される
* Playwrightでは `evaluate("window.scrollBy()")` を使って自動スクロール
* DOMの `count()` が増えなくなった時点で完了と判断

```python
prev = 0
while True:
    await page.evaluate("window.scrollBy(0, document.body.scrollHeight)")
    await page.wait_for_timeout(1000)
    curr = await page.locator(".card").count()
    if curr == prev:
        break
    prev = curr
```



### スクレイピング戦略決定のフロー例

1. **静的 or 動的** → Playwrightが必要か判定
2. **全件取得 or 一部抽出** → スケーリング方針を検討
3. **ページ数・構造の変化の有無** → セレクタ戦略を調整
4. **ビジュアル情報・スクリーンショットが必要か** → Playwright導入価値を再検討
5. **セッション・Cookie管理が必要か** → `storage_state` 設定や `context` 管理を検討



### 補足：安定した運用のための技術的配慮

* `try-except` で各スクレイピング単位を囲み、**失敗しても次へ進む**
* セレクタ指定は `data-testid`, `role`, `aria-label` を優先（CSS構造変化に強い）
* スクレイピング対象が明示的にDOM上にあるか `locator.wait_for()` で確認
* スクレイピング結果は JSON/CSV/SQLite に保存し、**再実行時の再取得を回避**


スクレイピングにおける戦略選定は、単に「動的か静的か」の二択ではなく、「どの方式なら再現性・保守性・コストのバランスが最適か」を冷静に判断する必要がある。Playwrightはその中でも「複雑な挙動と再現性の担保」が必要な場面において最も強力な選択肢となる。



## 第20章：リスト型ページの走査とデータ抽出



### 一覧ページに含まれるアイテムのループ処理

リスト型ページは、同一レイアウトで複数のアイテム（商品、記事、ユーザーなど）が繰り返し表示される構造を持つ。これらをPlaywrightで処理するには、以下のステップで対応するのが基本となる：

1. `.locator(".item")` などでリスト項目のセレクタを指定
2. `.count()` で項目数を取得し、ループ処理に渡す
3. `.nth(i)` を使って個別要素に順次アクセスし、属性やテキストを抽出

```python
items = page.locator(".item")
count = await items.count()
for i in range(count):
    title = await items.nth(i).locator(".title").text_content()
    price = await items.nth(i).locator(".price").text_content()
    print(f"{title} - {price}")
```

Playwrightの`locator`は遅延評価されるため、ループ中にDOMが変わっても都度再評価される点が利点である。



### 各詳細ページへの遷移と戻り処理

一覧から詳細への遷移は、スクレイピング対象が深い構造を持つ場合に必要となる。以下のように**詳細ページに遷移→抽出→戻る**というサイクルで走査を行う：

#### 方法1：同一タブ内遷移＋戻る

```python
links = await page.locator(".item a").evaluate_all("els => els.map(el => el.href)")
for url in links:
    await page.goto(url)
    await page.wait_for_selector(".detail")
    extract_detail()
    await page.go_back()
```

#### 方法2：新規タブ生成による分離実行

```python
links = [...]
for url in links:
    new_page = await context.new_page()
    await new_page.goto(url)
    await new_page.wait_for_selector(".detail")
    extract_detail(new_page)
    await new_page.close()
```

* 新規タブ方式は元の一覧ページを保持できるため、**状態管理が容易**
* タブ切替は `context.expect_page()` を使えばポップアップにも対応可能



### ページネーションと `has_next` 判定の自動化

リスト型ページの走査では、複数ページにまたがるデータを自動的にクロールする必要がある。そのためには、**ページ送りボタンの存在検知とクリック制御**が必要となる。

#### 明示的な「次へ」ボタン検出

```python
while await page.locator(".next-button").is_visible():
    await process_items()
    await page.click(".next-button")
    await page.wait_for_load_state("networkidle")
```

#### 非表示・無効状態での終了判定

* ボタンの`disabled`属性を確認することで次がないことを判定可能
* `locator.get_attribute("aria-disabled") == "true"` のような方法も有効



### 非同期読み込みやローディングの対応

近年のリスト型ページでは、ページ遷移ではなく「もっと見る」ボタンやスクロールによる**動的読み込み**が採用されているケースも多い。この場合は、待機処理を工夫する：

```python
prev = 0
while True:
    await page.click(".load-more")
    await page.wait_for_timeout(1000)
    curr = await page.locator(".item").count()
    if curr == prev:
        break
    prev = curr
```

* ローディングスピナーがある場合は `.wait_for_selector(".spinner", state="hidden")` で制御
* アイテム数の変化を追うことは信頼性の高い手法である



### 抽出対象が複数属性を持つ場合の構造整理

リスト内の各アイテムが複数の情報（例：名前、価格、説明、画像URLなど）を持つ場合、辞書型で整理すると後工程での処理が楽になる：

```python
products = []
for i in range(count):
    el = items.nth(i)
    data = {
        "name": await el.locator(".name").text_content(),
        "price": await el.locator(".price").text_content(),
        "url": await el.locator("a").get_attribute("href"),
        "img": await el.locator("img").get_attribute("src"),
    }
    products.append(data)
```

* この形式はJSON保存、DB登録、CSV変換などあらゆる形式に容易に展開可能



### エラー耐性とログ記録の実装

リスト走査において途中でDOM構造が変化したり、要素が欠落していたりすることは珍しくない。以下のような冗長処理が堅牢性を高める：

* 各アイテムの処理を `try-except` で囲む
* `timeout=5000` などタイムアウトを明示的に設定し、フリーズを防止
* ログ出力をファイル保存して進捗記録（`logging`, `datetime`, `uuid`の活用）


リスト型の走査と詳細抽出は、**すべてのWebスクレイピングの基本動作**であり、動的・静的を問わず頻出する構造である。Playwrightはセレクタの柔軟さとステップ操作の強さにより、**複雑な遷移を含む多段階構造にも対応できる**。この章の戦略を軸に、他の構造への応用（無限スクロール、ID分解、フィルター操作など）も容易に展開できる。




## 第21章：動的ロード・無限スクロール型の対処



### ボタン型読み込み (`もっと見る`) への自動対応

多くの動的ロード型ページでは、「もっと見る」や「Load More」ボタンをクリックすることで、新しい要素が非同期に追加される。これらは以下のようにPlaywrightで処理する：

1. `.is_visible()` でボタンの表示状態を確認
2. `.click()` で押下
3. `.wait_for_selector()` または `.wait_for_timeout()` で新規要素の出現を待機
4. 新規アイテムが増えなくなったら終了

```python
while await page.locator(".load-more").is_visible():
    prev_count = await page.locator(".item").count()
    await page.click(".load-more")
    await page.wait_for_timeout(1000)
    curr_count = await page.locator(".item").count()
    if curr_count == prev_count:
        break
```

ボタンが非表示になった、あるいはアイテム数が変化しないといった**変化の停止**を終了条件とするのが基本。



### スクロールイベントの自動化（`evaluate` による `scrollBy` 等）

「ページ最下部までスクロールすると自動で読み込まれる」形式の無限スクロールでは、`page.evaluate()` で `window.scrollBy()` を繰り返し実行し、アイテムが増えるたびに待機・再スクロールを行う。

```python
prev = 0
while True:
    await page.evaluate("window.scrollBy(0, document.body.scrollHeight)")
    await page.wait_for_timeout(1000)
    curr = await page.locator(".item").count()
    if curr == prev:
        break
    prev = curr
```

* `wait_for_timeout()` はネットワーク状況に応じて調整（500〜1500ms程度）
* `.count()` の変化量が閾値未満ならループを打ち切る設計も可能



### DOM変化の検知としきい値の設計

DOM変化を検知するためには、次のような条件を組み合わせると安定する：

* **要素数が変化したか**（`.count()` 比較）
* **特定要素がDOMに追加されたか**（`.nth(last).is_visible()`）
* **レスポンスが新たに到着したか**（`page.on("response")`）
* **スピナーやローダーが非表示になったか**（`state="hidden"`）

```python
await page.wait_for_selector(".spinner", state="hidden")
```

しきい値の設計例：

* 直近3回のスクロールで `.item.count()` が増加しなかったら終了
* 最大100回までスクロールを許容し、それ以上は強制終了（無限ループ対策）



### パフォーマンスと検出精度の両立

無限スクロールでは、**高速にスクロールしすぎると描画に追いつかずアイテムが見逃される**ため、適度なウェイトが必要。対策として：

* `scrollBy` を1回の大きな距離ではなく、複数回の小ステップで実行
* `await page.wait_for_function("document.readyState === 'complete'")` を挟んで描画完了を確認
* `page.wait_for_response("...")` で新たなデータ取得が完了してから次のスクロールに進む

```python
await page.wait_for_response(lambda resp: "/api/list" in resp.url and resp.status == 200)
```



### 非同期要素に対するセレクタの工夫

* `.item` よりも `data-testid` や `role="listitem"` など構造に準拠したセレクタが安定
* 親要素の `.list` → 子要素 `.list-item` のように階層構造で指定することで、**意図しない要素混入を防ぐ**
* 一度取得したアイテムのURLやIDをset()で保持し、**重複検出・無限ループ回避**に活用できる



### スクレイピング設計上の留意点

* 無限スクロールは**仕様変更やDOM構造の崩壊に弱い**ため、ログ記録とエラーリカバリ設計が必須
* 特定条件でしか表示されない要素がある場合、`page.keyboard.press("End")` などで強制スクロールも選択肢
* 大量データの場合、**途中保存・中断復帰**の仕組み（例：スクロール位置・IDログの保存）が有効



### 応用：画像や動画コンテンツの確実な読み込み

無限スクロール型で画像や動画が遅延読み込みされる場合、次のような工夫が必要：

* `.get_attribute("src")` が `data-src` である場合は `evaluate()` で補完
* `<video>` タグに対して `.wait_for_selector("video", state="attached")` で再生状態を確認
* スクリーンショットによる確認では、`full_page=True` を付けて**すべてのDOM領域をキャプチャ**

```python
await page.screenshot(path="full.png", full_page=True)
```


無限スクロール型のデータ抽出は、Playwrightの**スクリプト的操作力・DOM制御能力・レスポンス検知機能**を最大限に活かせる領域である。動的なDOM操作に対する堅牢な対処法と、パフォーマンス・検出精度の両立は、実運用において極めて重要な課題であり、それに応えうる設計とテストが求められる。


## 第22章：大量データ対応と永続化設計



### 中断・再開可能なスクレイピング設計

大量のデータを対象とするスクレイピングでは、途中でエラーが発生したり、ネットワークが切断されたりするケースが避けられない。そのため、**冪等性（何度実行しても同じ結果になる）と途中再開機能**を備えた構成が必須となる。

#### 基本方針

* **対象ID一覧**や**ページ番号**を事前に取得・保存
* \*\*進捗ログ（例：成功ID・失敗ID）\*\*をファイルやDBに記録
* 次回起動時に**未処理のデータのみ再試行**

#### 実装例（IDログ形式）

```python
import json

done = set(json.load(open("done_ids.json")))
target = [1, 2, 3, ..., 1000]

for id in target:
    if id in done:
        continue
    try:
        fetch_and_save(id)
        done.add(id)
        json.dump(list(done), open("done_ids.json", "w"))
    except Exception as e:
        log_error(id, e)
```

* ID単位で処理可能な場合に特に有効
* 完了IDを随時保存することで、異常終了後でも再開が容易



### 保存形式ごとの利点と選択基準

Playwrightで抽出したデータは、後工程での扱いやすさ・量・構造に応じて保存形式を選定する必要がある。

#### 1. JSON

* **階層構造**に強く、文字列の柔軟性が高い
* 各アイテムを1ファイルに保存、あるいは1レコードずつ追記も可能
* Python・Node.jsなどで扱いやすく、**最も汎用的**

```python
with open("output.json", "w") as f:
    json.dump(results, f, ensure_ascii=False, indent=2)
```

#### 2. CSV

* **平坦なデータ**に強く、表形式処理がしやすい
* Pandas等との連携が容易
* Unicode処理に注意（BOM付きUTF-8推奨）

```python
import csv
with open("output.csv", "w", newline="", encoding="utf-8-sig") as f:
    writer = csv.DictWriter(f, fieldnames=columns)
    writer.writeheader()
    writer.writerows(data)
```

#### 3. SQLite（またはPostgreSQL）

* **構造的に管理**したい場合や、大量データを高速に検索・更新したい場合に適する
* `sqlite3`, `SQLAlchemy`, `Django ORM` などとの統合が容易
* **進捗ログ／メタ情報／再取得フラグ**などの補助情報も併せて格納可能



### スクレイピング処理と保存処理の分離方針

大量データ処理では、スクレイピングと永続化の責務を分けることで再利用性と保守性が高まる。

#### 分離のメリット

* エラー発生時、**保存処理のみ再試行**できる（スクレイピング再実行不要）
* 保存先の変更（JSON → DBなど）が容易
* 処理負荷の分散（並列取得 → バッチ保存）

#### 実装例（データをキューに積んで保存）

```python
queue = []

for item in items:
    data = scrape(item)
    queue.append(data)
    if len(queue) >= 50:
        save(queue)
        queue = []

# 残りを保存
if queue:
    save(queue)
```



### データ件数に応じたバッチ設計の工夫

1件ずつの処理では時間効率が悪いため、バッチ設計によりパフォーマンスを向上させる。

* **件数単位の分割**（例：100件ごとに保存）
* **時間間隔で保存**（例：30秒ごとに書き込み）
* **一時ファイルに保持 → 終了時にまとめて保存**

また、並列取得を行う場合、保存処理は**スレッドセーフ**または**プロセスセーフ**に設計する必要がある（例：Queue, Lockの導入）。



### Playwrightでの実装上の注意点

* ページの状態を保持したまま大量遷移する場合、**メモリリークやキャッシュ肥大化**に注意
* 定期的に `page.close()` や `context.new_page()` でページを開き直すと安定性が向上
* `browser.new_context(storage_state=...)` を使ってログイン状態を使い回す



### エラーリカバリと監視

* **例外の種別ごとにログ記録**（例：Timeout, NetworkErrorなどを分類）
* **ログファイルにタイムスタンプ付きで出力**し、CI/CDでSlack通知やメール送信も可能
* 失敗リスト（IDやURL）を `failed.json` に保存して再取得対象とする設計も有効

大量データのスクレイピングは**正確な分割・堅牢な設計・保存形式の最適化**により、手動操作を最小限に抑えつつ長期運用可能な構成へと進化させられる。Playwrightの操作力をベースに、抽出・整理・永続化を分離した設計は、実務における**大規模データ収集の標準形**となる。


## 第23章：セレクタ耐性と構造変化への対策



### CSSセレクタの弱点とセマンティック指定の活用

スクレイピングや自動テストにおいて、`".item > .title"` のようなCSSセレクタは**見た目上はシンプルだが、構造変化に非常に弱い**。クラス名の変更や階層の追加、順序変更などの**UI変更に脆弱**である。

この問題に対処するためには、**セマンティックなセレクタ**（意味的に安定した要素指定）を優先すべきである。Playwrightは次のような高レベルセレクタを提供しており、これを活用することで**テストやスクレイピングの保守性**を大幅に高められる。

#### セマンティックセレクタの例

* `get_by_role()`：ボタンや見出しなど、ARIAロールベースで指定
* `get_by_text()`：ラベルや表示文字列に基づいて要素を特定
* `get_by_label()`：フォームや入力に対してラベル文字列で指定
* `get_by_test_id()`：開発者が付与した `data-testid` を使用（最も堅牢）

```python
await page.get_by_role("button", name="次へ").click()
await page.get_by_test_id("card-title").text_content()
```



### `try-except` による壊れたDOM対応

実運用においては、想定されたセレクタが**突然存在しなくなる**ことは避けられない。これに備えて、セレクタ操作は常に `try-except` で包むか、`.is_visible()` を組み合わせて防御的に設計する。

#### パターン例：

```python
if await page.locator(".profile").is_visible():
    name = await page.locator(".profile .name").text_content()
else:
    name = "取得不可"
```

```python
try:
    description = await page.locator("#desc").text_content()
except Exception:
    description = None
```

* 複数セレクタ候補を用意し、順に試すフェイルオーバー設計も有効
* 欠落しても致命的でなければ記録しつつ継続（ログに記録する）



### HTML構造変化の検知スクリプトと通知の仕組み

Webサイトの構造が変更されると、既存のセレクタは機能しなくなり、**抽出失敗**や**自動テストの崩壊**を引き起こす。そのため、HTML構造に変化があったことを自動で検出し、**早期に通知できる仕組み**を整えることが重要である。

#### アプローチ例：

1. **HTMLスナップショットを保存**（定期的に `page.content()` をJSON or HTML形式で保存）
2. **前回との差分を比較**（`difflib`, `bs4`, `html.parser` を使用）
3. **差分があれば通知**（Slack webhook / メール / GitHub Issue自動生成など）

```python
from difflib import unified_diff

prev = open("last.html").read().splitlines()
curr = await page.content()
diff = list(unified_diff(prev, curr.splitlines()))

if diff:
    send_to_slack(diff)
```

* 差分の中でも `.class` 変更や構造追加は特に注視する
* JSによる描画差異は `page.screenshot()` で視覚比較も有効



### 構造変化に強いテストとスクレイピング設計原則

構造変化に強い＝保守しやすい設計には、以下の原則が有効である。

#### 原則1：セマンティック重視

* `role`, `aria-label`, `data-testid` を優先
* class名やタグ名に依存しない指定方法を検討

#### 原則2：要素存在の確認を徹底

* `.is_visible()`, `.is_enabled()`, `.count() > 0` などで前提を確認
* `.wait_for_selector()` は状態指定を併用（例：`visible`, `attached`）

#### 原則3：ログと例外の管理

* セレクタの取得失敗はログに詳細記録（対象URL・時刻・エラー内容）
* 自動テストであればスクリーンショットを `screenshots/failed/` に保存

#### 原則4：柔軟な抽出ルールの構造化

* 複数パターンのセレクタを列挙して、順番に試す戦略的構成（例：`primary`, `fallback`, `legacy`）

```python
for selector in selectors:
    try:
        data = await page.locator(selector).text_content()
        if data:
            break
    except Exception:
        continue
```



### 長期運用を見据えたセレクタ戦略

スクレイピングやE2Eテストの長期運用では、**初期構築時のセレクタ戦略が保守性を大きく左右する**。Playwrightの強みは以下のような点にある：

* **意味ベースでの要素取得（get\_by\_\*）**
* **明示的な待機と確認（wait\_for, is\_visible）**
* **エラー・構造変化への対策（try-except, 差分検出）**

加えて、構造の変化やAPIの追加に対応するために、**セレクタマップ（config/seletors.json等）を外部ファイルで定義**しておくと、コードの変更なく再利用できる柔軟性を持てる。

**構造の変化は避けられない**。しかし、セレクタ設計と検知・通知の体制を整備すれば、変化に「壊される」のではなく、変化を「察知し、制御する」側に回ることができる。それこそが、Playwrightを用いた高度なスクレイピング／E2Eテストの真価である。



## 第24章：スクレイピングと倫理・法的配慮



### robots.txtと利用規約の遵守

スクレイピングを行う前に最初に確認すべきなのが、そのサイトの**robots.txt**である。これはサイト運営者が**クロールを許可・制限するルール**を記述したファイルであり、少なくとも以下のチェックを行うべきである：

* `https://example.com/robots.txt` にアクセスし、該当パスに `Disallow` が設定されていないか確認
* `User-agent: *` の対象に自分のクローラが含まれるとみなす
* `Crawl-delay` が定義されている場合、その指示に従う（リクエスト間隔を明示）

Playwrightのような高機能クローラであっても、**robots.txtは無視可能**な仕組みではあるが、無視してよい理由にはならない。法的トラブルやサービス停止を避けるためにも、**事前の機械的チェックと明示的な制限回避**の判断が重要である。

また、robots.txtに加えて、対象サイトの**利用規約（Terms of Service）**に「自動取得・商用利用・再配布の禁止」などが明記されている場合、それを侵害すれば**法的リスク**が生じる。



### アクセス頻度の制御と `wait_for_timeout` の適正使用

スクレイピングによる高頻度のアクセスは、対象サーバに**過剰な負荷**をかけ、結果的にサービス側からアクセスブロックやIP制限を受けることがある。そのため、**アクセス間隔（レート制限）を必ず設計に組み込むこと**が必要である。

#### 制御のポイント：

* `await page.wait_for_timeout(1000)` を各リクエスト間に必ず挿入
* 複数リクエストを並列化せず、逐次処理とする
* 複数ページにアクセスする場合は、**1秒以上のウェイト**を基本とする
* WebSocketやGraphQLを使用している場合、**通信量そのものが制限対象**になることもあるため要注意

#### 負荷回避の設計例：

```python
for url in urls:
    await page.goto(url)
    await page.wait_for_timeout(1500)  # 最低1.5秒待機
```

* 高頻度でアクセスが必要な場合は、robots.txtが許可されていること、または**商用利用APIが用意されているか**の検討も必要



### 商用利用・再配布における注意点

個人利用と異なり、スクレイピングで取得したデータを**商用プロダクトや公開リポジトリに使用する場合**、以下の法的リスクと社会的責任が発生する：

1. **著作権侵害**：
   文章・商品名・レビュー等のコンテンツをそのまま転載することは、原則著作物と見なされ、転載許可が必要となる

2. **契約違反**：
   利用規約に明記されていなくても、**明示的にAPIやデータ提供手段が存在する**サイトでは、スクレイピングが不正手段と判断されることがある

3. **サービス停止リスク**：
   一定以上のアクセスを繰り返すことで、運営元のCDNやWAFが自動遮断し、**IPブロックやアカウント凍結**に発展する可能性がある

4. **訴訟の可能性**：
   企業サイトのスクレイピングが営業妨害と判断された場合、**損害賠償の対象**になるリスクがある（実際に提訴された事例も存在）



### Playwrightの識別回避策と限界

Playwrightのヘッドレスブラウザは、デフォルトのままではJavaScriptによって「自動操作ブラウザ」として検出されることがある。これにより、**コンテンツをブロックされる・CAPTCHAが発生する・画面が非表示になる**などの影響が出る。

#### 対策例：

* **ユーザーエージェントの変更**：
  `context = browser.new_context(user_agent="...")`

* **`navigator.webdriver` を偽装**（Playwrightでは難しい）

* **ランダムなマウス移動やクリックを挿入**して人間らしさを演出

* **ヘッドレスモードをオフにする**：
  開発中は `headless=False` で可視化し、検出回避

```python
browser = await playwright.chromium.launch(headless=False, slow_mo=200)
```

ただし、これらの識別回避策は**倫理的にグレーゾーン**であり、特に**意図的な検出回避は対象サイトの規約違反**につながる場合がある。**正当な理由がない限り推奨されない**。



### 実務的な対処戦略まとめ

* **スクレイピング対象の選定基準**を定める（robots.txt + 規約）
* **商用利用／研究利用／個人利用**の区別を明確にし、目的に応じてリスクを評価
* \*\*定期取得のスケジューリング（cron）\*\*は、夜間帯や非ピーク時間に設定
* **自動取得が禁止されているサービス（例：Twitter, Instagram等）には手を出さない**
* データがAPIで取得可能な場合は、**スクレイピングではなくAPI使用を最優先**



### 倫理的配慮の具体的チェックリスト

* ✅ `robots.txt` を確認したか
* ✅ サイトの利用規約にスクレイピング禁止と明記されていないか
* ✅ サイト提供者に問い合わせまたは許可を取る選択肢を検討したか
* ✅ 商用利用の有無を明確にしたか
* ✅ データ取得の頻度を低く抑えているか（1秒以上の間隔）
* ✅ 非表示のAPIが存在しないか、HTML構造に過度に依存していないか
* ✅ CAPTCHAやWAFによる検出を回避しようとする行為を正当化できるか


スクレイピングは強力な情報取得手段である一方で、**法的・倫理的リスクを常に伴う技術行為**である。Playwrightのような強力なツールを使いこなすには、技術力だけでなく、**公共性・透明性・節度**を持った設計と運用が不可欠である。持続可能な開発のためには、機能の実現だけでなく**責任の伴う技術使用**が求められる。


