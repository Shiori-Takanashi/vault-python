
# ✅ `asyncio.create_task()` の詳細解説

## 概要

`asyncio.create_task()` は、**非同期関数（コルーチン）を「Task」としてイベントループに登録し、即時に実行を開始する関数**です。単に `await coro()` と書く場合と異なり、**複数の非同期処理を並列に走らせる**ためには `create_task()` のように明示的な並列化が必要です。

---

## コアの役割

> `create_task()` は、**コルーチンをスケジューラが管理する実行可能な「Task」に変換する仕組み**です。

Pythonにおけるコルーチン（`async def`）は、単体では実行されません。明示的に `await` するか、`create_task()` に渡してイベントループ上に「実行すべき対象」として登録する必要があります。

---

## ✅ 基本構文

```python
import asyncio

async def say(msg, sec):
    await asyncio.sleep(sec)
    print(msg)

async def main():
    task1 = asyncio.create_task(say("Hello", 1))
    task2 = asyncio.create_task(say("World", 2))
    await task1
    await task2

asyncio.run(main())
```

### 実行結果：

```
Hello
World
```

→ 1秒後に "Hello" が出力され、さらに1秒後に "World" が出力される。2つのタスクは**並列に実行されている**。

---

## ✅ `await` との違い

```python
await coro1  # 実行が終わるまで他の処理をブロック
```

```python
task = asyncio.create_task(coro1)
# → 登録後すぐに次の処理に進む（非同期的に実行される）
```

### 結論：

* `await`: 順次処理（直列）
* `create_task`: 並列処理（同時実行）

---

## ✅ 典型的な使用パターン

### 並列で複数の処理を開始して、最後にまとめて回収する

```python
import asyncio

async def fetch_data(n):
    await asyncio.sleep(n)
    return f"done {n}"

async def main():
    tasks = [asyncio.create_task(fetch_data(i)) for i in [1, 2, 3]]
    results = await asyncio.gather(*tasks)
    print(results)

asyncio.run(main())
```

→ すべての `fetch_data()` が**同時に開始され**、最大でも3秒で終了する。

---

## ✅ `create_task()` が便利なケース

| ユースケース                    | 解説                                          |
| ------------------------- | ------------------------------------------- |
| ログ送信やメトリクス送信などの「後回しでよい」処理 | メイン処理を止めずに背後で非同期処理を実行                       |
| 再帰的な非同期処理                 | 呼び出し先がまた `create_task()` を呼ぶことで非同期タスクツリーを作成 |
| タイムアウト制御やキャンセル設計          | Task にしておくと `.cancel()` などで明示的に中断・制御できる     |

---

## ✅ よくある誤りと対策

### ❌ ただコルーチンを作っただけでは動かない

```python
say("hi", 1)  # これだけでは何も起こらない
```

→ コルーチンは生成しただけでは **未実行のオブジェクト**。`await` するか、`create_task()` に渡さない限り無視される。

---

### ❗ Task を参照しないと警告

```python
asyncio.create_task(my_coro())
# ❌ 参照も `await` もせずに放置 → 警告が出る
```

* `RuntimeWarning: coroutine was never awaited`
* Python 3.8以降、参照なしのタスクは「放置されている」と見なされ警告が出る
* 少なくとも変数に格納する or `.add_done_callback()` を指定すること

---

## ✅ 補助関数との連携

| 関数名                | 用途例                       |
| ------------------ | ------------------------- |
| `asyncio.gather()` | 複数 Task を一括で待つ            |
| `task.cancel()`    | 実行中タスクを明示的にキャンセル          |
| `task.done()`      | タスクが終了しているか確認             |
| `task.result()`    | 完了後の戻り値を取得（未完了状態で呼ぶと例外）   |
| `asyncio.wait()`   | 終了・未終了を分けて集めたいときに便利       |
| `asyncio.shield()` | Task をキャンセルの対象から守りたいときに使用 |

---

## ✅ 複雑な応用：途中キャンセル付きタスク実行

```python
async def long_task(n):
    await asyncio.sleep(n)
    return f"done {n}"

async def main():
    task = asyncio.create_task(long_task(5))
    await asyncio.sleep(1)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task was cancelled")

asyncio.run(main())
```

→ 長時間かかるタスクを明示的にキャンセルし、例外を補足して処理。

---

## ✅ 結論

`asyncio.create_task()` は、**非同期処理を並列に開始したい場面で必須の低レベルAPI**です。以下のような設計に向いています：

* `await` のブロッキングを避けたい
* 複数の非同期処理を同時に走らせたい
* 背景で処理しつつ、別の処理を進めたい
* 明示的に `.cancel()` や `.result()` を制御したい

この関数を理解することで、**Python の非同期プログラミングは一気に柔軟性と表現力を増します**。

---
