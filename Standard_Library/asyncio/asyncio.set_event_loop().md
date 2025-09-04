### ✅ `asyncio.set_event_loop()` とは？

> **現在のスレッドに、新たなイベントループを関連付ける関数**です。
> 主に `asyncio.new_event_loop()` で作成したループを、**明示的に「このスレッド用」に設定するために使います**。

---

## ✅ 一文で言えば：

> **「このスレッドで使うイベントループはこれだ」と宣言する関数。**

---

## ✅ 使用例（典型）

```python
import asyncio

loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
loop.run_until_complete(asyncio.sleep(1))
loop.close()
```

---

## ✅ なぜ必要？

| なぜ？                              | 理由                                                               |
| -------------------------------- | ---------------------------------------------------------------- |
| `get_event_loop()` で現在のループを取得したい | `set_event_loop()` をしないと、`get_event_loop()` は `RuntimeError` を返す |
| 複数スレッドでそれぞれ独立したループを使いたい          | 各スレッドに **手動で紐づける必要がある**（デフォルトでは None）                            |

---

## ✅ スレッドと組み合わせた例

```python
import threading
import asyncio

def run():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(asyncio.sleep(1))
    loop.close()

thread = threading.Thread(target=run)
thread.start()
thread.join()
```

---

## ✅ `set_event_loop()` が必要になる場面

| 場面                            | 説明                                                                          |
| ----------------------------- | --------------------------------------------------------------------------- |
| 新しいループを作ったとき                  | `new_event_loop()` は単体では無所属なので設定が必要                                         |
| 別スレッドで使うとき                    | 各スレッドに対してループを個別に設定する必要がある                                                   |
| `get_event_loop()` を使う | `get_event_loop()` が `RuntimeError` を出す（3.10以降） |

---

## ✅ 補足：`asyncio.run()` との関係

* `asyncio.run()` は内部で自動的に：

  1. `new_event_loop()` でループを作成し、
  2. `set_event_loop()` で現在のスレッドに紐づけて、
  3. 実行・閉鎖する

---

## ✅ まとめ

> **`asyncio.set_event_loop()` は、「このスレッドではこのイベントループを使え」と登録する関数。**
> `new_event_loop()` との併用が基本で、特に**マルチスレッド環境での非同期処理**には必須です。

---

必要であれば、`get_event_loop_policy()` や `loop.set_task_factory()` といったイベントループの高度な制御手段も解説可能です。希望があればどうぞ。
