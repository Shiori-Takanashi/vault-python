以下に、`asyncio` の中核的・実用的で**まだ紹介していない主要メソッド・関数**を列挙します（Python 3.12 時点での情報に基づく）。**標準的なアプリ開発・ツール作成で実用性が高いもの**に絞っています。

---

## ✅ 未紹介のメジャー関数・メソッド一覧

| 関数・メソッド名                                       | 概要                                                                                |
| ---------------------------------------------- | --------------------------------------------------------------------------------- |
| `asyncio.create_task(coro)`                    | コルーチンをイベントループに登録して即実行される `Task` にする（並列実行の起点）                                      |
| `asyncio.gather(*tasks)`                       | 複数の `await` を並列に実行し、すべての結果をまとめて返す                                                 |
| `asyncio.wait(tasks)`                          | `gather` に似るが、完了順を制御したり、タイムアウト管理が柔軟                                               |
| `asyncio.as_completed(tasks)`                  | 複数タスクを **完了した順にイテレート**できる（先着順で処理したいとき）                                            |
| `asyncio.sleep(delay)`                         | 指定時間だけ非同期で待機（非ブロッキングな `time.sleep()`）                                             |
| `asyncio.run_coroutine_threadsafe(coro, loop)` | 他スレッドからイベントループにコルーチンを安全に登録・実行（マルチスレッドで有用）                                         |
| `asyncio.TimeoutError`                         | タイムアウト時に投げられる例外（例：`wait_for()`）                                                   |
| `asyncio.wait_for(coro, timeout)`              | コルーチンにタイムアウトを設定して実行（失敗時に `TimeoutError`）                                          |
| `asyncio.Queue()`                              | 非同期対応の FIFO キュー（`async with`, `await put()`, `await get()`）で、プロデューサ・コンシューマ間の通信に使う |
| `asyncio.Event()`                              | 非同期で使えるイベント通知オブジェクト（`.set()`, `.clear()`, `await .wait()`）                        |
| `asyncio.Semaphore(n)`                         | 非同期処理の同時実行数を制限（すでに登場済み）                                                           |
| `asyncio.Lock()`                               | 単一リソースの排他制御に使う非同期ロック（`async with lock:`）                                          |
| `asyncio.StreamReader`, `StreamWriter`         | `asyncio.open_connection()` などで使う TCP ソケット通信の非同期ストリーム制御（低レベル通信）                   |
| `asyncio.subprocess` モジュール                     | 非同期でサブプロセスを起動・制御（`create_subprocess_exec` など）                                     |

---

## ✅ 使用頻度の高いものを分類すると：

### ◾ 並列実行制御系

* `create_task()`
* `gather()`
* `wait()`
* `as_completed()`
* `wait_for()`

### ◾ 並行同期ツール

* `Queue`
* `Event`
* `Lock`
* `Semaphore`

### ◾ タスク管理・例外処理

* `TimeoutError`
* `run_coroutine_threadsafe()`

### ◾ 通信系（ソケット・プロセス）

* `open_connection()`, `start_server()`
* `create_subprocess_exec()`

---

## ✔ 補足

これらは「標準的な非同期プログラミング」を行う上で、ほぼ必須レベルの API 群です。
「1対多の非同期処理」「並列I/O」「制御構造の同期化」などの際に活用されます。

---

必要であれば、各関数の**ユースケース付き実践例**、または**最低限知っておくべき10選**などにも整理可能です。希望があればお申し付けください。
