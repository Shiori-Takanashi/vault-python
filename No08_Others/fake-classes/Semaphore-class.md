
## Semaphore クラス

```python
class Semaphore:
    def __init__(self, max_permits):
        # 同時に許可する処理数の上限を設定
        self._max = max_permits
        self._count = 0  # 現在の取得数
        self._waiters = []  # 待機中のコールバック
        self._lock = _thread.allocate_lock()  # 排他制御用ロック

    def acquire(self, callback):
        # リソースの取得を試みる（非同期にコールバックを用いて行う）
        def try_acquire():
            with self._lock:
                if self._count < self._max:
                    # 上限未満なら即時取得成功、カウント増加してコールバック実行
                    self._count += 1
                    callback()
                else:
                    # 上限に達していれば、待機キューに追加
                    self._waiters.append(callback)
        try_acquire()

    def release(self):
        # リソースの解放処理（他の待機者がいれば即時再取得を試みさせる）
        with self._lock:
            self._count -= 1
            if self._waiters:
                # 待機キューの先頭を取り出し、即時再取得を許可
                next_cb = self._waiters.pop(0)
                self._count += 1  # 取得とみなして再カウント
                next_cb()
```

---

