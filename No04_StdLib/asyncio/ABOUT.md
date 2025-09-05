# 📢 `asyncio` の主なメソッド・関数

---

## 📍 イベントループ制御関連

| 関数・メソッド                         | 説明                             |
| ------------------------------- | ------------------------------ |
| `asyncio.get_event_loop()`      | 現在のスレッドのイベントループを取得（非推奨: 3.10+） |
| `asyncio.get_running_loop()`    | 実行中のイベントループを取得                 |
| `asyncio.new_event_loop()`      | 新しいイベントループを生成                  |
| `asyncio.set_event_loop(loop)`  | イベントループを現在のスレッドに設定             |
| `loop.run_until_complete(coro)` | コルーチンが完了するまでループを実行             |
| `loop.run_forever()`            | 永久にイベントループを実行                  |
| `loop.stop()`                   | `run_forever()` の停止            |
| `loop.close()`                  | イベントループの明示的なクローズ（以降使用不可）       |

---

## 📍 タスク・コルーチン関連

| 関数・メソッド                                           | 説明                    |
| ------------------------------------------------- | --------------------- |
| `asyncio.create_task(coro)`                       | 新たなタスクを生成しスケジューリング    |
| `asyncio.ensure_future(coro_or_future)`           | タスクかFutureを返す（レガシー互換） |
| `asyncio.gather(*coros, return_exceptions=False)` | 複数のコルーチンを並列実行して全結果を返す |
| `asyncio.wait(tasks)`                             | タスクの完了待ち（状態に応じた監視）    |
| `asyncio.wait_for(coro, timeout)`                 | タイムアウト付きでコルーチンを待機     |
| `asyncio.shield(coro)`                            | タイムアウトなどによるキャンセルを防止   |

---

## 📍 Future / Task オブジェクト

| オブジェクト                                | 主な用途                             |
| ------------------------------------- | -------------------------------- |
| `asyncio.Future()`                    | 明示的な将来の値（通常は自作しない）               |
| `asyncio.Task()`                      | 実行中の非同期処理。`create_task()` によって生成 |
| `.done()` / `.cancel()` / `.result()` | タスク・Futureの状態取得／キャンセル／結果取得       |

---

## 📍 非同期 I/O 関連

| 関数・メソッド                                       | 説明                         |
| --------------------------------------------- | -------------------------- |
| `asyncio.open_connection(host, port)`         | TCP接続の確立（reader, writer返す） |
| `asyncio.start_server(client_cb, host, port)` | TCPサーバーを起動                 |
| `asyncio.open_unix_connection(path)`          | UNIXソケットへの接続               |
| `asyncio.start_unix_server()`                 | UNIXドメインソケットサーバーの起動        |

---

## 📍 非同期同期処理（同期的なwait制御）

| 関数・メソッド                            | 説明                   |
| ---------------------------------- | -------------------- |
| `asyncio.sleep(seconds)`           | 非同期でのスリープ（ブロッキングしない） |
| `asyncio.to_thread(func, *args)`   | 同期関数を別スレッドで非同期実行     |
| `loop.run_in_executor(exec, func)` | executor上で同期関数を非同期化  |

---

## 📍 ロック・同期プリミティブ

| クラス名                       | 説明                |
| -------------------------- | ----------------- |
| `asyncio.Lock()`           | 非同期用のロック（with構文可） |
| `asyncio.Semaphore(value)` | 同時アクセス制限に使うセマフォ   |
| `asyncio.Event()`          | 通知用イベント           |
| `asyncio.Condition()`      | 条件付き待機に使用（ロックと併用） |
| `asyncio.Queue()`          | 非同期対応のキュー（FIFO）   |

---

## 📍 タイマー・遅延実行

| 関数・メソッド                            | 説明                  |
| ---------------------------------- | ------------------- |
| `loop.call_later(delay, callback)` | 指定秒数後に関数を実行（非async） |
| `loop.call_at(time, callback)`     | 絶対時間で実行             |
| `loop.call_soon(callback)`         | すぐにコールバック実行         |

---

## 📍 キャンセル・タイムアウト処理

| 名前                                   | 説明            |
| ------------------------------------ | ------------- |
| `asyncio.TimeoutError`               | タイムアウト例外      |
| `asyncio.CancelledError`             | キャンセル例外       |
| `task.cancel()`                      | タスクをキャンセルする   |
| `try/except asyncio.CancelledError:` | キャンセル処理の明示的捕捉 |

---
