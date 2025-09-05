### ✅ イベントループとは？（非同期処理における）

> Python の非同期処理（`asyncio` モジュール）において、イベントループは「**コルーチンやタスクを管理し、実行タイミングを制御する中心的な仕組み**」を指します。

---

## ✅ 一文で言えば

> イベントループは「**非同期処理をスケジューリングし、IO待ちやスリープ中のコルーチンを適切なタイミングで再開する機構**」です。

---

## ✅ 具体例

```python
import asyncio

async def say_hello():
    await asyncio.sleep(1)
    print("Hello, world!")

# 1) イベントループを取得または生成
loop = asyncio.get_event_loop()

# 2) コルーチンをタスク化してスケジューリング
task = loop.create_task(say_hello())

# 3) イベントループを回し、コルーチンを実行（完了まで待機）
loop.run_until_complete(task)

# 4) イベントループを閉じる
loop.close()
```

* `get_event_loop()`：既存のイベントループを取得（ない場合は新規作成）。
* `create_task(...)`：コルーチンをタスク化して、イベントループの管理下に乗せる。
* `run_until_complete(...)`：指定したタスクが完了するまで、イベントループを稼働させる。
* 結果として、1秒後に `"Hello, world!"` が表示される。

---

## ✅ 詳細解説

### イベントループの役割

* **コルーチンのスケジューリング**

  * `await` や `sleep()`、ソケットIO、ファイルIOなどの「待ち状態」に入ったコルーチンを一旦中断し、他の処理に CPU を回す。
  * 待ちが解除されたタイミングで、該当コルーチンを「実行可能状態」に戻す。

* **タスク管理**

  * `asyncio.create_task()` や `asyncio.gather()` などで生成されたタスクを一元的に管理し、「いつ実行すべきか」を判断する。
  * キャンセル、例外伝播、戻り値の取得といった制御もここで行う。

* **コールバックの実行**

  * コールバック関数（`call_soon()`, `call_later()` など）を登録し、指定されたタイミングで呼び出す。
  * タイマー処理や、将来のイベントをスケジューリングする機能を提供する。

---

### イベントループの基本操作

1. **ループの取得／生成**

   * `loop = asyncio.get_event_loop()`：既存のループを返す（なければ新規作成）。
   * `loop = asyncio.new_event_loop()`：強制的に新しいループを得たいときに使う。

2. **タスクの登録**

   * `loop.create_task(coro)`：コルーチンを「Task オブジェクト」としてスケジューリングする。
   * `asyncio.ensure_future(coro)`：タスク化と同等の処理を行うユーティリティ関数。

3. **ループの実行**

   * `loop.run_until_complete(future)`：指定した `Future` または `Task` が完了するまでループを走らせる。
   * `loop.run_forever()`：明示的に `stop()` が呼ばれるまでループを継続的に走らせる。

4. **コールバック登録**

   * `loop.call_soon(callback, *args)`：次回ループサイクルの際に `callback(*args)` を実行する。
   * `loop.call_later(delay, callback, *args)`：`delay` 秒後に `callback(*args)` を実行する。

5. **ループの停止／クローズ**

   * `loop.stop()`：`run_forever()` を停止し、イベントループを抜ける。
   * `loop.close()`：内部リソースを解放し、ループそのものを閉じる。

---

### 典型的な書き方

最近の Python では、上記のように手動でループを操作するよりも、シンプルに以下のように書くケースがほとんどです。

```python
import asyncio

async def main():
    # 複数コルーチンを並列実行して結果を待つ例
    async def task1():
        await asyncio.sleep(1)
        return "Task1 完了"

    async def task2():
        await asyncio.sleep(2)
        return "Task2 完了"

    results = await asyncio.gather(task1(), task2())
    print(results)

# いちばん外側で asyncio.run() を呼ぶだけ
asyncio.run(main())
```

* `asyncio.run(main())` の内部で、

  1. 新しいイベントループを作成
  2. `main()` コルーチンを実行
  3. すべてのタスクが終わると自動的にループをクローズ
     という手順が行われる。
* そのため、手動で `get_event_loop()` や `run_until_complete()` を扱う必要がなく、ほとんどのケースで十分。

---

## ✅ まとめ

* **イベントループは非同期処理の“心臓部”であり、コルーチンやタスクの実行タイミングを制御する**。
* `asyncio.run()` を使えば、内部でイベントループの作成～実行～クローズまでを自動で行ってくれる。
* 手動でループを扱う場合は、`get_event_loop()`, `create_task()`, `run_until_complete()`, `call_soon()/call_later()` などの API を駆使して細かく制御できる。
* `asyncio.run()` で十分なケースがほとんどだが、フレームワークやサーバー実装などで「カスタムループを生成／再利用」したい場合には手動操作が必要になる。

このように、イベントループを理解・活用することで、複数の IO 処理を効率よく並列実行したり、高度なタイマー制御を行ったりする非同期プログラミングが可能になります。
