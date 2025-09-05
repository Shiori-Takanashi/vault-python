# 📢Page-Method 

---

## 📍 ナビゲーション

以下のメソッドを活用することで、ページが完全に読み込まれるのを待ったり、特定のURLに遷移したことを検知したりといった制御が可能になります。これにより、**非同期で変化するWebページに対しても安定した自動操作が実現できます**。特に `wait_for_*` 系のメソッドは、要素が表示される前に操作を実行してしまう問題を回避するための基本手段です。



### 📄 `goto(url, **kwargs)`

ページを指定したURLへ遷移させるメソッドです。オプションの `wait_until` 引数により、ページのどの読み込み段階まで待つかを制御できます。たとえば `"networkidle"` を指定すると、すべてのネットワーク通信が一定時間静止した時点まで待機します。

```python
page.goto("https://example.com", wait_until="networkidle")
```



### 📄 `reload(**kwargs)`

現在のページを再読み込みします。ユーザー操作後の状態更新や、キャッシュを無視した動作確認などに適しています。`goto()` と同様に `wait_until` を併用可能です。

```python
page.reload()
```



### 📄 `go_back(**kwargs)`

ブラウザの履歴を1つ戻ります。リンククリックやフォーム送信後に前のページへ戻る処理をスクリプトで再現する際に使用します。

```python
page.go_back()
```



### 📄 `go_forward(**kwargs)`

`go_back()` で戻った履歴を再び進む操作を行います。ブラウザの「進む」ボタンの動作に相当します。

```python
page.go_forward()
```



### 📄 `wait_for_url(url, **kwargs)`

現在のページのURLが指定されたパターンに一致するまで待機します。フォーム送信後や認証後に遷移先が確認できるまで待つようなケースで使います。

```python
page.wait_for_url("**/thank-you")
```


### 📄 `wait_for_load_state(state)`

ページの読み込み状態が指定した段階に達するまで明示的に待機します。`"load"`（全読み込み完了）、`"domcontentloaded"`（DOM構築完了）、`"networkidle"`（リクエスト静止状態）から選択します。

```python
page.wait_for_load_state("domcontentloaded")
```

---


## 📍 要素の取得・操作

以下に示すメソッドは、いずれも `page.locator(selector)` を**内部的に使用**して要素にアクセス・操作を行う仕組みになっています。すなわち、これらのメソッドを呼び出すと Playwright は自動的に `Locator` オブジェクトを生成し、その上で対応するアクション（クリック、入力など）を実行します。この構造により、明示的に `locator()` を使わずとも、**安全かつ再試行を伴う堅牢な操作**が可能になります。


### 📄 `locator(selector)`

指定したCSSセレクタに一致する要素を取得するためのエントリーポイントです。この戻り値は `Locator` オブジェクトであり、クリックやテキスト取得などの操作に使用できます。

```python
heading = page.locator("h1")
print(heading.text_content())
```


### 📄 `click(selector)`

指定された要素をクリックします。内部的には `locator(selector).click()` 相当の操作が行われます。ボタンやリンク、トグルなどのUI操作に広く使用されます。

```python
page.click("button#submit")
```



### 📄 `dblclick(selector)`

対象要素に対してダブルクリックを行います。通常は `click()` とは異なる動作を持つUI（たとえば名前編集やファイル選択の開始など）に使われます。

```python
page.dblclick(".editable-field")
```


### 📄 `fill(selector, value)`

入力フィールドに指定した文字列を直接セットします。既存の内容は自動で上書きされます。テキストボックスや検索フォームなどの入力に使います。

```python
page.fill('input[name="username"]', "test_user")
```



### 📄 `type(selector, text)`

ユーザーが手動でタイプしたように、一文字ずつ入力します。文字ごとの入力イベントが発火するため、リアルな入力動作を再現したい場合に使用します。

```python
page.type('input[name="comment"]', "Hello world!")
```




### 📄 `check(selector)`

チェックボックスをONにします。すでにONのときは何も起きません。利用規約への同意や設定ONに使われます。

```python
page.check('input[type="checkbox"]')
```



### 📄 `uncheck(selector)`

チェックボックスをOFFにします。すでにOFFのときは何も起きません。設定をOFFにするようなUI操作に適しています。

```python
page.uncheck('input[type="checkbox"]')
```




### 📄 `select_option(selector, value)`

ドロップダウン（`<select>`要素）から、指定された値やラベルのオプションを選択します。フォーム入力やカテゴリ指定に使われます。

```python
page.select_option('select[name="color"]', 'red')
```




### 📄 `set_input_files(selector, filepath)`

ファイル選択UIに対して、プログラムからファイルを指定してアップロード操作を行います。画像アップロードやCSVインポートなどで使用されます。

```python
page.set_input_files('input[type="file"]', 'sample.csv')
```



### 📄 `press(selector, key)`

指定要素上で、特定のキー入力（Enter、Tabなど）をシミュレートします。フォーム送信やフォーカス移動に便利です。

```python
page.press('input[name="search"]', 'Enter')
```


---

## 📍 可視化（スクリーンショット・PDF）

以下に示すメソッドは、Webページの視覚的な状態を画像やPDFとして保存するために使用されます。UIの自動検証、報告用キャプチャ、レスポンシブデザイン確認など、Playwrightによる画面出力操作の中心となります。



### 📄 `screenshot(path, full_page=False)`

現在のページをPNG形式の画像として保存します。指定された `path` にファイルを出力し、`full_page=True` を指定するとページ全体（スクロール含む）をキャプチャできます。デフォルトでは表示領域のみが対象です。

```python
page.screenshot(path="capture.png", full_page=True)
```



### 📄 `pdf(path, format="A4")`

ページ全体をPDFとして出力します。この機能はChromiumブラウザでのみ利用可能です。印刷レイアウトの確認や、資料としての保存に適しています。

```python
page.pdf(path="page.pdf", format="A4")
```

---

## 📍 アサーション・待機

以下に示すメソッドは、ページの状態や要素の存在を確認する目的で使用されます。非同期に変化するUIに対して適切な待機処理を挿入することで、テストや操作の安定性を大きく向上させることができます。



### 📄 `wait_for_selector(selector)`

指定したセレクタに一致する要素が DOM に現れるまで待機します。動的に生成されるコンテンツや、Ajaxによって後から表示される要素の操作前に必須となるメソッドです。

```python
page.wait_for_selector("#result")
```



### 📄 `wait_for_timeout(ms)`

指定されたミリ秒（ms）だけ明示的に待機します。デバッグ時や、意図的に処理間隔を空けたい場合に使用されます。Python の `time.sleep()` と異なり、Playwrightの内部イベントループに沿った非ブロッキングな待機です。

```python
page.wait_for_timeout(2000)  # 2秒待機
```



### 📄 `wait_for_function(expr)`

指定された JavaScript 関数の評価結果が真（truthy）になるまで待機します。DOM状態の変化を監視する柔軟な条件を記述でき、より複雑な状態待機に対応できます。

```python
page.wait_for_function("document.querySelector('#status')?.textContent.includes('完了')")
```

---

## 📍 JavaScript 実行

これらのメソッドは、ページ内で JavaScript を直接実行するために使用されます。Playwright を通じて DOM やブラウザの状態にアクセスするだけでなく、複雑なロジックを評価・検証するのにも役立ちます。



### 📄 `evaluate(expr)`

指定された JavaScript 式をページ内で実行し、その戻り値を取得します。例えば `document.title` を取得したり、ページ上の状態を取得・検証する用途に広く使われます。

```python
title = page.evaluate("() => document.title")
print(title)
```


### 📄 `evaluate_handle(expr)`

JavaScript のオブジェクト（たとえば DOM 要素や関数）への参照を取得するために使用します。得られた参照（`JSHandle`）は後で操作したり、破棄（`dispose()`）したりできます。長期的に使う要素に適しています。

```python
element_handle = page.evaluate_handle("() => document.querySelector('#main')")
```

---

## 📍 イベント処理・通信

これらのイベントハンドラは、ページ内で発生する各種イベント（ダイアログ、リクエスト、エラーなど）を監視・制御するために使用されます。ユーザー操作の再現や、通信の観察、エラーの検出といったユースケースで役立ちます。


### 📄 `on("dialog", handler)`

ページ内でアラート、確認、プロンプトなどのダイアログが表示された際に呼び出されるイベントハンドラを設定します。対話型ポップアップへの応答を自動化できます。

```python
def handle_dialog(dialog):
    print("ダイアログ内容:", dialog.message)
    dialog.accept()

page.on("dialog", handle_dialog)
```



### 📄 `on("request")` / `on("response")`

ページが発行したリクエストおよびそれに対するレスポンスを監視します。外部通信の可視化や、リクエストログの取得、レスポンスステータスの検証に利用されます。

```python
page.on("request", lambda request: print("リクエスト:", request.url))
page.on("response", lambda response: print("レスポンス:", response.status))
```



### 📄 `on("console")`

ページ内で `console.log()` や `console.error()` などが呼ばれた際の出力を受け取ることができます。フロントエンドのログをPlaywright側で取得するのに便利です。

```python
page.on("console", lambda msg: print("コンソール出力:", msg.text))
```



### 📄 `on("pageerror")`

JavaScript の実行中に発生した未処理例外（例: 参照エラーや構文エラー）を検出します。UIバグの検出や、スクリプトエラーの監視に使用されます。

```python
page.on("pageerror", lambda error: print("ページエラー:", error.message))
```

---

## 📍 ページの状態制御

これらのメソッドは、ページのタイトルやURLといった基本情報を取得・設定したり、HTMLの内容を操作したり、スクリプトやスタイルを注入したりと、**ページ全体の状態を制御する操作**に関係します。テストや処理の起点として非常に重要な役割を持ちます。



### 📄 `title()`

現在のページのタイトル（`<title>` タグの内容）を文字列として取得します。ナビゲーション後の状態確認や、ページの識別・検証に使われます。

```python
print(page.title())
```



### 📄 `url`（プロパティ）

現在表示中のページのURLを取得するプロパティです。リダイレクト後の遷移確認や、動的遷移の検証に使用されます。

```python
print(page.url)
```



### 📄 `content()`

ページ全体のHTMLソースを文字列として取得します。スクレイピングやHTML構造の検証、状態スナップショットとしての記録などに利用されます。

```python
html = page.content()
print(html)
```



### 📄 `set_content(html)`

与えられたHTML文字列をそのままページ内容として反映させます。特定のHTMLで状態を構成し、そこから操作や検証を行うテストなどで有効です。

```python
page.set_content("<h1>Hello World</h1>")
```



### 📄 `add_script_tag()`

外部スクリプトファイルやインラインJavaScriptをページに注入します。追跡コードや分析用スクリプトを追加したいときや、動的に機能を差し込みたいときに使用されます。

```python
page.add_script_tag(url="https://example.com/analytics.js")
```


### 📄 `add_style_tag()`

外部CSSまたはインラインスタイルを注入します。視覚スタイルの変更や、要素の非表示、デザイン検証などに使われます。

```python
page.add_style_tag(content="body { background: black; }")
```


### 📄 `close()`

現在のページ（タブ）を閉じます。リソースの開放や、複数タブを管理する際の明示的な終了処理に使用されます。

```python
page.close()
```

---


## 📍 ユーティリティ

このセクションでは、`Page` に直接関連する補助的な機能について紹介します。UIの可視化、ユーザー入力のシミュレーション、録画による記録といった目的に対応するメソッドやプロパティが含まれます。テスト中の状態を把握したり、操作ログとして記録する場面で有用です。


### 📄 `bring_to_front()`

ページタブにフォーカスを当て、ブラウザウィンドウの最前面に表示します。マルチタブ環境で特定のページを強調したいときや、デバッグで視覚的に確認したいときに使用されます。

```python
page.bring_to_front()
```



### 📄 `keyboard` / `mouse`

`page.keyboard` や `page.mouse` を使うことで、キーボード入力やマウス操作を直接制御できます。たとえば、キーを押す・離す・クリック・移動・ドラッグ操作などを詳細に記述可能です。

```python
page.keyboard.press("Enter")
page.mouse.move(100, 200)
page.mouse.click(100, 200)
```

---

### 📄 `video`

録画機能にアクセスするためのプロパティです。録画自体は `BrowserContext` 側で有効化が必要ですが、録画中のページオブジェクトに `video.path()` などのメソッドを通じてアクセスできます。テストの再現やUI確認のための記録に適しています。

```python
path = page.video.path()
print("録画ファイル:", path)
```

---
