# 第1章 BeautifulSoupとは何か

---

## 解説

HTMLは視覚的に構造を把握しやすい一方で、プログラムから扱う際にはタグ階層・属性などを適切に処理する必要がある。このようなHTMLドキュメントを\*\*構文解析（パース）\*\*して、特定の情報を効率的に取り出すために使われるのが「HTMLパーサ」である。

\*\*BeautifulSoup（bs4）\*\*は、その中でも特に「柔軟性」と「学習コストの低さ」に優れたPython用のHTMLパーサライブラリである。タグの閉じ忘れや入れ子構造の不整合といった不完全なHTMLにも強く、スクレイピングやHTML整形などで広く用いられている。

---

## bs4の役割と位置づけ

| 項目     | 内容                                                     |
| ------ | ------------------------------------------------------ |
| ライブラリ名 | BeautifulSoup（インポート時は `from bs4 import BeautifulSoup`） |
| 主な用途   | HTML/XMLの解析・整形・検索・編集                                   |
| 位置づけ   | **HTMLパーサのラッパー**（Python的なアクセス手段を提供）                    |
| サポート対象 | HTML4/HTML5, XHTML, XML                                |
| 対応エラー  | 閉じタグ抜け、入れ子エラー、属性ミスなど（多くのブラウザと同様の許容性を持つ）                |
| 学習難易度  | 低（シンプルな文法と明快なメソッド群）                                    |

---

## 他のライブラリとの違い

| ライブラリ                 | 特徴                                               | 備考                                     |
| --------------------- | ------------------------------------------------ | -------------------------------------- |
| BeautifulSoup         | 柔軟性が高く、初心者にも扱いやすい。HTMLの破損にも強い                    | 内部的に複数のパーサ（html.parser, lxml等）を使い分けられる |
| lxml                  | 非常に高速かつ厳密なパーサ。XPathによる強力な検索が可能                   | XPathが書けるユーザー向け                        |
| html.parser           | Python標準のHTMLパーサ。依存が少なく動作も軽いが、柔軟性にやや劣る           | 標準ライブラリのため、インストール不要                    |
| Selenium / Playwright | JavaScript生成コンテンツにも対応。ブラウザエミュレーションを行うため、動的なページ向き | BeautifulSoupとは棲み分け                    |

---

## パーサの選択肢

BeautifulSoupは、**どのパーサを使うか**を明示的に指定することができる。主に以下の3種が用いられる。

| パーサ名          | 特徴                              | 記述例                                  |
| ------------- | ------------------------------- | ------------------------------------ |
| `html.parser` | 標準ライブラリ、インストール不要。十分な性能と柔軟性。     | `BeautifulSoup(html, "html.parser")` |
| `lxml`        | 高速で堅牢。XMLにも対応。XPathを併用したい場合に最適。 | `BeautifulSoup(html, "lxml")`        |
| `html5lib`    | 最も寛容な解析器。Webブラウザと同じ構文解析規則に準拠。   | `BeautifulSoup(html, "html5lib")`    |

---

## ユースケースの一例

* HTMLファイルから指定タグのテキストを取得
* クラス名で要素を抽出し、属性情報を整理
* aタグのhrefからリンク一覧を構築
* 改行や空白を除去して構造化テキストを作成
* 表形式データを2次元リストに変換しCSV出力

---

## 開発者にとっての利点

* **明快なAPI設計**：Pythonicな書き方で直感的に操作できる
* **構文エラー耐性**：壊れたHTMLも扱えるため、実際のWebページに強い
* **柔軟な絞り込み**：属性指定、テキスト一致、正規表現など多彩な絞り込み手段が用意されている
* **他ライブラリと相性良好**：`requests`, `re`, `pandas` などと容易に連携できる

---

## BeautifulSoupの限界

JavaScriptによって**動的に描画される要素**には対応できない。あくまで「HTMLソースの時点で取得可能な情報」に限定されるため、**動的ページにはSeleniumやPlaywrightとの併用が必要**になる。

---

# 第2章 環境構築と基本の使い方

---

## BeautifulSoupのインストール

BeautifulSoupはpip経由で簡単にインストールできる。通常は、パーサである `lxml` や `html5lib` を併せて導入することで、柔軟な動作が可能になる。

```bash
$ pip install beautifulsoup4
$ pip install lxml html5lib  # 推奨（パーサの選択肢を増やす）
```

### 補足

| ライブラリ            | 説明                            |
| ---------------- | ----------------------------- |
| `beautifulsoup4` | 本体ライブラリ。`bs4` モジュール名でインポートされる |
| `lxml`           | 高速・堅牢なパーサ（XML/HTML）           |
| `html5lib`       | HTML5仕様準拠のパーサ。構文の修復能力が高い      |

---

## 最小構成のスクレイピング例（requests + BeautifulSoup）

以下のコードは、最も基本的なHTMLページ取得＋解析の流れを示す。

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com/"
res = requests.get(url)
soup = BeautifulSoup(res.text, "html.parser")  # パーサを指定
print(soup.title.string)
```

### 処理の流れ

1. `requests.get(url)` によりHTMLを取得
2. `BeautifulSoup(res.text, "html.parser")` でパース（構文解析）
3. `soup.title.string` で `<title>` タグの文字列を抽出

---

## ローカルHTMLファイルの読み込みとパース

HTMLファイルを直接読み込んで解析したい場合、以下のように書ける。

```python
from bs4 import BeautifulSoup

with open("sample.html", encoding="utf-8") as f:
    soup = BeautifulSoup(f, "html.parser")

print(soup.prettify())  # 整形表示
```

### 補足ポイント

| 処理内容                  | 解説                                    |
| --------------------- | ------------------------------------- |
| `open(..., encoding)` | ファイルの文字エンコーディングに注意（特に日本語HTML）         |
| `soup.prettify()`     | HTMLをインデント付きで整形して表示できる                |
| `.text` vs `.string`  | `.text` は再帰的、`.string` は子要素が1つだけのとき有効 |

---

## サンプル：タグ取得の基本例

```python
print(soup.h1)            # 最初の<h1>タグ
print(soup.find("p"))     # 最初の<p>タグ
print(soup.find_all("a")) # すべての<a>タグ
```

このように、基本的な操作だけでも多くの情報を取得できる。次章以降でさらに詳細なフィルタリングや構造解析を行っていく。

---

## 最小構成のまとめ

| 必要項目   | 内容                            |
| ------ | ----------------------------- |
| HTTP取得 | `requests.get()`              |
| パース    | `BeautifulSoup(html, parser)` |
| 要素抽出   | `find()`, `find_all()` など     |
| ファイル入力 | `open()` とエンコーディング            |
| 整形表示   | `soup.prettify()`             |

BeautifulSoupの基本操作は非常にシンプルであり、**数行のコードでHTML構造を自在に扱える**のが最大の強みである。

---

# 第3章 find / find\_allの基本

---

## 概要

BeautifulSoupではHTMLの中から**特定の要素を効率的に取得**するために、`find()` と `find_all()` という2つの基本メソッドが提供されている。これらを使いこなすことで、必要な情報を簡潔に抽出できる。

---

## 単一要素取得（find）

`find()` は条件に一致する**最初の1要素のみ**を返す。

```python
soup.find("p")
```

これは `<p>` タグのうち、最初に出現したものを返す。

```python
<p class="intro">これは紹介文です。</p>
```

---

## 複数要素取得（find\_all）

`find_all()` は条件に一致する**すべての要素**をリスト形式で返す。

```python
soup.find_all("a")
```

これにより、ページ内のすべての `<a>` タグ（リンク）を取得できる。

```html
<a href="/news">ニュース</a>
<a href="/contact">お問い合わせ</a>
```

---

## タグ名の指定

```python
soup.find("h1")             # 最初の <h1>
soup.find_all("li")         # すべての <li>
```

`find_all()` の代わりに `soup("li")` と省略形も使える。

---

## 属性の指定

```python
soup.find("div", attrs={"id": "main"})
soup.find_all("p", attrs={"class": "text"})
```

簡易表記も可能：

```python
soup.find("p", class_="text")  # class は Pythonの予約語なので「class_」とする
```

---

## 文字列の指定（string引数）

文字列の内容でマッチさせたいときには、`string=` を使う。

```python
soup.find("a", string="詳細はこちら")
```

部分一致ではなく完全一致であることに注意。

---

## 総合例

以下のようなHTMLを対象とした場合：

```html
<div id="main">
  <h1>タイトル</h1>
  <p class="text">本文1</p>
  <p class="text">本文2</p>
</div>
```

抽出例：

```python
soup.find("div", id="main")          # <div id="main">...</div>
soup.find_all("p", class_="text")    # [<p class="text">本文1</p>, <p class="text">本文2</p>]
soup.find("p", string="本文1")       # <p class="text">本文1</p>
```

---

## 戻り値とその扱い方

| メソッド         | 戻り値型          | 特徴           |
| ------------ | ------------- | ------------ |
| `find()`     | TagまたはNone    | 見つからなければNone |
| `find_all()` | list（Tagのリスト） | 見つからなければ空リスト |

---

## 注意点

* `.find_all()` の戻り値は必ずリストなので、ループで処理するのが基本。
* 属性は `attrs={...}` 形式でも、キーワード引数でも指定できる（複数属性の指定は後述）。
* 一致条件が厳密である点（string=）と、部分一致にしたい場合は次章以降で扱う正規表現やラムダを活用。

---

## まとめ

| 用途       | メソッド         | 戻り値例                       |
| -------- | ------------ | -------------------------- |
| 最初の1要素取得 | `find()`     | `<p>本文1</p>`               |
| 複数要素取得   | `find_all()` | `[<p>本文1</p>, <p>本文2</p>]` |
| 属性フィルタ   | `attrs=...`  | 条件付きで取得できる                 |
| テキスト一致   | `string=`    | 指定文字列と一致する要素               |

`find` / `find_all` はBeautifulSoupの中心機能であり、**タグ・属性・テキスト**のいずれでも検索可能な点が非常に強力である。

---

# 第4章 セレクタ指定（CSSセレクタ）

---

## 概要

`find()` や `find_all()` はタグ・属性ベースでの検索を提供するが、より柔軟な指定方法として**CSSセレクタ**を使う `select()` / `select_one()` が用意されている。これにより、**クラス名・ID・階層構造・属性条件**などをCSSと同様の文法で記述できる。

---

## select / select\_one の基本

| メソッド名             | 説明                     | 戻り値          |
| ----------------- | ---------------------- | ------------ |
| `select_one(sel)` | CSSセレクタに一致する**最初の1要素** | Tag または None |
| `select(sel)`     | CSSセレクタに一致する**すべての要素** | list\[Tag]   |

```python
soup.select_one("div.article")
soup.select("ul > li.active")
```

---

## クラス、ID、階層セレクタ

| セレクタ例              | 対象                                  |
| ------------------ | ----------------------------------- |
| `div`              | すべての `<div>` タグ                     |
| `.title`           | class="title" を持つすべての要素             |
| `#main`            | id="main" を持つ要素（※idは一意）             |
| `div p`            | `<div>` の子孫であるすべての `<p>`            |
| `ul > li`          | `<ul>` の**直下**にある `<li>`            |
| `section h2.title` | `<section>` 内の `<h2 class="title">` |

---

## 属性セレクタ（a\[href^="https"] など）

CSSでは属性条件を柔軟に指定できる。BeautifulSoupの `select()` でも同様の構文が使える。

| セレクタ                     | 説明                           |
| ------------------------ | ---------------------------- |
| `a[href]`                | `href` 属性を持つ `<a>` タグ        |
| `a[href^="https"]`       | `href` が "https" で始まる `<a>`  |
| `img[src$=".jpg"]`       | `src` が ".jpg" で終わる `<img>`  |
| `div[data-id="123"]`     | `data-id="123"` の `<div>` タグ |
| `input[type="checkbox"]` | チェックボックス用の `<input>`         |

---

## 総合例

対象HTML：

```html
<div id="main">
  <h2 class="title">見出し</h2>
  <ul>
    <li class="active">項目1</li>
    <li>項目2</li>
  </ul>
  <a href="https://example.com">リンク</a>
</div>
```

抽出例：

```python
soup.select_one("#main")                    # <div id="main">
soup.select("h2.title")                     # [<h2 class="title">見出し</h2>]
soup.select("ul > li.active")              # [<li class="active">項目1</li>]
soup.select('a[href^="https"]')            # [<a href="https://example.com">リンク</a>]
```

---

## 戻り値と扱い

| メソッド           | 戻り値型         | 取得対象    |
| -------------- | ------------ | ------- |
| `select()`     | list\[Tag]   | 該当する全要素 |
| `select_one()` | Tag または None | 最初の1要素  |

`select()` の結果はループで回して `.text`, `.attrs`, `.get("href")` などで個別処理するのが基本。

---

## 使用上の注意点

* `select()` は内部でCSSパーサを使っているため、タグ階層や複雑な属性指定にも強い。
* `.class` や `#id` の形式を使うため、**CSSの知識があれば直感的に使える**。
* 対象要素が1つと分かっている場合は `select_one()` を使うとよい（処理が簡潔になる）。

---

## find系との比較

| 項目    | `find()` / `find_all()` | `select()` / `select_one()` |
| ----- | ----------------------- | --------------------------- |
| 基本記法  | タグ名・属性・文字列で指定           | CSSセレクタ                     |
| 柔軟性   | 単純な検索に向く                | ネスト構造・属性条件に強い               |
| 学習コスト | Python流で習得しやすい          | CSS記法の知識が必要                 |
| 利用場面  | 汎用的なタグアクセス              | 複雑な条件付き・UI構造の模倣             |

---

## まとめ

BeautifulSoupでは `find` 系と `select` 系の両方が使えるが、**CSSセレクタを活用すると柔軟な抽出が可能**になる。HTML構造が複雑なページでは、`select()` を中心に設計するのが効率的である。

---

# 第5章 要素の中身へのアクセス

---

## 概要

要素を取得しただけでは不十分であり、**中身のテキスト・属性・親子関係などにアクセス**する必要がある。BeautifulSoupでは、取得したTagオブジェクトを使って多様な情報を取り出すことができる。

---

## テキストの取得方法（text / string / get\_text）

| アクセス方法        | 説明                                    | 使用例                             |
| ------------- | ------------------------------------- | ------------------------------- |
| `.string`     | 子要素が**テキストノード1つのみ**のときに使える            | `tag.string` → `'こんにちは'`        |
| `.text`       | 子要素すべてを**再帰的に連結**して返す                 | `tag.text` → `'こんにちは世界'`        |
| `.get_text()` | `.text`と同様だが**区切り文字やstripオプションが指定可能** | `tag.get_text(" ", strip=True)` |

### 例

```html
<p>こんにちは <b>世界</b></p>
```

```python
tag = soup.find("p")
tag.string         # None（子が複数）
tag.text           # 'こんにちは 世界'
tag.get_text(" - ")# 'こんにちは - 世界'
```

---

## 属性アクセス：`attrs` と `.get()`

Tagオブジェクトは、**辞書のように属性へアクセス**できる。

```python
a = soup.find("a")
a.attrs["href"]
a.get("href")  # 同じ意味（存在しないときは None）
```

| 方法           | 戻り値               | 補足            |
| ------------ | ----------------- | ------------- |
| `tag.attrs`  | 辞書：{属性名: 値}       | すべての属性一覧を取得   |
| `tag["属性名"]` | 値（存在しないとKeyError） | 存在確認が必要な場合に注意 |
| `tag.get()`  | 値または None         | 安全に取り出す場合に便利  |

---

## name：タグ名の取得

```python
tag.name
```

例：

```html
<h1 class="main">見出し</h1>
```

```python
tag = soup.find("h1")
tag.name  # 'h1'
```

---

## parent：親要素へのアクセス

`tag.parent` を使うと、**現在の要素の直上の親タグ**を取得できる。

```python
tag = soup.find("b")
tag.parent  # <p>...</p>
```

---

## contents / children：子要素のリスト化

| プロパティ          | 戻り値           | 特徴               |
| -------------- | ------------- | ---------------- |
| `tag.contents` | 子要素のリスト（順序あり） | テキスト・タグ含む        |
| `tag.children` | 子要素の**イテレータ** | メモリ効率が良く、ループに適する |

```python
for child in tag.children:
    print(child)
```

---

## 応用例：リンク一覧の抽出と整形

```python
for a in soup.select("a[href]"):
    text = a.get_text(strip=True)
    href = a.get("href")
    print(f"{text} -> {href}")
```

このように、**タグの属性とテキストを組み合わせて扱う**のが典型的なパターンである。

---

## まとめ

| アクセス対象 | プロパティ / メソッド                      | 備考                |
| ------ | --------------------------------- | ----------------- |
| テキスト   | `.string`, `.text`, `.get_text()` | 構造によって使い分ける       |
| 属性     | `.attrs`, `.get()`, `[...]`       | 辞書のように扱える         |
| タグ名    | `.name`                           | 'div', 'p' などの文字列 |
| 親要素    | `.parent`                         | 1階層上のタグオブジェクト     |
| 子要素    | `.contents`, `.children`          | リストとイテレータで使い分ける   |

**BeautifulSoupは「取得したTagオブジェクトをナビゲートする」思想で設計されている。**
この章で紹介した属性はその基礎であり、**HTML構造を自在にたどる力**を与えてくれる。

---

# 第6章 属性によるフィルタと抽出

---

## 概要

HTMLタグの検索において、タグ名だけでなく「属性」も重要なフィルタ条件となる。BeautifulSoupでは、属性を柔軟に指定することで、**複雑な要素の抽出**を可能にしている。

---

## 基本：`attrs={"属性名": "値"}` の形式

最も一般的な方法は、`find()` や `find_all()` に `attrs` 引数で辞書を渡す方法。

```python
soup.find("div", attrs={"class": "main"})
soup.find_all("a", attrs={"href": "/contact"})
```

これは以下のようなHTMLに対応する：

```html
<div class="main">…</div>
<a href="/contact">お問い合わせ</a>
```

---

## 簡易形式：クラスやIDの省略記法

属性名が `class` や `id` の場合、次のような省略記法が使える：

```python
soup.find("div", class_="main")  # class 属性
soup.find("span", id="title")    # id 属性
```

### 注意点

* `class` はPythonの予約語のため `class_` と書く。
* この形式は `attrs={"class": "main"}` と同等。

---

## 複数属性の同時指定

複数の属性に一致する要素だけを抽出するには、`attrs` に複数のキーを含める：

```python
soup.find("input", attrs={"type": "checkbox", "checked": "checked"})
```

対象HTML：

```html
<input type="checkbox" checked="checked" name="agree">
```

---

## 属性値がリストの場合

`class` 属性のように、スペース区切りで複数値を持つものは、BeautifulSoupが**自動的にリストとして扱う**。

```html
<div class="article main featured">内容</div>
```

```python
tag = soup.find("div", class_="main")
tag["class"]  # ['article', 'main', 'featured']
```

---

## 柔軟な条件指定：lambda関数による抽出

条件が静的に書けない場合は、`lambda` 関数を用いることで**動的条件フィルタ**が可能。

```python
soup.find_all("a", href=lambda x: x and x.startswith("/news"))
```

例：

```html
<a href="/news/2025">2025年ニュース</a>
<a href="/contact">お問い合わせ</a>
```

上記では `/news` で始まるリンクのみが対象となる。

---

## その他：部分一致や否定などの応用例

```python
# href に "login" を含むリンク
soup.find_all("a", href=lambda x: x and "login" in x)

# data-* 属性を持つタグ
soup.find_all(attrs=lambda attr: "data-" in str(attr))
```

---

## 注意点とベストプラクティス

| 方法                | 推奨度 | 備考                   |
| ----------------- | --- | -------------------- |
| `attrs={"属性": 値}` | ◎   | 一番汎用的で明示的            |
| `class_=` や `id=` | ○   | 簡潔だが、予約語に注意          |
| `lambda` を使った条件   | ◎   | 柔軟だが、複雑すぎる条件は読みづらくなる |
| 複数属性のネスト          | ◎   | 精密なフィルタに必須           |

---

## まとめ

| 機能       | 方法例                                       | 解説                          |
| -------- | ----------------------------------------- | --------------------------- |
| 単一属性指定   | `attrs={"class": "main"}`                 | 基本構文                        |
| 複数属性指定   | `attrs={"type": "submit", "value": "送信"}` | 複合条件                        |
| 簡略記法     | `class_="title"`, `id="section1"`         | 可読性向上                       |
| 動的条件     | `lambda attr: 条件式`                        | startswith, in, endswith など |
| 属性リストの扱い | `tag["class"]` → `["a", "b", "c"]`        | 自動で分解される（class専用）           |

BeautifulSoupでは属性によるフィルタ指定が非常に柔軟であり、**静的な条件も動的な条件も直感的に記述できる**。
この章を通じて、「条件に合致する要素のみを精密に抽出する技術」が身につく。

---

# 第7章 複雑な構造の走査（再帰）

---

## 概要

HTML構造は階層的かつ柔軟であり、BeautifulSoupでは取得したタグから**親・子・兄弟・全子孫ノード**などに対して簡単に移動・走査できる。これにより、構造全体を辿る再帰的な処理や、隣接要素へのアクセスが可能になる。

---

## 子孫要素をたどる：`descendants` / `children`

| メソッド           | 内容                        | 型・特徴         |
| -------------- | ------------------------- | ------------ |
| `.children`    | **直下の子要素**のみを対象           | イテレータ        |
| `.contents`    | `.children`と同じだが**リスト形式** | メモリ消費多       |
| `.descendants` | 再帰的にすべての**子孫要素**を走査       | イテレータ（再帰的処理） |

### 例：

```python
div = soup.find("div", id="main")
for child in div.children:
    print(child.name)  # div直下のタグ

for desc in div.descendants:
    print(desc.name)   # 再帰的にすべての子孫
```

---

## 兄弟要素をたどる：`next_sibling` / `previous_sibling`

| メソッド                 | 説明               |
| -------------------- | ---------------- |
| `.next_sibling`      | 次の兄弟ノード（改行や空白含む） |
| `.previous_sibling`  | 前の兄弟ノード          |
| `.next_siblings`     | 次の兄弟ノードたち（イテレータ） |
| `.previous_siblings` | 前の兄弟ノードたち（イテレータ） |

### 注意：

* 改行や空白テキストノードも兄弟と見なされるため、`.strip()` や `.name is not None` チェックが有効。

```python
tag = soup.find("li", class_="active")
next_tag = tag.find_next_sibling()  # <li>...</li>
```

---

## `next_elements` / `previous_elements`

これらは**文書全体の中での前後関係**をたどるために使う。兄弟や子孫だけでなく、**より広い範囲での走査**が可能。

| メソッド                 | 内容                |
| -------------------- | ----------------- |
| `.next_elements`     | 現在のタグ以降すべての要素・文字列 |
| `.previous_elements` | 現在のタグ以前のすべての要素    |

### 例：

```python
tag = soup.find("h2")
for elem in tag.next_elements:
    if getattr(elem, "name", None):
        print(elem.name)
```

---

## parent / parents（親要素へのアクセス）

| メソッド       | 説明                  |
| ---------- | ------------------- |
| `.parent`  | 直上の親タグ              |
| `.parents` | すべての上位親タグを辿る（イテレータ） |

```python
tag = soup.find("b")
print(tag.parent.name)          # 例: "p"

for parent in tag.parents:
    print(parent.name)
```

---

## 応用例：次の見出しに対応する段落を取得

```python
h2 = soup.find("h2", string="概要")
for sibling in h2.next_siblings:
    if sibling.name == "p":
        print(sibling.get_text(strip=True))
        break
```

このように、HTMLの構造に依存した**意味的な情報抽出**にも応用できる。

---

## まとめ

| 対象範囲   | アクセス方法                                 | 用途           |
| ------ | -------------------------------------- | ------------ |
| 直下の子   | `.children`, `.contents`               | 単純なループ処理     |
| 再帰的な子孫 | `.descendants`                         | 全ての入れ子を処理    |
| 親      | `.parent`, `.parents`                  | 所属構造を上へたどる   |
| 兄弟     | `.next_sibling`, `.previous_sibling`   | 同階層のタグを順に調べる |
| 前後全要素  | `.next_elements`, `.previous_elements` | ドキュメントの前後走査  |

HTMLの構造が複雑な場合でも、これらの走査手段を組み合わせれば、**柔軟かつ再帰的な探索や処理が可能**となる。BeautifulSoupの本質的な強みは「構造をたどる力」にある。


---

# 第8章 正規表現による検索

---

## 概要

HTMLタグや属性の値が動的だったり部分一致したりする場合、\*\*完全一致ではなく「パターンマッチング」\*\*が必要になる。
BeautifulSoupはPythonの `re` モジュール（正規表現）と連携して、柔軟な検索が可能である。

---

## 準備：`re.compile()` の基本

まず、正規表現を使うには `import re` と `re.compile("パターン")` を用いる。

```python
import re
pattern = re.compile("^https://")
```

この `pattern` を `find()` や `find_all()` に渡すことで、**正規表現に一致するタグや属性**を抽出できる。

---

## タグ名の正規表現による抽出

タグ名そのものをパターンで指定したいとき：

```python
soup.find_all(re.compile("^h[1-6]$"))
```

これは、`<h1>`〜`<h6>` タグをすべて対象とする。

---

## 属性値に対する正規表現：`attrs` に適用

```python
soup.find_all("a", href=re.compile("^https://"))
```

例：

```html
<a href="https://example.com">公式</a>
<a href="/local/page">内部リンク</a>
```

上記コードでは「https\://」で始まるリンクだけを抽出できる。

---

## 特定パターンのリンク収集（応用例）

### ニュース記事のリンクだけを取得：

```python
soup.find_all("a", href=re.compile("^/news/"))
```

対象HTML：

```html
<a href="/news/2025">2025年</a>
<a href="/blog">ブログ</a>
```

結果：`/news/2025` のみがヒット。

---

## 数値や日付パターンの抽出

```python
soup.find_all(string=re.compile(r"\d{4}/\d{2}/\d{2}"))
```

これは `2025/07/18` のような日付形式を含むテキストノードを対象とする。

---

## 複数属性のパターン指定

複数属性に正規表現を使う場合は、`attrs={...}` に複数指定する：

```python
soup.find_all("img", attrs={
    "src": re.compile(r".*\.(jpg|png)$"),
    "alt": re.compile("商品")
})
```

これは「画像形式がjpgまたはpngかつaltに“商品”を含む」画像のみを対象とする。

---

## select() との違い

`select()` はCSSセレクタの構文であり、**正規表現のような曖昧検索には向かない**。
正規表現を使いたい場合は、`find` / `find_all` を用いるのが基本。

---

## 組み合わせ：lambda × 正規表現

```python
soup.find_all("a", href=lambda x: x and re.search("login|signin", x))
```

これは、`href` に "login" または "signin" を含む `<a>` タグを抽出する。

---

## まとめ

| 対象範囲    | 使用例                                    | 効果                   |
| ------- | -------------------------------------- | -------------------- |
| タグ名     | `find_all(re.compile("^h[1-6]$"))`     | `<h1>`〜`<h6>` をすべて取得 |
| 属性値     | `href=re.compile("^https")`            | "https"で始まるリンク       |
| テキスト    | `string=re.compile("特定語句")`            | 含むテキストノード全体          |
| 複合条件    | `attrs={"src": re.compile(".*\.jpg")}` | jpg画像の抽出             |
| lambda式 | `lambda x: re.search(...)`             | より柔軟な条件式             |

BeautifulSoupと正規表現を組み合わせることで、**固定されていないパターンにも強いスクレイピング**が可能になる。
不規則・可変なHTMLに対応するための重要な技術である。

---

# 第9章 コメント・スクリプト・改行の扱い

---

## 概要

HTML内には人間向けの**コメント**や、解析に不要な**script / styleタグ**、意味のない**空白・改行**などが含まれている。
BeautifulSoupはこうしたノイズを適切に処理できる仕組みを備えており、**構造化・整形されたデータの抽出**において重要な役割を果たす。

---

## コメントノードの扱い：`Comment` クラス

HTMLのコメント（`<!-- -->`）は `bs4.Comment` オブジェクトとして扱われる。

```html
<!-- これはコメントです -->
```

```python
from bs4 import Comment

for c in soup.find_all(string=lambda t: isinstance(t, Comment)):
    print(c)  # 'これはコメントです'
```

### 注意点：

* `soup.string` で取得できる文字列の中にコメントが混ざることがある。
* `Comment` を除外したい場合は `not isinstance(t, Comment)` でフィルタする。

---

## script / style タグの除去

これらは**見た目には存在するが、スクレイピングでは不要**な場合が多い。

```python
for tag in soup(["script", "style"]):
    tag.decompose()
```

| メソッド           | 説明                   |
| -------------- | -------------------- |
| `.decompose()` | タグ本体ごと削除してメモリから除去    |
| `.extract()`   | タグを取り出して返す（任意で再利用可能） |

---

## 改行タグ（`<br>`）と空行の扱い

HTMLでは改行を `<br>` で表現することが多いが、解析時にはそのままだと扱いにくい。必要に応じて改行に置き換えるとよい。

```python
for br in soup.find_all("br"):
    br.replace_with("\n")
```

また、テキスト整形時に `.get_text("\n", strip=True)` を使うと、改行を保持しつつ不要な空白を除去できる。

---

## 空白ノードや空行の除去

BeautifulSoupでは、HTMLの中の空白文字列（例: `'\n  '`）も**要素間に存在するテキストノード**として扱われる。

```python
for e in soup.stripped_strings:
    print(e)
```

これにより、`"\n"` や `" "` などを無視し、**実質的な文字列のみを取り出す**ことができる。

---

## 応用：本体抽出前の「前処理」としてのノイズ除去

BeautifulSoupで情報を抽出する前に、以下のような処理を行うと品質が高まる：

```python
# コメントとscript/styleの削除
for tag in soup(["script", "style"]):
    tag.decompose()

for comment in soup.find_all(string=lambda t: isinstance(t, Comment)):
    comment.extract()
```

---

## まとめ

| 対象要素         | 処理方法                     | 効果                     |
| ------------ | ------------------------ | ---------------------- |
| コメント         | `isinstance(t, Comment)` | コメント除去・収集              |
| script/style | `tag.decompose()`        | ノイズ除去（画面には表示されるが不要な要素） |
| brタグ         | `br.replace_with("\n")`  | 改行テキストへの置換             |
| 空白ノード        | `soup.stripped_strings`  | 意味のある文字列だけを抽出          |

HTMLは視覚的には自然でも、構造的には冗長なノードが多く含まれる。
**不要な情報を除去し、意味ある部分だけを残す前処理**がデータ抽出の精度を高める鍵となる。


---

# 第10章 要素の追加と変更

---

## 概要

BeautifulSoupは読み取り専用ではなく、**HTMLツリーを編集・構築する機能**も備えている。
この章では、**タグの追加・テキストや属性の変更・要素の挿入**などを通じて、HTML構造を動的に生成・改変する方法を扱う。

---

## 新しいタグの作成：`soup.new_tag()`

新たなタグを生成するには、`new_tag()` を使用する。

```python
new_tag = soup.new_tag("p")
new_tag.string = "新しい段落"
```

属性を指定することも可能：

```python
link = soup.new_tag("a", href="https://example.com", target="_blank")
link.string = "リンク"
```

---

## タグの追加：`append()` / `insert()`

### `append()`

あるタグの**末尾**に新しいタグを追加する。

```python
div = soup.find("div", id="main")
div.append(new_tag)
```

### `insert()`

特定のインデックスに挿入できる。

```python
div.insert(0, new_tag)  # 最初に追加
```

---

## 特定位置への挿入：`insert_after()` / `insert_before()`

兄弟ノードとして隣接位置に追加したいときは以下を使う：

```python
target = soup.find("h2")
target.insert_after(new_tag)
```

または：

```python
target.insert_before(new_tag)
```

---

## 属性の変更・追加・削除

### 変更・追加

```python
tag["class"] = "highlight"
tag["data-id"] = "123"
```

### 削除

```python
del tag["class"]
```

属性の存在確認も可能：

```python
if "href" in tag.attrs:
    print(tag["href"])
```

---

## テキストの変更

既存タグのテキスト内容は `.string` や `.string.replace_with()` で変更できる。

```python
tag.string = "更新された内容"
```

または：

```python
tag.string.replace_with("差し替えテキスト")
```

---

## 応用：空の `<ul>` に `<li>` を動的に追加

```python
ul = soup.new_tag("ul")
for i in range(3):
    li = soup.new_tag("li")
    li.string = f"項目{i+1}"
    ul.append(li)

soup.body.append(ul)
```

出力：

```html
<ul>
  <li>項目1</li>
  <li>項目2</li>
  <li>項目3</li>
</ul>
```

---

## 変更後の出力：`str(soup)` / `soup.prettify()`

編集されたHTML全体を再出力するには以下を用いる。

```python
html = str(soup)          # コンパクトなHTML文字列
formatted = soup.prettify()  # インデント付き表示
```

---

## まとめ

| 操作内容    | メソッド                                 | 補足                 |
| ------- | ------------------------------------ | ------------------ |
| 新しいタグ作成 | `soup.new_tag("tag")`                | 任意の属性・テキストも指定可     |
| 要素の追加   | `append()`, `insert()`               | 子として末尾または任意位置に挿入可能 |
| 兄弟ノード挿入 | `insert_after()`, `insert_before()`  | 位置関係を保った編集ができる     |
| 属性の操作   | `tag["key"] = val`, `del tag["key"]` | 動的に変更・削除           |
| テキストの操作 | `.string = ...`, `.replace_with()`   | 書き換えや差し替え          |

BeautifulSoupは単なるスクレイピングツールではなく、**HTMLドキュメントを自在に構築・変形できる編集ツール**でもある。
本章で紹介した操作により、HTMLテンプレートの生成や整形にも柔軟に対応できるようになる。

---