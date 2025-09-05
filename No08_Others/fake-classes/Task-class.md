## Task クラス

```python
class Task(Future):
    def __init__(self, gen, loop):
        super().__init__()
        self._gen = gen  # 非同期処理を表すジェネレータオブジェクト
        self._loop = loop  # 実行に使うイベントループ
        # 最初のステップをイベントループに登録して開始
        self._loop.call_soon(lambda: self._step(None))

    def _step(self, value):
        # ジェネレータに値を送信し、次のFutureを取得
        try:
            fut = self._gen.send(value)
        except StopIteration as e:
            # ジェネレータが終了した場合、結果をセット
            self.set_result(getattr(e, "value", None))
        else:
            # 次のFutureが完了したら、_wakeupをイベントループに登録
            fut.add_done_callback(lambda f: self._loop.call_soon(lambda: self._wakeup(f)))

    def _wakeup(self, fut):
        # 完了したFutureから値を受け取り、次のステップへ
        self._step(fut.get_result())
```

---

