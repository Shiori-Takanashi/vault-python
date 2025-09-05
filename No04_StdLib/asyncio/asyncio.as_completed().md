
# ✅ `asyncio.as_completed()` 

---

## 概要

`asyncio.as_completed()` は、**複数の非同期タスクを並列に実行し、「完了したものから順に処理する」ためのジェネレータ関数**です。

例えば、10件のAPIを非同期でリクエストして「応答が早い順に処理したい」場合、通常の `gather()` ではすべてが終わるのを待ちますが、`as_completed()` を使えば**先に終わったものから順に1件ずつ取得・処理**できます。これはリアルタイム性が求められる処理や、**途中で中断したい処理**との相性が非常に良いです。

---

## 基本構文とポイント

```python
for coro in asyncio.as_completed(awaitables):
    result = await coro
```

* `awaitables`：非同期処理（コルーチン or `Task`）のリスト
* `as_completed()` はイテレータであり、非同期イテレータ（`async for`）ではないため、**通常の `for` 文で良い**（中の `coro` は `await` 必須）
* 戻り値は**非同期オブジェクトのジェネレータ**で、完了した順に順次返される

---

## 使用例：完了した順に出力する

```python
import asyncio
import random

async def worker(n):
    await asyncio.sleep(random.uniform(0.1, 1.0))
    return f"Worker-{n} completed"

async def main():
    tasks = [worker(i) for i in range(5)]
    for coro in asyncio.as_completed(tasks):
        result = await coro
        print(result)

asyncio.run(main())
```

### 実行結果（順不同）例：

```
Worker-3 completed
Worker-0 completed
Worker-2 completed
Worker-4 completed
Worker-1 completed
```

→ **完了した順に出力される**。並列実行された各タスクが終わり次第、その都度 `await coro` によって結果が取り出されている。

---

## 他の手段との比較：`gather()` との違い

| 比較項目       | `asyncio.gather()`   | `asyncio.as_completed()`        |
| ---------- | -------------------- | ------------------------------- |
| 結果の順序      | 入力した順に結果が並ぶ          | **完了した順に処理される**                 |
| 処理の開始      | 同時に開始                | 同時に開始                           |
| 結果の取得      | 一括で結果を得る（全タスク完了後）    | **1件ずつ逐次取得可能（完了順）**             |
| キャンセルや打ち切り | 不可（全完了まで待つ）          | **任意の件数だけ取得し、残りを `cancel()` 可** |
| 用途         | 全件同時にまとめて処理・集計が必要な場面 | 応答の早いものから順に反応・打ち切りたい場面          |

---

## 実践例：最初の3件だけ使い、残りはキャンセル

```python
import asyncio
import random

async def worker(n):
    await asyncio.sleep(random.uniform(0.1, 1.0))
    return f"Task {n}"

async def main():
    tasks = [asyncio.create_task(worker(i)) for i in range(10)]
    completed = []
    for i, coro in enumerate(asyncio.as_completed(tasks)):
        result = await coro
        completed.append(result)
        if i == 2:  # 最初の3件で十分
            break
    for t in tasks:
        t.cancel()  # 残りはキャンセル
    print("First 3 results:", completed)

asyncio.run(main())
```

→ **早く終わった3件だけ使い、残りの負荷を削減**。Webスクレイピングやログ監視など「最初の成功だけで十分」なケースに有効。

---

## 注意点と設計上の留意事項

* `as_completed()` の戻り値はイテレータであり、**一度使い切ると再利用不可**
* 渡すコルーチンは `Task` にしておくとキャンセルや完了状態の管理がしやすい
* エラー処理は各 `await coro` に `try-except` を明示的に書く必要がある
* `asyncio.wait()` のように `done`, `pending` に分ける機能はない

---

## 補足：他の選択肢との適材適所

| 関数                       | 主な特徴                             |
| ------------------------ | -------------------------------- |
| `asyncio.gather()`       | 全ての結果を順序保証つきで一括取得                |
| `asyncio.as_completed()` | 終了順に1件ずつ取り出し可能、**途中終了可能**        |
| `asyncio.wait()`         | `done`, `pending` を分離でき、柔軟な制御が可能 |

---

## 結論

`asyncio.as_completed()` は、**非同期タスク群から「終了順に」逐次処理するための最も柔軟な関数**であり、特に以下のような場面に最適です：

* 応答の早いものから処理し、レイテンシを最小化したい
* 一部の結果だけで十分なので、残りを途中で打ち切りたい
* 長時間かかる処理を一括せず、都度処理・途中出力したい

---

