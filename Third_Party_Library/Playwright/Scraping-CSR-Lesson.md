# Scraping-CSR-Lesson


## 第1章：Playwrightのインストールとブラウザセットアップ

この章では、Python環境にPlaywrightを導入し、ブラウザの制御準備を整えます。対象サイトはCSRで構築されているため、JavaScriptを実行可能なブラウザが必要であり、Playwrightはそれを自動で操作するためのライブラリです。

---

### 1.1 Pythonのバージョン確認

PlaywrightはPython 3.7以降に対応しています。ターミナルで以下を実行して確認してください。

```
python --version
```

バージョンが古い場合は、3.10以降の導入を推奨します。

---

### 1.2 Playwrightのインストール

まず、Python環境にPlaywright本体をインストールします。

```
pip install playwright
```

仮想環境を使っている場合は、その内部で実行してください。

---

### 1.3 ブラウザバイナリのインストール

Playwrightをインストールしただけでは、ブラウザ本体は含まれていません。以下のコマンドで、操作対象のブラウザ（Chromium, Firefox, WebKit）を取得します。

```
playwright install
```

エラーが出る場合、インターネット接続やプロキシ環境を確認してください。

---

### 1.4 サンプルスクリプトの実行テスト（任意）

動作確認のため、以下の簡易スクリプトを実行して、正常にブラウザが立ち上がるかを確認します。

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    print(page.title())
    browser.close()
```

正しく動作すれば `"Example Domain"` というタイトルが表示され、セットアップ完了です。

---

## 第2章：プロジェクトディレクトリの作成

この章では、Playwrightを用いたスクレイピングの作業を構造的に進めるために、適切なディレクトリ構成とファイルの雛形を用意します。小規模でも、最低限の構造化を行うことで、後々の保守性と再利用性が大きく向上します。

---

### 2.1 最低限の構成ディレクトリ

以下のような構成を初期状態として用意します。

```
quotes-scraper/
├── scraper/
│   ├── __init__.py
│   └── main.py
├── data/
├── README.md
└── .gitignore
```

各ディレクトリ・ファイルの意味は以下の通りです。

* `scraper/`: 実装ロジック（Playwright操作）をまとめる
* `main.py`: 実行の起点
* `__init__.py`: Pythonパッケージとして認識させる（空でよい）
* `data/`: スクレイピング結果（JSONなど）の保存場所
* `README.md`: 説明・使い方
* `.gitignore`: キャッシュやログ、バイナリ等の除外指定用

---

### 2.2 `.gitignore` の推奨内容（初期）

Gitで管理する場合は、以下を `.gitignore` に含めるとよいでしょう。

```
__pycache__/
*.pyc
*.log
*.png
data/
```

実験ログやスクリーンショットなど、生成ファイルはGitで管理すべきでない場合が多いため、`data/` を初期から除外対象とします。

---

### 2.3 `README.md` の初期テンプレート（任意）

簡易的に以下のように記述しておくと、今後の整理に役立ちます。

```
# Quotes Scraper (Playwright / CSR)

This project scrapes quotes from https://quotes.toscrape.com/js/ using Playwright.
It handles client-side rendering by waiting for JavaScript-rendered elements.

## Structure

- scraper/: core scraping logic
- data/: output JSON files
```

---

### 2.4 main.py の初期内容

この時点ではまだ中身は空でもよいですが、仮に実行用のプレースホルダとして以下のように記述しておいても差し支えありません。

```python
def main():
    print("準備完了。実装はこれから。")

if __name__ == "__main__":
    main()
```

---

## 第3章：Playwright同期APIの基本構文確認

### 3.1 sync\_playwright() の役割

Playwrightには非同期（async）APIと同期（sync）APIの2種類がありますが、本カリキュラムでは扱いやすさを優先し、同期APIを使用します。エントリポイントとなるのが `sync_playwright()` で、`with` 文と併用することでPlaywrightの起動と終了処理を自動で行うことができます。

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    ...
```

### 3.2 ブラウザインスタンスの生成

Playwrightでは、Chromium・Firefox・WebKitの3つのブラウザを制御できます。通常は最も安定して動作するChromiumを選択します。

```python
browser = p.chromium.launch()
```

ブラウザの操作を画面で確認しながら進めたい場合は、`headless=False` を指定することで可視化できます。

```python
browser = p.chromium.launch(headless=False)
```

### 3.3 コンテキストとページの生成

ブラウザ内での操作は「コンテキスト（仮想的なユーザー環境）」と「ページ（タブ）」を使って行います。新しいタブを開いて操作するには、以下のように記述します。

```python
context = browser.new_context()
page = context.new_page()
```

### 3.4 ページの遷移と表示確認

ページオブジェクトを使ってURLへ遷移し、Webサイトを開くことができます。また、タイトルやHTMLの内容を取得することも可能です。

```python
page.goto("https://example.com")
print(page.title())
```

### 3.5 クローズ処理の重要性

ブラウザを使用し終えた後は、必ず `browser.close()` を呼び出して明示的に終了するようにしてください。リソースが解放されず、バックグラウンドで残ることを防ぐためです。

```python
browser.close()
```

---
## 第4章：SSRとCSRの違いを可視化する

### 4.1 SSRとCSRの概念的な違い

* SSR（Server Side Rendering）：HTMLがサーバ側で生成され、レスポンスとしてブラウザに送られます。
* CSR（Client Side Rendering）：HTMLのひな型とJavaScriptが送られ、ブラウザ内でDOMが生成されます。

Playwrightを用いたスクレイピングでは、この違いを理解していないと「HTMLを取得したが要素が存在しない」といった状況に陥ります。

### 4.2 対象サイトの構造確認

Playwrightで `https://quotes.toscrape.com/`（SSR）と `https://quotes.toscrape.com/js/`（CSR）をそれぞれ開き、HTMLの構造差を確認してみます。

```python
page.goto("https://quotes.toscrape.com/")
print(page.content())
```

この例では、名言や著者情報が `<div class="quote">` として明示的にHTMLに含まれていることが確認できます。

続いて、CSRページの場合：

```python
page.goto("https://quotes.toscrape.com/js/")
print(page.content())
```

この出力では、初期HTMLには `<div class="quote">` が含まれておらず、JavaScriptの実行後にのみ要素が表示されることが分かります。

### 4.3 スクレイピングの観点からの違い

* SSRページでは `page.content()` や `response.text()` で目的のHTMLを取得できます。
* CSRページでは `page.locator()` によるDOMの待機と操作が必須になります。

この違いを意識することで、Playwrightを使用する意味（＝ブラウザのJavaScript実行環境を利用する）を明確に理解できます。

---
## 第5章：初期HTMLが空でもlocatorで取得できる理由

### 5.1 CSRサイトでの初期HTMLの状態

`https://quotes.toscrape.com/js/` のようなCSRサイトでは、ブラウザに最初に送られてくるHTMLは非常に簡素です。JavaScriptが実行されるまで、コンテンツ（名言や著者情報など）はDOM上に存在しません。

そのため、以下のように `page.content()` を使用しても、意図した要素が含まれないことがあります。

```python
page.goto("https://quotes.toscrape.com/js/")
print(page.content())
```

この結果は、**静的HTMLのみ** を出力しており、JavaScriptによって後から挿入される内容は含まれていません。

### 5.2 locator() の再評価の仕組み

`page.locator(selector)` は、Playwrightが提供する**遅延評価可能なオブジェクト**です。このオブジェクトは、HTMLを即座に確定させるのではなく、**要素が出現するまで何度も内部で評価を繰り返します**。

```python
locator = page.locator(".quote")
print(locator.count())
```

このように `locator` を通して要素にアクセスする場合、JavaScriptでDOMが更新されてからの状態が対象になります。

### 5.3 text\_content() による要素取得の確認

以下のように `.locator()` で取得したオブジェクトに対して `.text_content()` を呼び出すことで、実際にレンダリングされた内容を取得できます。

```python
first_quote = page.locator(".quote").first
print(first_quote.text_content())
```

JavaScriptの実行が完了していないタイミングでは、エラーや空文字が返るため、適切な待機（次章で解説）が必要です。

### 5.4 なぜ query\_selector より locator が推奨されるのか

* `.query_selector()` は即座に現在のDOM状態を評価します。
* `.locator()` は将来的なDOM変化にも対応できるため、**安定性が高く、CSR向き**です。

特に動的なページでは `.locator()` を用いることで、非同期変化に対応した堅牢なスクリプトが構築可能です。

---
## 第6章：待機戦略の導入（wait\_for\_selector）

### 6.1 なぜ待機が必要なのか

CSRサイトでは、ページ遷移直後に要素を取得しようとしても、JavaScriptの実行が完了していないため、DOMに目的の要素が存在しない場合があります。そのため、**必要な要素が描画されるまで待機する処理**が不可欠です。

このような待機なしのコードは失敗します。

```python
page.goto("https://quotes.toscrape.com/js/")
quote = page.locator(".quote")
print(quote.count())  # 0になることがある
```

### 6.2 wait\_for\_selector の基本構文

Playwrightでは `.wait_for_selector()` を使って、要素の出現を明示的に待機できます。

```python
page.wait_for_selector(".quote")
```

この処理は、`.quote` 要素がDOM上に現れるまでブロックします（デフォルトでは最大30秒）。

### 6.3 正常な取得との組み合わせ

待機後に `.locator()` を使用することで、CSRによって描画された要素を安全に取得できます。

```python
page.goto("https://quotes.toscrape.com/js/")
page.wait_for_selector(".quote")
quotes = page.locator(".quote")
print(quotes.count())
```

このように、**遷移 → 待機 → 操作**の流れを明確にすることで、非同期描画に強いスクレイピングが実現できます。

### 6.4 明示的待機と暗黙的待機の比較

* 明示的待機：`wait_for_selector()` で指定要素を待つ（推奨）
* 暗黙的待機：Playwright内部の再評価に任せる（不安定になりやすい）

実際のスクレイピングでは、**明示的な待機処理を挿入することが、再現性と安定性の面で必須**です。

---
## 第7章：単一ページから全quote要素の抽出

### 7.1 対象要素の確認

対象サイト（`https://quotes.toscrape.com/js/`）では、名言ごとに以下のような構造が繰り返されています。

```html
<div class="quote">
    <span class="text">“名言の内容...”</span>
    <span>by <small class="author">著者名</small></span>
    <div class="tags">
        <a class="tag">タグ1</a>
        <a class="tag">タグ2</a>
    </div>
</div>
```

まずは `.quote` 要素をすべて取得することが目標です。

### 7.2 page.locator() による要素のリスト取得

Playwrightでは、該当する複数のDOM要素を `page.locator(".quote")` で一括して取得できます。これにより、各要素を `.nth(i)` で順番に取り出せるようになります。

```python
page.wait_for_selector(".quote")
quotes = page.locator(".quote")
```

### 7.3 要素数の確認

取得した `Locator` オブジェクトには `.count()` メソッドがあり、対象となる `.quote` 要素がいくつあるかを確認できます。

```python
count = quotes.count()
print(f"このページには {count} 件の名言があります")
```

### 7.4 順次処理のための準備

次章以降でこの `quotes` オブジェクトを `for` ループで反復し、名言テキスト・著者名・タグをそれぞれ取り出していきます。

Playwrightでは、`Locator` オブジェクト自体はイテラブルではないため、`.count()` と `.nth(i)` の組み合わせで反復処理を行います。

---
## 第8章：各quote要素からtext・authorを抽出

### 8.1 目的

前章で `.quote` 要素をすべて取得しました。本章ではその中から、名言本体（text）と著者名（author）を個別に取り出します。対象要素は、それぞれ `.text` と `.author` のクラス名を持っています。

### 8.2 1件目の抽出処理（例）

まずは `.quote` の1件目から名言テキストと著者名を取り出す例です。

```python
quote = page.locator(".quote").nth(0)
text = quote.locator(".text").text_content()
author = quote.locator(".author").text_content()

print(f"{text} — {author}")
```

このように、親要素（`.quote`）から相対的に `.text` と `.author` を探索することで、要素の関係を保ったまま情報を抽出できます。

### 8.3 複数件への拡張

すべての `.quote` 要素に対して同様の処理を行うには、次のようにループで `.nth(i)` を使って順次アクセスします。

```python
quotes = page.locator(".quote")
count = quotes.count()

for i in range(count):
    quote = quotes.nth(i)
    text = quote.locator(".text").text_content()
    author = quote.locator(".author").text_content()
    print(f"{text} — {author}")
```

### 8.4 エラーへの注意

`Locator` オブジェクトに対して `.text_content()` を呼び出す前に、要素が存在していることを前提とします。CSRサイトでは、描画が遅れている場合に `None` や空文字列が返ることがあるため、必ず `wait_for_selector()` を使ってから `.locator()` を実行してください。

---
## 第9章：タグ一覧の抽出（複数要素）

### 9.1 対象構造の確認

名言ごとの `.quote` 要素内には、タグを含む `.tags` ブロックが存在し、複数の `<a class="tag">` 要素としてリストされています。

```html
<div class="tags">
    <a class="tag">life</a>
    <a class="tag">inspirational</a>
    <a class="tag">humor</a>
</div>
```

タグは存在しない場合もあるため、抽出に際しては例外的なケースも想定する必要があります。

### 9.2 Locatorの繰り返し処理におけるタグ取得

1つの `.quote` 要素から複数の `.tag` 要素を取得し、すべてのテキストをリスト化するには `.all_text_contents()` を使用します。

```python
quote = page.locator(".quote").nth(0)
tags = quote.locator(".tag").all_text_contents()
print(tags)  # 例: ['life', 'inspirational', 'humor']
```

`.all_text_contents()` は、複数のノードを一括で処理できる便利なメソッドであり、タグのように繰り返し構造を持つ場面で非常に有効です。

### 9.3 タグが存在しない場合への備え

タグが一つも存在しない `.quote` 要素もあるため、必ずリストとして受け取る構文にしておくことで、空リストでも安全に処理が進行します。

```python
tags = quote.locator(".tag").all_text_contents()
if not tags:
    print("タグなし")
```

このようにして、名言・著者・タグの3要素を一貫した構造で取得できるようになります。次章ではこれらを辞書形式に整形します。

---
## 第10章：要素単位での辞書構造への変換

### 10.1 目的

各 `.quote` 要素から抽出した情報（名言テキスト、著者名、タグ）を、後工程で扱いやすくするために、辞書（`dict`）形式にまとめます。これにより、JSONへの変換やデータ加工が容易になります。

### 10.2 基本形式

以下の3つの情報を1つの辞書にまとめます。

* `"quote"`: 名言テキスト（例：`"“Life is what happens when you're busy making other plans.”"`）
* `"author"`: 著者名（例：`"John Lennon"`）
* `"tags"`: タグのリスト（例：`["life", "plans"]`）

### 10.3 実装例（1件分）

```python
quote = page.locator(".quote").nth(0)

text = quote.locator(".text").text_content()
author = quote.locator(".author").text_content()
tags = quote.locator(".tag").all_text_contents()

quote_dict = {
    "quote": text,
    "author": author,
    "tags": tags
}

print(quote_dict)
```

出力例：

```python
{
    "quote": "“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”",
    "author": "Albert Einstein",
    "tags": ["change", "deep-thoughts", "thinking", "world"]
}
```

### 10.4 データ整形時の注意点

* `.text_content()` で取得した文字列には前後の空白や改行が混じることがあります。
* 必要であれば `.strip()` を適用してもよいですが、本カリキュラムでは元の文字列を保持する方針とします。
* `None` が返る可能性がある場合には、`or ""` などで空文字に置き換えることを検討してください。

---
## 第11章：全要素の辞書化とリスト構造化

### 11.1 複数の `.quote` 要素の処理方針

1ページ内には複数の `.quote` 要素が存在します。それぞれから名言・著者・タグを抽出し、辞書形式に整形した上でリストとしてまとめます。これにより、後のJSON出力や統計処理が容易になります。

### 11.2 `.count()` と `.nth(i)` を用いたループ

Playwrightの `Locator` オブジェクトは直接 `for` で反復できないため、インデックス指定で処理します。

```python
quotes = page.locator(".quote")
count = quotes.count()
results = []

for i in range(count):
    quote = quotes.nth(i)
    text = quote.locator(".text").text_content()
    author = quote.locator(".author").text_content()
    tags = quote.locator(".tag").all_text_contents()

    quote_dict = {
        "quote": text,
        "author": author,
        "tags": tags
    }
    results.append(quote_dict)
```

### 11.3 最終的なリスト構造

上記の `results` は次のような形式になります。

```python
[
    {
        "quote": "“A witty saying proves nothing.”",
        "author": "Voltaire",
        "tags": ["wit"]
    },
    {
        "quote": "“The world as we have created it is a process of our thinking...”",
        "author": "Albert Einstein",
        "tags": ["change", "deep-thoughts"]
    },
    ...
]
```

このリストは次章でそのままJSONに変換してファイル出力するための入力として使用します。

---
## 第12章：単ページデータをJSON出力

### 12.1 データの出力形式と目的

前章で作成した名言情報のリスト（`list[dict]`）を、JSONファイルとして保存します。JSON形式で保存することで、後のデータ集計・再利用・検証が容易になります。

### 12.2 Python標準ライブラリによる保存

Pythonの `json` モジュールと `pathlib.Path` を使って、ファイル出力を行います。以下のように書くことで、安全かつUTF-8形式で保存できます。

```python
import json
from pathlib import Path

Path("data").mkdir(exist_ok=True)
Path("data/page1.json").write_text(
    json.dumps(results, ensure_ascii=False, indent=2),
    encoding="utf-8"
)
```

### 12.3 各オプションの説明

* `ensure_ascii=False`：日本語などの非ASCII文字をエスケープせずに出力する
* `indent=2`：見やすいインデント付きで整形
* `encoding="utf-8"`：ファイルが文字化けしないよう明示指定

### 12.4 書き込み完了の確認

書き込み後、ファイルを開いて以下のような形式で出力されていれば成功です。

```json
[
  {
    "quote": "“A witty saying proves nothing.”",
    "author": "Voltaire",
    "tags": ["wit"]
  },
  ...
]
```

このファイルが正常に作成されることを確認したうえで、次章からはページ遷移を含む複数ページのスクレイピングに進みます。

---
## 第13章：「次へ」リンクの構造を確認

### 13.1 対象サイトのページネーション構造

`https://quotes.toscrape.com/js/` は複数ページにまたがって名言が掲載されており、ページ下部に「Next」リンクが表示されます。HTML構造は以下のようになっています。

```html
<ul class="pager">
  <li class="next">
    <a href="/js/page/2/">Next →</a>
  </li>
</ul>
```

このリンクが存在するか否かで、**ページが続くかどうかを判定**できます。

### 13.2 nextリンクのセレクタ

次ページのリンクを取得するためには、以下のようにセレクタを指定します。

```python
next_link = page.locator(".next > a")
```

Playwrightの `.locator()` は、指定した要素がDOM上に存在しない場合でもエラーを出さず、存在判定やクリック前提の処理を柔軟に行えます。

### 13.3 存在チェックの方法

「次へ」が存在するかを判定するには `.count()` を使用します。

```python
if next_link.count() > 0:
    print("次のページがあります")
else:
    print("最終ページです")
```

`count()` が 0 であれば `.next` ブロックそのものが存在していない、すなわち最終ページであると判断できます。

### 13.4 注意点

* CSRサイトでは、`.next` 要素もJavaScriptで描画されるため、`page.wait_for_selector(".quote")` によって主要コンテンツが表示されてからでないと `.next` の有無が正確に判定できません。
* `page.wait_for_selector(".next")` を直接待機すると、最終ページでタイムアウトが発生するおそれがあります。存在チェックは `.count()` を使うことで安全に行えます。

---
## 第14章：クリック操作の基本とページ切替

### 14.1 ページ遷移の必要性

前章で「次へ」リンクの存在を確認しました。本章では、実際にこのリンクをクリックして次のページへ遷移し、再びスクレイピングを継続する処理を実装します。

### 14.2 基本的なクリック処理

Playwrightでは、`Locator` オブジェクトに対して `.click()` を呼び出すことで、クリック操作を行います。

```python
next_link = page.locator(".next > a")
next_link.click()
```

この操作で次のページに遷移しますが、**遷移後の要素描画が完了している保証はない**ため、明示的な待機処理が必要です。

### 14.3 遷移後の待機処理

CSRサイトでは、リンクのクリック後にJavaScriptを通じてDOMが再描画されます。そのため、次のページにおいて `.quote` 要素が出現するのを待機する必要があります。

```python
page.wait_for_selector(".quote")
```

この2行をセットで使用することで、次のページへの安全な遷移とスクレイピングの継続が可能になります。

```python
next_link.click()
page.wait_for_selector(".quote")
```

### 14.4 注意点と落とし穴

* `.click()` は非同期で進行するため、**必ず何らかの待機処理を直後に設ける**ことが重要です。
* `.goto()` でURLを再度指定することも可能ですが、本サイトでは `.click()` によるナビゲーションが自然です。
* ボタンクリック直後に `.locator()` を用いて要素にアクセスしようとすると、DOM未構築によるエラーや空データの原因になります。

この処理をループに組み込むことで、ページを巡回しながら全データを取得する準備が整います。

---
## 第15章：次のページの取得とループ構造

### 15.1 全ページを自動巡回する必要性

`https://quotes.toscrape.com/js/` は、複数ページに分割されて名言が掲載されており、ページ下部の「Next」リンクをたどることで全件を取得できます。本章では、**繰り返しクリックによって全ページを巡回するループ構造**を実装します。

### 15.2 ループの基本構造

「次のページが存在する限り、クリックして進む」処理は以下のように `while True` で構成します。

```python
while True:
    # 名言の取得処理（後述）
    
    next_link = page.locator(".next > a")
    if next_link.count() == 0:
        break  # 最終ページなので終了

    next_link.click()
    page.wait_for_selector(".quote")
```

このループによって、`"Next"` リンクがある限り繰り返し遷移を行い、最終ページでのみ脱出します。

### 15.3 安全な構造のポイント

* `.count()` による存在チェックで、**明示的に終了条件を管理**できます。
* `.click()` の直後に `.wait_for_selector(".quote")` を挿入し、**描画待ち**を保証します。
* `.quote` が存在しないページはこのサイトには存在しないため、`.quote` の有無を使ったブレーク条件には不適です。

### 15.4 繰り返しの中でデータを蓄積

ループ内に「1ページ分の名言データ取得処理」を挿入することで、全ページの情報を収集可能になります。

```python
all_results = []

while True:
    results = extract_quotes_from_current_page(page)
    all_results.extend(results)

    next_link = page.locator(".next > a")
    if next_link.count() == 0:
        break

    next_link.click()
    page.wait_for_selector(".quote")
```

この構成により、全ページを漏れなく、エラーなく処理する基盤が完成します。

---
## 第16章：ループ内に単ページ抽出を関数化

### 16.1 抽出処理の責務を明確に分離する

前章のループ内では、ページごとの名言データの抽出を直接記述していました。しかし、可読性と再利用性の観点から、**1ページ分の抽出処理を関数として切り出すことが望ましい**です。

ここでは以下のような関数を定義します。

```python
def extract_quotes_from_page(page) -> list[dict]:
    ...
```

この関数は、引数に渡された `page` オブジェクトをもとに、単一ページ内の `.quote` 要素から辞書形式のデータを取得し、`list[dict]` として返します。

### 16.2 関数内部の実装例

```python
def extract_quotes_from_page(page) -> list[dict]:
    quotes = page.locator(".quote")
    count = quotes.count()
    results = []

    for i in range(count):
        quote = quotes.nth(i)
        text = quote.locator(".text").text_content()
        author = quote.locator(".author").text_content()
        tags = quote.locator(".tag").all_text_contents()

        results.append({
            "quote": text,
            "author": author,
            "tags": tags
        })

    return results
```

この関数は、**1ページに含まれる名言情報すべてを抽出して返す**ことに責任を持ちます。

### 16.3 ループ内での関数呼び出し

抽出処理を上記関数に置き換えることで、ループ本体は以下のように簡素化されます。

```python
all_results = []

while True:
    data = extract_quotes_from_page(page)
    all_results.extend(data)

    next_link = page.locator(".next > a")
    if next_link.count() == 0:
        break

    next_link.click()
    page.wait_for_selector(".quote")
```

このように構造化することで、テスト可能性・見通し・再利用性のすべてが向上します。

---
## 第17章：ページ数カウントとログ出力

### 17.1 処理状況の把握の重要性

長時間実行されるスクレイピング処理では、**現在どのページを処理しているか**をログ出力で確認できるようにしておくことが重要です。これにより、進捗の把握、異常検出、途中停止時の再実行判断が可能になります。

### 17.2 ページ番号カウントの導入

ループの外でページ番号を初期化し、各ループごとにインクリメントします。

```python
page_num = 1
```

ループ内部では、抽出後に以下のようにログを表示します。

```python
print(f"[Page {page_num}] {len(data)} quotes collected")
page_num += 1
```

これにより、標準出力上に以下のような情報が逐次表示されます。

```
[Page 1] 10 quotes collected
[Page 2] 10 quotes collected
[Page 3] 5 quotes collected
```

### 17.3 ログ出力の整形と応用

必要に応じて、ファイルへの記録やタイムスタンプの付与なども行えますが、ここでは標準出力のみを使用します。

冗長さを避けるため、1ページごとに1行だけ出力するのが望ましく、過度なログは避けます。

### 17.4 完全なループ構造の再掲（簡略）

```python
all_results = []
page_num = 1

while True:
    data = extract_quotes_from_page(page)
    all_results.extend(data)
    print(f"[Page {page_num}] {len(data)} quotes collected")
    page_num += 1

    next_link = page.locator(".next > a")
    if next_link.count() == 0:
        break

    next_link.click()
    page.wait_for_selector(".quote")
```

この出力があることで、進捗・停止位置・全体量などの把握が即座に可能になります。

---
