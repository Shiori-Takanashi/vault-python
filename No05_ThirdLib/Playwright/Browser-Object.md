# 📢2. **`Browser`**

---

## 📍概要

`Browser` は、Playwright における**ブラウザインスタンス**そのものを表すオブジェクトです。`chromium` / `firefox` / `webkit` など、実際のブラウザエンジンごとに起動・操作するために使用されます。

このオブジェクトを通じて、新しいページやコンテキスト（セッション）を生成したり、起動中のブラウザを閉じたりといった管理操作が可能です。

---

## 📍主な用途

* 特定のブラウザエンジン（Chromium, Firefox, WebKit）を起動する
* 新しいページ（`Page`）やコンテキスト（`BrowserContext`）を生成する
* ブラウザ全体を終了（`close()`）させる
* 起動オプション（headless, devtools, proxy等）の設定を行う

---

## 📍使用方法

`Browser` は通常、`sync_playwright()` から取得した `playwright.chromium.launch()` のような関数を通じてインスタンス化されます。取得したインスタンスは、ページ生成や終了処理に使用されます。

以下に、各コード例の上に**より詳細な解説**を追加したバージョンを提示します。`sync_playwright()` に不慣れな読者でも理解しやすいよう、**操作対象・動作内容・目的**を明記しています。

---

## 📍コード例

### **📄1. Chromium ブラウザを起動してページを開く**

**➡ GUI表示ありでChromiumを起動し、新しいタブ（`Page`）を開いて `https://example.com` にアクセスします。ページタイトルを取得・出力し、最後にブラウザを閉じます。開発中にページ内容を目視で確認したい場合などに有効です。**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()
```


### **📄2. ヘッドレスモードでFirefoxを起動**

**➡ 画面を表示せず（headless）、Firefoxを起動して `https://example.org` を読み込みます。ページ全体のスクリーンショットを `firefox.png` という名前で保存し、終了します。CI/CD や自動テスト環境などで実行ログ代わりに使うケースに適しています。**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.firefox.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.org")
    page.screenshot(path="firefox.png")
    browser.close()
```



### **📄3. `BrowserContext` を使ってセッション分離**

**➡ WebKitブラウザを起動し、2つの独立したセッション（`BrowserContext`）を作成して、異なるページをそれぞれのタブで開きます。同一ログイン情報が共有されないため、マルチアカウントテストやCookieの影響を分離したい場合に有効です。**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.webkit.launch()
    context1 = browser.new_context()
    context2 = browser.new_context()

    page1 = context1.new_page()
    page2 = context2.new_page()

    page1.goto("https://example.com")
    page2.goto("https://example.org")

    print(page1.url)
    print(page2.url)
    browser.close()
```


### **📄4. 起動オプションをカスタマイズ**

**➡ Chromiumブラウザを、UI表示あり・操作に500ミリ秒の遅延を加え・DevToolsを自動で開いた状態で起動します。操作ステップをゆっくり確認したいときや、DOM変化をDevToolsで観察しながらデバッグしたい場面に便利です。**

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False, slow_mo=500, devtools=True)
    page = browser.new_page()
    page.goto("https://example.com")
    browser.close()
```

---



## 📍代表的なメソッド一覧

| メソッド名                   | 説明                                                    |
| ----------------------- | ----------------------------------------------------- |
| `launch(**kwargs)`      | 新しいブラウザインスタンスを起動（`chromium`, `firefox`, `webkit` で使用） |
| `new_page()`            | 新しいページ（`Page`）を作成し、デフォルトのコンテキスト上で開く                   |
| `new_context(**kwargs)` | 新しいブラウザコンテキスト（`BrowserContext`）を生成                    |
| `contexts`              | 開かれているすべての`BrowserContext`を取得（`List[BrowserContext]`） |
| `is_connected()`        | ブラウザがPlaywrightと接続されているかどうかを確認（切断されていればFalse）         |
| `version()`             | ブラウザのバージョン文字列を返す（例: `"HeadlessChrome/125.0.0.0"`）     |
| `close()`               | ブラウザ全体を終了し、すべてのページ・コンテキストを閉じる                         |

---

## 📍`launch()` のオプション

| 引数名               | 型      | 説明                                             |
| ----------------- | ------ | ---------------------------------------------- |
| `headless`        | `bool` | ヘッドレスモードで起動する（デフォルトはTrue）                      |
| `slow_mo`         | `int`  | 各操作の間に指定ミリ秒の遅延を追加（デバッグ用）                       |
| `devtools`        | `bool` | DevTools を開いた状態で起動（`headless=False` 時のみ有効）     |
| `args`            | `list` | コマンドライン引数をブラウザ起動時に付与                           |
| `proxy`           | `dict` | プロキシ設定（例: `{"server": "http://myproxy:3128"}`） |
| `executable_path` | `str`  | カスタムブラウザバイナリのパスを指定（例: 特定のChromeを使用）            |

---


