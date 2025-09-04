
# ✅ `asyncio.Lock()` 
---

## 概要

`asyncio.Lock()` は、**非同期処理における排他制御（mutual exclusion）を実現するためのプリミティブ**です。複数の非同期タスクが同じリソースにアクセスする際、「同時に入れるのは1つだけ」に制限するために使用します。

---

## ✅ 一言でまとめると：

> **「1人ずつしか入れない非同期用の鍵」**
> 他のタスクは鍵が空くまで順番に待機する。

---

## ✅ 基本構文

```python
import asyncio

lock = asyncio.Lock()

async def critical_section(i):
    async with lock:
        print(f"Task {i} entered")
        await asyncio.sleep(1)
        print(f"Task {i} exited")

async def main():
    await asyncio.gather(*(critical_section(i) for i in range(3)))

asyncio.run(main())
```

### 出力（順番に実行）：

```
Task 0 entered
Task 0 exited
Task 1 entered
Task 1 exited
Task 2 entered
Task 2 exited
```

→ **常に1つのタスクだけがロック内に入っている**

---

## ✅ 仕組みと動作

1. `lock = asyncio.Lock()`：ロックインスタンスを生成
2. `async with lock:`：ロックを取得して処理に入る（取得済みなら待機）
3. 処理終了後：ロックが自動解放され、次の待機タスクが実行される

---

## ✅ よくある用途

| ケース                 | 解説                                   |
| ------------------- | ------------------------------------ |
| ログファイルへの同時書き込み防止    | 出力が混ざるのを防ぎ、**順番通りに書き込ませる**           |
| 状態変数や共有カウンタの更新      | **競合状態を防止**。加算・減算などで同時実行されると壊れる操作を守る |
| 単一リソースの制御（ソケット・接続等） | 同時に使えるのが1つだけの外部リソースを保護               |

---

## ✅ 明示的な制御パターン

```python
await lock.acquire()
try:
    await do_something()
finally:
    lock.release()
```

* `async with` を使わない場合、**例外補足して必ず `release()` を呼ぶこと**
* `release()` を忘れると **ロックが永久に開放されず全体停止する**

---

## ✅ よくある誤解とその訂正

| 誤解                           | 実際                                          |
| ---------------------------- | ------------------------------------------- |
| `asyncio.Lock` はスレッドセーフである   | ❌ いいえ、**スレッド間の制御には `threading.Lock` を使うべき** |
| `asyncio.Lock()` は即座に効果を発揮する | ❌ 実際は `async with` で囲まないと効果なし               |
| 非同期関数以外でも使える                 | ❌ 非同期 (`async def`) の中でのみ利用可能（同期関数では動作しない）  |

---

## ✅ `Semaphore` との比較

| 比較点   | `asyncio.Lock()` | `asyncio.Semaphore(n)` |
| ----- | ---------------- | ---------------------- |
| 同時実行数 | 常に1（排他）          | 最大n（制限付き並列）            |
| 用途    | リソースの排他制御        | タスクの同時実行数の制限           |
| 概念的比喩 | 鍵付きの個室           | 椅子がn脚ある待合室             |

---

## ✅ 応用例：ログ出力制御

```python
log_lock = asyncio.Lock()

async def safe_log(msg):
    async with log_lock:
        print(msg)
```

→ ログ出力が **他のタスクの出力と混ざらないようにするためにロック**をかける。

---

## ✅ 注意点

* `Lock` を **過度に使いすぎると性能が低下**する（ボトルネックになる）
* 必ず **`async with` で囲んで、自動的に開放させる**設計が望ましい
* `acquire()` 後に `release()` を忘れると、デッドロックになる

---

## ✅ 結論

`asyncio.Lock()` は、**非同期プログラミングにおける「1タスクだけがアクセスできる状態」を保証する基本的ツール**です。

* ✅ 排他制御が必要な箇所に `async with lock:`
* ✅ 非同期環境における `threading.Lock` の代替
* ✅ 状態破壊や同時出力などのリスクを抑えるために不可欠

---

