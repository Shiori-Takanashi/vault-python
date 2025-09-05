## Thread クラス

```python
class Thread:
    def __init__(self, target=None, args=(), kwargs=None):
        # 実行する関数、引数、キーワード引数を保持
        self._target = target
        self._args = args
        self._kwargs = kwargs or {}

    def start(self):
        # _thread モジュールで新しいスレッドを開始し、_bootstrap メソッドを実行
        _thread.start_new_thread(self._bootstrap, ())

    def _bootstrap(self):
        # 保持していた関数を引数付きで呼び出す（スレッド内で実行）
        if self._target:
            self._target(*self._args, **self._kwargs)
```

---

