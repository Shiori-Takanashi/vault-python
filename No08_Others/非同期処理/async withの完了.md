# ❓質問
>  async withは、awaitが完了したら閉じるのか？

# ❗回答
> いいえ、**`async with` は `await` の完了ではなく、「`async with` ブロックを抜けた時点」で自動的にリソースを閉じます**。`await` の完了だけでは閉じません。

---

### 例：`async with` と `await` の関係

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        print("セッション開始")
        response = await session.get("https://example.com")
        print("リクエスト完了")
        await response.text()
        print("本文取得完了")
    print("セッション閉じた")

asyncio.run(main())
```

このとき：

- `await session.get(...)` が終わっても、**まだ `session` は有効**
    
- `async with` ブロックが終わった（`print("本文取得完了")` の後）、**そのタイミングで `session.__aexit__()` が呼ばれ、セッションが閉じられる**
    

---

### 要点の整理

|タイミング|セッションの状態|
|---|---|
|`await session.get(...)` の後|開いたまま|
|`async with` ブロック内|常に有効|
|`async with` ブロックを抜けた時点|自動的に閉じる|

---

### 技術的には

- `__aenter__()` でリソースを開き
    
- `__aexit__()` が `await` 付きで呼ばれ、非同期的にクローズ処理が行われる
    

---

したがって、**`await` が完了してもブロックを抜けない限り、`async with` の管理対象は開いたまま**です。リソース解放を保証するには、ブロックのスコープを正しく設計する必要があります。