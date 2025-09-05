
# ✅ `asyncio.new_event_loop()` 
---
## 概要

`asyncio.new_event_loop()` は、**新しいイベントループインスタンス（`EventLoop` オブジェクト）を手動で作成するための関数**です。この関数は通常の `asyncio.run()` のような高レベルAPIとは異なり、**自前でループを構築・登録・破棄する責任**が生じます。

これは、以下のような特殊・高度なユースケースに対応するための「低レベルAPI」と位置付けられます。

---

## ✅ 本質的な役割

> **「このスレッド用に、まだ誰にも使われていないまっさらなループを1つ作って、自由に制御したい」**
> そういう場面でのみ使うべき関数です。

---

## ✅ 最小構成の使用例

```python
import asyncio

loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
loop.run_until_complete(asyncio.sleep(1))
loop.close()
```

* `new_event_loop()`：ループのオブジェクトを作成（**動いていない状態**）
* `set_event_loop()`：現在のスレッドに割り当て（重要）
* `run_until_complete()`：ループを起動して1回の処理を実行
* `close()`：必ず明示的に終了処理を行う必要がある

---

## ✅ `asyncio.run()` との違い（構造比較）

| 比較項目        | `asyncio.run()`            | `asyncio.new_event_loop()`         |
| ----------- | -------------------------- | ---------------------------------- |
| ループの作成・登録   | 自動                         | 手動                                 |
| 実行処理の開始     | `run(main())` だけでOK        | 明示的に `run_until_complete()` が必要    |
| 終了後のクリーンアップ | 自動的に `loop.close()` まで行われる | 手動で `loop.close()` しないとメモリリーク等の原因に |
| 用途          | 1回だけコルーチンを安全に動かす（高レベル）     | カスタム制御・マルチスレッドなど（低レベル）             |

---

## ✅ 使用例①：スレッドごとに独立したループを持たせる

```python
import asyncio
import threading

def worker():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(asyncio.sleep(1))
    loop.close()

thread = threading.Thread(target=worker)
thread.start()
thread.join()
```

* Python の `asyncio` では、**スレッドごとにループが1つだけ存在**可能
* `asyncio.run()` はメインスレッド専用に近く、**スレッドで動かすには `new_event_loop()` が必須**

---

## ✅ 使用例②：REPL・Jupyter Notebook でのループ再生成

```python
# 一部の古いJupyterでは get_event_loop() が壊れてる場合がある
import asyncio

loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
loop.run_until_complete(asyncio.sleep(0.5))
loop.close()
```

→ Jupyter環境ではイベントループが「既に存在」していたり「壊れている」ケースがあり、**再生成による回避が必要になる**

---

## ✅ 使用例③：並行して複数の非同期処理ループを走らせたい場合（特殊設計）

※ 非推奨だが可能

```python
def create_isolated_loop():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    return loop

loop1 = create_isolated_loop()
loop2 = create_isolated_loop()
```

* 本来、1スレッドに1ループが原則
* **開発ツールやテスト基盤などで特殊な使い方をすることがある**

---

## ✅ 注意点（重要）

| 注意項目                       | 理由                                        |
| -------------------------- | ----------------------------------------- |
| `set_event_loop()` を必ず行うこと | **作成しただけでは `get_event_loop()` 等が機能しない**   |
| `loop.close()` を忘れるとメモリリーク | 終了後もイベントループが開いたままになり、ファイルディスクリプタが残る可能性あり  |
| 例外処理を含む構成を設計すること           | loop 実行中に例外が発生しても `close()` が必ず呼ばれる構成にすべき |

---

## ✅ よくある誤解とその訂正

| 誤解                           | 実際の動作と注意点                                                |
| ---------------------------- | -------------------------------------------------------- |
| `new_event_loop()` を呼べばすぐ使える | ❌ いいえ、`set_event_loop()` を明示的に行う必要がある                    |
| どんな状況でも複数ループを作ってよい           | ❌ 基本的に**1スレッド＝1ループ**が原則（マルチスレッドで使い分けるべき）                 |
| `asyncio.run()` の代わりにいつでも使える | ❌ `run()` は自動制御つきの高水準API。**`new_event_loop()` は低レベルAPI** |

---

## ✅ まとめ

> `asyncio.new_event_loop()` は、**イベントループを自分で生成・管理したいときのための低レベル関数**です。
> 通常の `asyncio.run()` で足りないとき、またはマルチスレッドなど特殊な状況で、**ループの完全な制御が必要な場合にのみ使用すべきです**。

---

## ✅ 実務的原則

* メインスレッドでは基本的に `asyncio.run()` を使うこと
* `new_event_loop()` の使用は次のようなケースに限定：

  * スレッドごとのループ管理
  * REPL/Jupyterなどの壊れたループ再設定
  * ライブラリ内部で明示的なループを切り替える必要があるとき

---

