
# ✅ `asyncio.get_running_loop()` 詳解

---

## 概要

`asyncio.get_running_loop()` は、**「現在動作中のイベントループ（EventLoop）」を安全かつ明示的に取得する唯一の公式手段**です。Python 3.7 以降で導入され、それ以前の `get_event_loop()` にあった曖昧さを排除しています。

---

## ✅ 要点まとめ

> **「イベントループが“今まさに実行中”のときだけ取得できる」関数**
> コードの外側や同期関数内では使えず、**非同期関数内限定で信頼できるループ取得手段**です。

---

## ✅ 基本使用例

```python
import asyncio

async def main():
    loop = asyncio.get_running_loop()
    print("現在のイベントループ:", loop)

asyncio.run(main())
```

### 出力（例）：

```
現在のイベントループ: <_UnixSelectorEventLoop running=True closed=False debug=False>
```

* `asyncio.run()` によりイベントループが起動
* `main()` 実行中に `get_running_loop()` を呼ぶことで、**今動いているループ**を取得できる

---

## ✅ 内部動作と保証

* 呼び出し時点で「**稼働中のループがなければ `RuntimeError` を例外として送出**」
* `loop` が取得できた場合、**それは必ず現在のスレッドで動いている正規のイベントループ**

この「明示性」が非同期設計において非常に重要です。

---

## ✅ `get_event_loop()` との明確な違い

| 比較項目       | `get_event_loop()`     | `get_running_loop()`         |
| ---------- | ---------------------- | ---------------------------- |
| ループが無い時の挙動 | 仮のループを勝手に作って返す（危険）     | 例外を投げる（`RuntimeError`）       |
| 実行中かの検査    | 行われない（止まっていても返る）       | 必ず「実行中のループ」でなければエラーになる       |
| 用途の推奨範囲    | REPL やレガシーコードなど互換性の維持用 | Python 3.7以降の**設計に適合する標準手段** |

---

## ✅ 誤用例とそのエラー

```python
# 非同期関数の外で呼ぶと例外になる
loop = asyncio.get_running_loop()
```

### 発生する例外：

```
RuntimeError: no running event loop
```

→ この関数は、**「既にイベントループが実行中」の状態でしか呼んではならない**
つまり、**`asyncio.run()` の外では使えない**

---

## ✅ 正しい使い所と活用場面

### 1. 非同期ライブラリ内部で「現在のループを取得」したい

```python
loop = asyncio.get_running_loop()
loop.create_task(my_coro())
```

* 呼び出し元がどのようなループであれ、**その場で動いているループを安全に取得**

---

### 2. スレッド実行（`run_in_executor()`）に必要なループを得る

```python
def blocking_io():
    import time
    time.sleep(1)
    return "完了"

async def main():
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(None, blocking_io)
    print(result)

asyncio.run(main())
```

* `run_in_executor()` はループのメソッドであるため、**現在のループの参照が必要**
* 非同期処理中に**同期処理を安全に回す**定番パターン

---

### 3. イベントループに属する各種リソース操作（例：コネクション、ハンドラ）

```python
loop = asyncio.get_running_loop()
server = await loop.create_server(...)
```

→ 明示的にイベントループを渡す必要がある設計（古典的コードや低レベルAPI）では有効

---

## ✅ asyncio.create\_task() との関係

```python
loop = asyncio.get_running_loop()
loop.create_task(my_coro())
```

と

```python
asyncio.create_task(my_coro())
```

は本質的に等価。ただしライブラリ内で「どのループに紐づけるか」を意識したいときは `loop.create_task()` が明示的であるため好まれる。

---

## ✅ よくある誤解

| 誤解内容                            | 実際の挙動と注意点                                       |
| ------------------------------- | ----------------------------------------------- |
| 任意の場所で呼び出していい                   | ❌ 非同期関数内・イベントループ実行中の限定条件がある                     |
| `get_event_loop()` の上位互換である     | ✅ 実質的には正しいが、**用途が異なる**ため代替可能とは限らない              |
| `new_event_loop()` を使えば使えるようになる | ❌ ループを「開始していない」と `get_running_loop()` は依然エラーになる |
| REPL で使えば動く                     | ❌ REPL（特に Jupyter）ではループ状態が特殊であり、誤作動の原因になる場合がある  |

---

## ✅ 結論

`asyncio.get_running_loop()` は、**「この瞬間に動いているループを取得する」ための唯一の信頼できる方法**です。

* ✅ `async def` 内での Task 登録・Executor 呼び出しに必須
* ✅ `get_event_loop()` のように曖昧な挙動をせず、**明示的かつ安全**
* ✅ Python 3.7 以降の非同期設計では**基本中の基本**

---

## ✅ 実務上の原則

* **原則として asyncio プログラムでは `get_running_loop()` を用いよ**
* `get_event_loop()` は、**REPLやレガシー互換目的以外では封印すべき**
* `loop = asyncio.get_running_loop()` を使う場面では、**ループに紐づいた処理が必要な証拠**

---

