# ✅ `asyncio.Semaphore()` 

## 概要

`asyncio.Semaphore` は、**非同期タスクの同時実行数を制限するための同期プリミティブ**です。多数のタスクが存在する状況で「一度に動かせる数を制限したい」場面に使用されます。

---

## ✅ 本質的な意味

> **「同時に実行してよい非同期タスク数の上限を設定する、非同期用の通行証」**

同期処理における `threading.Semaphore` の非同期版であり、**リソース保護・接続制限・APIレート制御**において不可欠です。

---

## ✅ 基本構文と説明

```python
sem = asyncio.Semaphore(3)

async def task():
    async with sem:
        await do_something()
```

* `Semaphore(n)`：同時に `n` 件まで `async with` の中へ入れる
* `async with sem:`：**スロットを取得し、使い終わったら自動で返却**
* `sem.acquire()` / `sem.release()` も使用可能（非推奨）

---

## ✅ 具体例：同時実行3件に制限

```python
import asyncio
import random

sem = asyncio.Semaphore(3)

async def worker(n):
    async with sem:
        print(f"Start {n}")
        await asyncio.sleep(random.uniform(0.5, 1.5))
        print(f"End   {n}")

async def main():
    await asyncio.gather(*(worker(i) for i in range(10)))

asyncio.run(main())
```

### 出力イメージ（時間は概略）：

```
Start 0
Start 1
Start 2
End   1
Start 3
End   0
Start 4
...
```

→ 最大3つまでしか同時に `worker()` の内部に入れない。**セマフォが空くまで他のタスクは待機**する。

---

## ✅ 使用目的とユースケース

| 分類             | 詳細                                               |
| -------------- | ------------------------------------------------ |
| 外部APIレート制限     | 一度にアクセス可能なリクエスト数（例：最大5接続）に制限                     |
| データベース接続制御     | 同時接続数の上限（例：同時に20件以上処理するとDBが重くなる）                 |
| 同時アクセスが重い処理の制御 | ファイルI/O・Webスクレイピング・GPU演算など、**並列にやりすぎると逆に遅くなる**処理 |

---

## ✅ よくある誤解と注意点

| 誤解                            | 実際                                                          |
| ----------------------------- | ----------------------------------------------------------- |
| `Semaphore(n)` を使えば勝手に制御される   | ❌ いいえ、**`async with` などで明示的に囲う必要がある**                       |
| スレッドセーフである                    | ❌ いいえ、**スレッドセーフではない**。マルチスレッド用途には `threading.Semaphore` を使用 |
| `Semaphore(1)` は `Lock()` と同じ | 概ね同じ動作だが、**意味的には違う**（`Lock` は明示的に「排他」を意図している）               |

---

## ✅ BoundedSemaphore との違い

* `Semaphore(n)`：release の回数に制限なし（バグで `n+1` 回 release しても例外にならない）
* `BoundedSemaphore(n)`：**release の回数が `n` を超えると例外**

→ 正確なスロット管理が求められる場合は `BoundedSemaphore` の方が安全。

---

## ✅ `Lock()` との比較

| 特徴項目    | `asyncio.Lock()` | `asyncio.Semaphore(n)` |
| ------- | ---------------- | ---------------------- |
| 同時に入れる数 | 最大1件             | 最大n件                   |
| 用途      | **排他制御（mutex）**  | **並列数制御（rate limit）**  |
| 使用例     | ログ書き込み、カウンタ更新等   | 外部API呼び出し、画像処理バッチ等     |

---

## ✅ 明示的に制御する場合の例

```python
sem = asyncio.Semaphore(2)

await sem.acquire()
try:
    await critical_section()
finally:
    sem.release()
```

→ `async with` を使わない形式。**中断や例外処理の設計上、あえて明示的に使う場合もある**。

---

## ✅ 結論

`asyncio.Semaphore()` は、\*\*非同期プログラムにおける「同時実行数の管理」\*\*を実現するための基本かつ重要な機構です。

* ✅ 非同期処理の過剰実行を防ぎ、安定性・パフォーマンスを保つ
* ✅ `async with` による簡潔なスロット管理が可能
* ✅ `Lock` や `gather` とは異なる、**数量制限という独自の役割**

---

