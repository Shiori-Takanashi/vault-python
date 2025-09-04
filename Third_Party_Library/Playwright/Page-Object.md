# 📢`Page`

---

## 📍概要

`Page` は、Playwrightにおける**ブラウザタブやウィンドウ**を表すオブジェクトです。
ユーザーが実際に操作するWebページそのものであり、URLへの移動・要素の取得・クリック・入力・スクリーンショット・ダイアログ処理など、あらゆるブラウザ操作はこの `Page` オブジェクトを通じて行います。

---

## 📍用途

* Webページへのアクセス（`goto()`）
* 要素の取得と操作（`locator()` / `click()` / `fill()` など）
* スクリーンショットやPDFの出力（`screenshot()` / `pdf()`）
* JavaScriptの実行（`evaluate()`）
* イベントのリッスン（`on("dialog")` など）

---

## 📍使用法

`Page` は、`Browser` または `BrowserContext` から `new_page()` を呼び出すことで生成されます。

```python
page = browser.new_page()
# または
page = context.new_page()
```

---


## 📍コード例

---

### 📩 1. 指定URLにアクセスしてタイトルを表示

新しいブラウザタブ（`Page`）を開き、`https://example.com` にアクセスします。ページタイトルを取得して出力し、ブラウザインスタンス全体を閉じます。Playwrightの基本的なページ操作の流れを示す典型例です。

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())  # タイトルを出力
    browser.close()
```

---

### 📩 2. 要素の取得とテキスト表示

`page.locator("h1")` によって、ページ内の最初の `h1` 要素を取得し、そのテキスト内容を表示します。見出しやラベルの確認・検証に使えます。

```python
heading = page.locator("h1")
print(heading.text_content())  # h1タグのテキストを表示
```

---

### 📩 3. 入力フォームの操作

名前・メールアドレスの入力フォームに値を入力し、送信ボタンをクリックします。`fill()` は既存の値を上書きして入力でき、送信動作には `click()` を使用します。フォーム送信テストに使われます。

```python
page.fill('input[name="name"]', "John Doe")
page.fill('input[name="email"]', "john@example.com")
page.click('button[type="submit"]')
```

---

### 📩 4. スクリーンショットの保存

現在のページの表示内容をPNG形式で保存します。ファイル名は `"screenshot.png"` に指定されており、UI検証やデバッグ用途に便利です。`headless` 環境でも使用可能です。

```python
page.screenshot(path="screenshot.png")
```

---

### 📩 5. JavaScript の実行

`evaluate()` を使って、ページ内でJavaScriptコードを実行し、その戻り値（ここでは `document.title`）を取得・出力します。ページ内変数や動的要素の確認に役立ちます。

```python
result = page.evaluate("() => document.title")
print(result)  # JavaScriptの戻り値を表示
```

---

### 📩 6. ダイアログイベントの処理

ページ内でアラートや確認ダイアログが表示されたとき、`dialog` イベントをトリガーにしてメッセージを出力し、自動で受け入れ（`accept()`）ます。人手を介さずにダイアログ対応が必要なE2Eテストで使用されます。

```python
def handle_dialog(dialog):
    print(dialog.message)
    dialog.accept()

page.on("dialog", handle_dialog)
```

---

### 📩 7. チェックボックスの操作

チェックボックス要素に対して `check()` を使うことでチェックを付け、`uncheck()` を使うことでチェックを外すことができます。設定変更や利用規約への同意のような操作を自動化する際に使用されます。

```python
page.check('input[type="checkbox"]')
page.uncheck('input[type="checkbox"]')
```

---

### 📩 8. セレクトボックスの選択

`select_option()` メソッドを使用すると、ドロップダウンメニューから特定の値を選択できます。フォームで色や国などを選ばせるような場面で有効です。指定は値（value）またはラベル（label）で行えます。

```python
page.select_option('select[name="color"]', 'red')
```

---

### 📩 9. タイピングのシミュレーション

`type()` メソッドを使うと、ユーザーがキーボードで入力する動作を一文字ずつ再現することができます。`fill()` よりもリアルな動作が必要な場面や、入力イベントに依存するUIの挙動を確認したい場合に適しています。

```python
page.type('input[name="username"]', 'user123')
```

---

### 📩 10. 特定要素が表示されるまで待機

ページの読み込みや動的コンテンツの表示に時間がかかる場合、`wait_for_selector()` を使用して、指定した要素がDOM上に現れるまで待機することで、スクリプトの安定性を高めることができます。

```python
page.wait_for_selector('#result')
```

---

### 📩 11. 要素が消えるまで待機

読み込み中のスピナーや一時的なメッセージなど、不要になった要素が非表示になるまで `wait_for_selector(..., state='hidden')` を使って待機することができます。次の操作のタイミング制御に役立ちます。

```python
page.wait_for_selector('#loading', state='hidden')
```

---

### 📩 12. ページHTMLの取得

現在表示されているページ全体のHTMLソースを文字列として取得するには `content()` を使います。これにより、HTMLの構造を後から検証したり、スナップショット的に記録することが可能です。

```python
html = page.content()
print(html)
```

---

