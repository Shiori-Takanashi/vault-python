## Future クラス

```python
class Future:
    def __init__(self):
        # 非同期処理の完了状態を保持する
        self._is_completed = False
        # 処理結果の保持用
        self._result_value = None
        # 処理完了時に呼ばれるコールバック関数群
        self._completion_callbacks = []

    def set_result(self, value):
        # 非同期処理の完了と結果を設定し、登録されたコールバックをすべて呼び出す
        self._is_completed = True
        self._result_value = value
        for cb in self._completion_callbacks:
            cb(self)

    def add_done_callback(self, cb):
        # 既に完了していれば即時呼び出し、未完了ならリストに追加
        if self._is_completed:
            cb(self)
        else:
            self._completion_callbacks.append(cb)

    def get_result(self):
        # 結果を返す（完了していない場合でも返すので注意）
        return self._result_value

    def __iter__(self):
        # ジェネレータで `yield` により待機可能にし、最終的に結果を返す
        yield self
        return self.get_result()
```

---

`Future` クラス単体は「非同期処理の完了通知」と「結果の受け渡し」に使える基本ユニットです。イベントループやタスクと組まずとも、以下のような「別スレッドで処理し、完了後にコールバックを呼ぶ」仕組みは実装可能です。

---

### 例：Future だけを使って非同期HTTPリクエスト

```python
def async_request(url):
    fut = Future()

    def run():
        try:
            res = requests.get(url)
            fut.set_result(res.text)
        except Exception as e:
            fut.set_result(f"[ERROR] {e}")

    _thread.start_new_thread(run, ())
    return fut

# ==============================

# 使用例
f = async_request("https://example.com")

def on_done(fut):
    print("レスポンスの一部:", fut.get_result()[:200])

f.add_done_callback(on_done)

# メインスレッドが即終了しないように待機
while not f._is_completed:
    time.sleep(0.1)
```

---

### 説明

* `async_request()` は `Future` を返し、スレッド内で非同期に `requests.get` を実行。
* 完了時に `set_result()` で結果を通知。
* `add_done_callback()` により、結果取得後の処理を別関数に委譲可能。

---

このように、`Future` だけでも「非同期の戻り値を一時的に保管し、完了時に通知する」用途に使えます。ただし、複数同時処理や順次処理には別途スケジューリング（EventLoopやTask）が必要になります。希望があれば、`Future` 単体を使った他のユースケースも提案可能です。
