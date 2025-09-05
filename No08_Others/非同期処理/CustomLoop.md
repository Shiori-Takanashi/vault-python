**カスタムイベントループの作成は通常の用途では不要**であり、**ライブラリ実装者・フレームワーク開発者向け**の高度な領域です。
ただし、Python標準APIで「明示的にカスタムループを作成して使う」最低限の構造は非常に簡潔です。

---

## ✅ カスタムイベントループの基本例

```python
import asyncio

async def job():
    await asyncio.sleep(1)
    print("実行完了")

# 新しいイベントループを作成
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)

try:
    loop.run_until_complete(job())
finally:
    loop.close()
```

---

## ✅ 要点

* `new_event_loop()`：既存とは独立した新しいイベントループを生成
* `set_event_loop(loop)`：`asyncio.get_event_loop()` でこのループを取得できるように登録
* `run_until_complete(...)`：ループを1回だけ実行し、`job()` の完了を待つ
* `loop.close()`：終了後、リソースを明示的に解放

---

## ✅ 使い所（実務）

* 他のスレッドやプロセスで別のイベントループを使いたいとき
* 複数のループを使い分けたいとき（例：GUIループ vs asyncio）
* ライブラリ内で `get_event_loop()` を汚染せずに、隔離されたループを使いたいとき

---

## ✅ 通常用途では？

```python
asyncio.run(main())
```

この一行で済む。**カスタムループは原則不要**。使うのは特殊な設計・制御が必要なケースのみ。
