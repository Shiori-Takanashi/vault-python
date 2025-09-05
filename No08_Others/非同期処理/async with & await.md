# ❓質問
 > async/withは、awaitが無くても成立するか？

# ❗回答
> はい、`async with` は **`await` が無くても構文上は成立します**。  
> ただし、以下の点に注意が必要です。

---

### ■ `async with` の本質

`async with` は、**非同期コンテキストマネージャ**の `__aenter__()` / `__aexit__()` を `await` 付きで呼び出します。

したがって、ブロックの中に `await` が無かったとしても：

- `__aenter__()` と `__aexit__()` 自体は自動的に `await` される
    
- 構文としては成立する（SyntaxError にはならない）
    

---

### ■ 例：ブロック内で `await` を使わない場合

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        print("非同期セッションは開いたが、awaitは使わない")

asyncio.run(main())
```

これは**実行可能**。ただし：

- 実質的に `ClientSession` を使って何もしていない
    
- リソースを開いて即座に閉じているだけの無意味な処理になる
    

---

### ■ 想定される実用ケース（まれ）

```python
class DummyAsyncContext:
    async def __aenter__(self):
        print("enter")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("exit")

async def main():
    async with DummyAsyncContext():
        print("中身で await は使わない")

asyncio.run(main())
```

これも正常に動く。**ただし `await` が無い＝非同期処理を一切していないだけ**。

---

### ■ 結論

- 成立：**する**
    
- 意味：**あるとは限らない**
    
- 典型的には `await` を伴う非同期処理を書くために使う
    

構文上は問題ないが、実用的には `await` がない `async with` は基本的に無意味と見なされることが多い。