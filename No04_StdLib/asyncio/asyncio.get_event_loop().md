
# ✅ `asyncio.get_event_loop()` 
---

## 概要

`asyncio.get_event_loop()` は、**現在のスレッドに紐づくイベントループを取得する古い関数**です。Python 3.6 以前では非同期処理の開始に必須でしたが、**Python 3.7 以降では基本的に非推奨**です。

---

## ✅ 一言でまとめると

> 「**今このスレッドで動いてる asyncio の中枢（イベントループ）を取ってくる**関数」
> ただし今は基本的に `asyncio.run()` を使うべきです。

---

## ✅ 基本構文と動作

```python
import asyncio

loop = asyncio.get_event_loop()
loop.run_until_complete(some_coroutine())
```

この書き方は Python 3.6 までの**非同期処理の起点**として多用されました。しかし現在では以下のように書くのが一般的です：

```python
asyncio.run(some_coroutine())
```

→ `run()` はイベントループの生成、実行、クリーンアップを **すべて自動で行う安全な関数**。

---

## ✅ 実行例：旧来の使い方

```python
import asyncio

async def hello():
    await asyncio.sleep(1)
    print("hello")

loop = asyncio.get_event_loop()
loop.run_until_complete(hello())
```

このコードは Python 3.6 では正常に動作します。
しかし Python 3.10 以降では、**特定条件下で `RuntimeError` を起こします**。

---

## ✅ なぜ非推奨になったのか？

### 問題点：

* **ループが「自動である」か「手動である」かが分かりにくい**
* 複数回呼び出すと予期しない挙動をする（例：停止済みループを再利用しようとしてエラー）
* **非同期関数の設計とループの制御が混ざる**ことで、設計が複雑化する
* **新しいスレッドではループが存在しないため、明示的に作成しないと例外になる**

---

## ✅ Python 3.10+ でのエラー例

```python
import asyncio

loop = asyncio.get_event_loop()
# ❌ RuntimeError: There is no current event loop in thread 'MainThread'.
```

→ **イベントループがまだ明示的に作成されていないスレッドでは使えない**

---

## ✅ 推奨される代替手段

| 目的                      | 推奨される関数                      | 解説                       |
| ----------------------- | ---------------------------- | ------------------------ |
| 非同期処理を走らせるだけ            | `asyncio.run(coro)`          | 推奨。ループの生成・開始・停止を自動で行う    |
| `async def` 内で現在のループを取得 | `asyncio.get_running_loop()` | `await` 中の関数内でのみ安全に取得できる |
| 明示的にループを作成              | `asyncio.new_event_loop()`   | マルチスレッド用途など特殊な状況で必要      |

---

## ✅ 各手段の使い分けと意義

| 関数名                  | 意味と使用例                 |
| -------------------- | ---------------------- |
| `get_event_loop()`   | 旧式、REPLや互換コードで一時的に使う   |
| `get_running_loop()` | `async def` 内部でのみ使用可   |
| `new_event_loop()`   | スレッドやユニットテスト用に新規ループを作成 |
| `asyncio.run()`      | 最も安全で推奨されるエントリーポイント    |

---

## ✅ 現代的な書き方との比較

| 旧構文（非推奨）                                                         | 現構文（推奨）            |
| ---------------------------------------------------------------- | ------------------ |
| `loop = asyncio.get_event_loop()`<br>`loop.run_until_complete()` | `asyncio.run(...)` |

---

## ✅ 具体例：`get_event_loop()` が必要になるケース

### 1. REPL（インタラクティブ実行環境）

```python
>>> import asyncio
>>> loop = asyncio.get_event_loop()
>>> loop.run_until_complete(async_func())
```

* `asyncio.run()` は REPL では使えないため、今も `get_event_loop()` に依存する

---

### 2. `threading.Thread` 上での非同期処理

```python
def worker():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(my_coro())

thread = threading.Thread(target=worker)
thread.start()
```

* スレッドにはデフォルトでイベントループが存在しない
* `new_event_loop()` + `set_event_loop()` の組み合わせが必要

---

## ✅ 結論

`asyncio.get_event_loop()` は、Python 非同期処理の**初期時代の中心的関数**でしたが、設計上のあいまいさや例外の起こりやすさから、現在では以下のように整理されます：

* ✅ **使うべきではない**（Python 3.7以降）
* ✅ `asyncio.run()` を使う
* ✅ 特殊用途（REPL・スレッド）以外では避けるべき
* ✅ 後方互換性のためだけに存在している

この関数の存在は、非同期処理の**過渡期の設計的妥協**の名残と位置づけられます。

---

