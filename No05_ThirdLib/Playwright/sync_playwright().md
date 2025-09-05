# 📢 **`sync_playwright()`**

---

## 📍概要

`sync_playwright()` は、Playwrightの同期APIを使用するためのエントリーポイントです。通常、Playwrightは非同期APIとして動作しますが、`sync_playwright()`を使用することで、非同期処理の複雑さを避け、同期的にブラウザ操作を行うことができます。これにより、非同期処理に不安がある開発者でも、直感的にブラウザの操作を行うことが可能になります。

---

## 📍主な用途

* 非同期処理を使いたくない、または使えない場合に、簡単にブラウザを同期的に操作したい時に使います。
* 非同期コードの管理やエラーハンドリングを気にせずに、シンプルなコードでブラウザを操作するために使います。

---

## 📍使用方法

`sync_playwright()`はコンテキストマネージャとして使用されます。このため、`with`ブロックを使ってPlaywrightの初期化からブラウザ操作終了までを簡単に管理できます。

---

## 📍コード例



### **📄1. 基本的なページ操作**

指定したURLにアクセスし、ページタイトルを取得する基本的なコードです。このコードは、ブラウザを起動して指定したURLを開き、ページタイトルを取得してコンソールに出力し、その後ブラウザを閉じる動作を行います。

```python
from playwright.sync_api import sync_playwright

# Playwrightを使ってブラウザを起動する
with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)  # headless=False でブラウザを表示
    page = browser.new_page()  # 新しいページを作成
    page.goto("https://example.com")  # 指定したURLに移動
    print(page.title())  # ページタイトルを出力
    browser.close()  # ブラウザを閉じる
```


### **📄2. スクリーンショットの取得**

ページのスクリーンショットを取得して、指定したパスに保存するコードです。このコードは、ブラウザを起動してURLを開き、ページのスクリーンショットを撮影して指定したパスに保存します。

```python
from playwright.sync_api import sync_playwright

# スクリーンショットを撮影する例
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # ヘッドレスモードで起動
    page = browser.new_page()  # 新しいページを作成
    page.goto("https://example.com")  # 指定したURLに移動
    page.screenshot(path="example_screenshot.png")  # スクリーンショットを保存
    browser.close()  # ブラウザを閉じる
```

### **📄3. 要素の取得と操作**

ページ内の特定の要素を取得し、その内容を取得したり操作したりする例です。このコードは、ページ内の`h1`タグのテキストを取得し、指定されたボタンをクリックする動作を行います。

```python
from playwright.sync_api import sync_playwright

# 要素を取得して内容を操作する例
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")

    # 例: "h1"タグのテキストを取得
    heading = page.locator("h1")
    print(heading.text_content())  # h1のテキストを出力

    # 例: ボタンをクリックする
    button = page.locator("button#submit")
    button.click()  # ボタンをクリック

    browser.close()
```

### **📄4. フォームの入力と送信**

フォームに値を入力し、送信するコード例です。このコードは、指定されたフォームの`name`と`email`フィールドに入力し、送信ボタンをクリックしてフォームを送信します。

```python
from playwright.sync_api import sync_playwright

# フォームの入力と送信
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com/contact")  # フォームがあるページに移動

    # フォームの入力フィールドに値を入力
    page.fill('input[name="name"]', 'John Doe')
    page.fill('input[name="email"]', 'john.doe@example.com')

    # フォームを送信
    page.click('button[type="submit"]')  # 送信ボタンをクリック

    # 結果を確認
    print(page.url())  # リダイレクト後のURLを確認
    browser.close()
```

### **📄5. ページ内のリンクをクリックして移動**

ページ内のリンクをクリックして別のページに移動するコードです。このコードは、ページ内の特定のリンクをクリックし、その後移動したページのURLを取得して表示します。

```python
from playwright.sync_api import sync_playwright

# リンクをクリックして移動する例
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")

    # リンクをクリックして移動
    link = page.locator("a[href='/about']")
    link.click()

    # 新しいページに移動した後のURLを確認
    print(page.url())  # 移動後のURLを表示
    browser.close()
```



### **📄6. 要素の存在確認と条件分岐**

ページ内に特定の要素が存在するかを確認し、その後の処理を分岐させる方法です。このコードは、ページ内に`h1`タグが存在するかどうかを確認し、その結果に応じてメッセージを出力します。

```python
from playwright.sync_api import sync_playwright

# 要素の存在確認と条件分岐
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")

    # 要素が存在するかどうかを確認
    if page.locator("h1").is_visible():
        print("タイトルが表示されています")
    else:
        print("タイトルが見つかりません")

    browser.close()
```


### **📄7. ダイアログの操作**

アラートや確認ダイアログを操作するコードです。このコードは、ページでダイアログが表示されると、それをキャッチしてメッセージを出力し、ダイアログを受け入れます。

```python
from playwright.sync_api import sync_playwright

# ダイアログを操作する例
def handle_dialog(dialog):
    print(f"ダイアログ内容: {dialog.message}")
    dialog.accept()  # ダイアログを受け入れる

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # ダイアログが出るアクションを行う
    page.on("dialog", handle_dialog)  # ダイアログが出たときに呼ばれる
    page.goto("https://example.com/alert")  # ダイアログをトリガーするページに移動

    browser.close()
```



### **📄8. ページの遅延を待機する**

ページの遅延やネットワーク要求が完了するまで待機するコードです。このコードは、指定されたページが完全に読み込まれるまで待機します。

```python
from playwright.sync_api import sync_playwright

# ページが完全に読み込まれるまで待機
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com", wait_until="networkidle")  # ページの読み込みが完了するまで待機
    print("ページが完全に読み込まれました")

    browser.close()
```



### **📄9. 複数ページを開いて操作**

同一ブラウザ内で複数のページを開き、それぞれに操作を加える方法です。このコードは、同一ブラウザ内で2つのページを同時に開き、それぞれのページのタイトルを表示します。

```python
from playwright.sync_api import sync_playwright

# 複数ページを開いて操作する例
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page1 = browser.new_page()
    page2 = browser.new_page()

    page1.goto("https://example.com")
    page2.goto("https://example.org")

    print(page1.title())  # 最初のページのタイトルを出力
    print(page2.title())  # 2番目のページのタイトルを出力

    browser.close()
```



---
