# ❓質問
> asyncio.sleep()とは？

# ❗回答
>  `asyncio.sleep()` は、**非同期関数内で一定時間だけ処理を中断する関数**です。
>  同期の `time.sleep()` と異なり、**イベントループをブロックせずに待機**できます。

---

## ✅ 構文

```python
await asyncio.sleep(seconds)
```

* `seconds`：浮動小数でもOK（例：`0.1` 秒）
* `await` なしでは意味がない

---

## ✅ 具体例

```python
import asyncio

async def main():
    print("処理開始")
    await asyncio.sleep(2)
    print("2秒経過")

asyncio.run(main())
```

出力：

```
処理開始
（2秒後）
2秒経過
```

この間、イベントループは**他の非同期タスクを自由に動かせる**。

---

## ✅ `time.sleep()` との違い

| 比較      | `time.sleep()`    | `asyncio.sleep()`      |
| ------- | ----------------- | ---------------------- |
| 待機中の動作  | **完全に停止**（ブロッキング） | **他のタスクが進行**（ノンブロッキング） |
| 使用場所    | 同期関数              | `async def` の中         |
| イベントループ | 停止する              | 停止しない                  |

---

## ✅ 使用場面

* **API連続呼び出しの間引き**
* **処理のポーリング**
* **負荷分散のための意図的な待機**

---

## ❌ 間違った使い方（非await）

```python
async def foo():
    asyncio.sleep(1)  # ❌ await していないため無意味

# 修正:
async def foo():
    await asyncio.sleep(1)  # ✅
```

---

## ✔ まとめ

> `asyncio.sleep()` は「イベントループを止めずに待機する」ための非同期用の sleep。
> `await` 付きで使わないと効果がなく、`time.sleep()` の非同期版と覚えるとよい。
> 実運用では、API呼び出しの間隔調整などで非常によく使われます。

---
