### ✅ `asyncio.wait()` とは？

> **複数の非同期タスクを待機し、完了したタスクと未完了のタスクに分けて返す関数**です。
> `gather()` よりも**柔軟な制御**ができ、**完了順やタイムアウト処理に向いています**。

---

## ✅ 一文で言えば：

> **「全部 or 一部の非同期タスクが終わるのを監視し、状態を判別したいときに使う関数」**。

---

## ✅ 使用例：基本

```python
import asyncio

async def wait_and_return(n):
    await asyncio.sleep(n)
    return n

async def main():
    tasks = [asyncio.create_task(wait_and_return(i)) for i in range(1, 4)]
    done, pending = await asyncio.wait(tasks)

    for task in done:
        print("完了:", task.result())

asyncio.run(main())
```

出力（最大3秒待った後）：

```
完了: 1
完了: 2
完了: 3
```

---

## ✅ 戻り値の構造

```python
done, pending = await asyncio.wait(tasks)
```

* `done`: 完了済みの `Task` の集合（set）
* `pending`: まだ終わってない `Task` の集合（set）

---

## ✅ `gather()` との違い

| 比較       | `gather()`  | `wait()`                  |
| -------- | ----------- | ------------------------- |
| 結果の形式    | 結果リストを返す    | `done` / `pending` セットを返す |
| 順序       | ✅ 入力順を保つ    | ❌ 完了順・未完了を分離（順序不定）        |
| 完了待機条件   | **全部終わるまで** | **指定した条件で途中終了も可能**        |
| タイムアウト対応 | 一部あり        | ✅ `timeout` 引数あり（強力）      |

---

## ✅ 完了条件の制御：`return_when`

```python
await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
```

| 値                       | 意味             |
| ----------------------- | -------------- |
| `ALL_COMPLETED` (デフォルト) | 全部終わるまで待つ      |
| `FIRST_COMPLETED`       | 最初に1つでも終わったら返る |
| `FIRST_EXCEPTION`       | 最初の例外が出たら返る    |

---

## ✅ タイムアウト付きの例

```python
done, pending = await asyncio.wait(tasks, timeout=2)

for t in done:
    print("完了:", t.result())

for t in pending:
    print("未完了 → キャンセル")
    t.cancel()
```

---

## ✅ よく使うケース

| シーン          | 内容                      |
| ------------ | ----------------------- |
| 一部だけ早く返したい   | 10タスク中、最初の1つが完了したら処理したい |
| タイムアウト付き並列処理 | 10秒で終わらなければキャンセル        |
| 状態監視         | 終了／例外／未完了の内訳を知りたい       |

---

## ✅ まとめ

> `asyncio.wait()` は、**複数のタスクを実行・監視し、「完了したもの」と「まだ動いてるもの」に分類するための関数**。
> `gather()` よりも **タイムアウト・中途終了・状態管理に優れた柔軟な制御**が可能。

---

必要であれば、`as_completed()` との違いや、`wait` を使った高度な監視構成も解説できます。希望があればどうぞ。
