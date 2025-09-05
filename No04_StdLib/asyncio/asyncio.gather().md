
# ✅ `asyncio.gather()`

## 概要

`asyncio.gather()` は、**複数の非同期処理（コルーチン or Task）を一括で並列実行し、そのすべてが完了するのを待つ関数**です。最も基本的かつ汎用的な非同期タスク集約手段であり、**逐次処理を明示的に並列化し、処理時間を短縮する**際に有効です。

---

## ✅ 一文でまとめると：

> `gather()` は「いくつかの `await` を**同時に走らせて**、\*\*全部終わったらまとめて結果を返す」関数。

---

## ✅ 基本構文と例

```python
import asyncio

async def process(n):
    await asyncio.sleep(n)
    return f"done {n}"

async def main():
    results = await asyncio.gather(
        process(1),
        process(2),
        process(3)
    )
    print(results)

asyncio.run(main())
```

### 実行結果（約3秒後）：

```
['done 1', 'done 2', 'done 3']
```

* 各 `process()` は**並列に開始される**
* 最も遅いもの（この例では3秒）が終わるまで `gather()` はブロック
* 結果は **渡した順に** 並んで返る（完了順ではない）

---

## ✅ 特徴と内部挙動の整理

| 特徴            | 説明                                                                |
| ------------- | ----------------------------------------------------------------- |
| 並列開始          | 渡された全てのコルーチンは即時に並列開始                                              |
| 結果の順序保証       | 完了順ではなく**引数の順序通りに結果を返す**                                          |
| 失敗時の伝播        | **どれか一つでも例外を raise すれば、全体が例外となる**（`return_exceptions=True` で回避可能） |
| キャンセル時の挙動     | 外部から `gather()` をキャンセルすると、中の全 Task もキャンセルされる                      |
| 例外を戻り値として受け取る | `return_exceptions=True` を指定すると、例外オブジェクトがそのまま結果に含まれる              |

---

## ✅ gather vs wait の比較

| 比較観点     | `gather()`         | `wait()`                          |
| -------- | ------------------ | --------------------------------- |
| 結果の返却    | ✅ 結果リストを返す         | ❌ `done`, `pending` の `Future` 集合 |
| 順序の保持    | ✅ 引数の順序で保持         | ❌ 完了順や順不同                         |
| エラー処理    | ❌ raise される（デフォルト） | ✅ タスクごとに例外が含まれる                   |
| 柔軟なキャンセル | ❌ 中断されると全部止まる      | ✅ 柔軟な制御が可能                        |

---

## ✅ 例外処理付きの gather（return\_exceptions）

```python
async def may_fail(n):
    if n % 2 == 0:
        raise ValueError(f"Fail {n}")
    await asyncio.sleep(0.1)
    return n

async def main():
    results = await asyncio.gather(
        *(may_fail(i) for i in range(5)),
        return_exceptions=True
    )
    for i, res in enumerate(results):
        if isinstance(res, Exception):
            print(f"Task {i} failed: {res}")
        else:
            print(f"Task {i} succeeded: {res}")

asyncio.run(main())
```

→ このようにすることで、**例外を全体中断せず個別処理**できる。サーバアクセス・ファイル操作など、部分的失敗が許容される場面に適する。

---

## ✅ `create_task()` + `gather()` の併用

```python
tasks = [asyncio.create_task(process(i)) for i in range(5)]
results = await asyncio.gather(*tasks)
```

この構成の利点：

* `create_task()` によって明示的に「並列に走る Task」として登録
* `gather()` によってその Task 群を一括で同期的に回収

このように、**「いつ走らせるか」と「いつ待つか」を分離できる**点が、`create_task()` との併用の最大のメリット。

---

## ✅ 実務における使用例

| シチュエーション            | 内容                                                     |
| ------------------- | ------------------------------------------------------ |
| 複数APIを同時に叩きたい       | 応答の早い遅いにかかわらず、**全リクエストを同時発行して待つ**                      |
| 画像ファイルを非同期で一括ダウンロード | `aiohttp`, `aiofiles` などで並列化し、ディスクI/OやネットI/Oの遅延を隠蔽     |
| センサ監視や複数デバイスの並行処理   | 同時にデータ取得 → 全部終わってから一括集計（順序保証が重要）                       |
| 並列処理の結果を UI やログに反映  | `gather()` で順序制御しながら取りまとめ → 並列処理でも UI に一定順序で出力したいときに便利 |

---

## ✅ gather() の落とし穴と注意点

* **1つでも例外があると即座に全体停止する（return\_exceptions=False の場合）**
* 一部だけ結果が必要な場合や、途中終了したい場合には向かない → `as_completed()` を使う
* 結果順が**タスクの終了順ではない**ため、**応答順で処理したいなら gather() は不適**

---

## ✅ 結論

`asyncio.gather()` は、**「全部同時に処理して、結果を順番に取りまとめたい」場面における最適解**です。以下のような特性を理解したうえで活用することが求められます：

* 複数の非同期処理を**一括管理**したい（進捗に依らずまとめて処理）
* 結果の**順序が重要**である（入力の順番を保持）
* 中断や例外処理の設計も考慮している（`return_exceptions=True` の使用を含む）

この関数は、非同期処理において「直列から並列への移行」を自然に行えるため、学習の初期段階でも登場頻度が高く、設計理解の軸となるべき関数です。

---
