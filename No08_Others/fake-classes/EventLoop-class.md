## EventLoop クラス

```python
class EventLoop:
    def __init__(self):
        # コールバックキューの初期化（Deque を使用）
        self._queue = Deque()

    def call_soon(self, cb):
        # 指定されたコールバック関数をすぐ実行するためのキューに追加
        self._queue.append(cb)

    def run_until_all_complete(self, tasks):
        # 指定されたタスク群のすべてが完了するまでイベントループを回す
        while any(not t._is_completed for t in tasks):
            if self._queue:
                # キューに何かあれば、先頭のコールバックを取り出して実行
                cb = self._queue.popleft()
                cb()
            else:
                # キューが空なら少し待機してCPU使用率を下げる
                time.sleep(0.01)
```

---
