# 📢`Locator`

---

## 📍概要

`Locator` は、**DOM内の要素を抽象的に参照するためのオブジェクト**です。
CSSセレクタやXPathなどによって対象を指定し、後からその要素群に対して操作や検査を実行できます。

* 実際の要素取得は**遅延評価（Lazy）**
* `locator()` は**毎回DOMを再スキャン**（自動再試行あり）
* `element_handle()` とは異なり、**変化に強い**

---

## 📍 オブジェクトの取得方法

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    
    # 取得例
    title_locator = page.locator("h1")
```

* `page.locator("セレクタ")`
* `locator.nth(i)`：同一セレクタで複数該当した場合のn番目（0-based）
* `locator.locator("子セレクタ")`：親から子への連鎖

---

## 📍 主要メソッド

Locatorは基本的に**すべてメソッド形式**で情報を取得します。
プロパティではなく、**非同期動作を含む操作**が主であり、メソッド呼び出しが必須です。

### 📄 要素の情報を取得する

| メソッド                   | 説明                            |
| ---------------------- | ----------------------------- |
| `.inner_text()`        | テキスト（見た目の内容）取得                |
| `.inner_html()`        | 内部HTML文字列取得                   |
| `.text_content()`      | 隠れているテキストも含めた取得               |
| `.get_attribute(name)` | 属性値の取得（例：`href` など）           |
| `.input_value()`       | `<input>`の現在の値を取得             |
| `.is_visible()`        | 表示状態かどうか                      |
| `.is_hidden()`         | 非表示かどうか                       |
| `.is_checked()`        | `<input type="checkbox">` の状態 |
| `.is_enabled()`        | 有効状態かどうか                      |
| `.is_disabled()`       | 無効状態かどうか                      |

---

### 📄 要素に操作する（クリック・入力・選択など）

| メソッド                    | 説明               |
| ----------------------- | ---------------- |
| `.click()`              | クリック             |
| `.dblclick()`           | ダブルクリック          |
| `.fill(value)`          | 入力値をセット          |
| `.type(text)`           | キーボードで打つように入力    |
| `.press("Enter")`       | 特定キーを押す（Enterなど） |
| `.check()`              | チェックボックスをON      |
| `.uncheck()`            | チェックボックスをOFF     |
| `.select_option(value)` | `<select>`要素の選択  |
| `.hover()`              | ホバー操作            |
| `.focus()`              | フォーカスを当てる        |

---

### 📄 要素数・ループ・階層処理

| メソッド                    | 説明               |
| ----------------------- | ---------------- |
| `.count()`              | 対象要素の数を返す        |
| `.nth(i)`               | n番目の要素（0-index）  |
| `.locator("child")`     | 子要素へのlocatorチェイン |
| `.first()` / `.last()`  | 最初 / 最後の要素を取得    |
| `.filter(has_text="x")` | テキストに基づく絞り込み     |

---

### 📄 待機・同期処理関連

| メソッド                            | 説明                                                   |
| ------------------------------- | ---------------------------------------------------- |
| `.wait_for()`                   | 要素が出現・可視になるのを待つ                                      |
| `.wait_for_element_state()`     | `visible`, `hidden`, `attached`, `detached` のいずれかを待つ |
| `.scroll_into_view_if_needed()` | 画面内にスクロール                                            |

---

## 📍 `element_handle()` との違い

| 特徴          | `Locator` | `ElementHandle` |
| ----------- | --------- | --------------- |
| 自動再試行       | ✅ あり      | ❌ なし            |
| 推奨度         | ✅ 高（公式推奨） | ❌ 非推奨（基本不要）     |
| DOM変化に対する耐性 | ✅ 高い      | ❌ 脆弱            |

---

