### ✅ `target` とは？（スレッドにおける）

> Python のスレッド処理（`threading.Thread`）において、
> \*\*`target` は「そのスレッドで実行させる関数」\*\*を指定する引数です。

---

## ✅ 一文で言えば

> `target=関数名` は「**このスレッドはこの関数を走らせる**」という意味。

---

## ✅ 具体例

```python
import threading

def work():
    print("スレッド内の処理")

t = threading.Thread(target=work)
t.start()  # → 別スレッドで work() が実行される
t.join()   # → メインスレッドはこの終了を待つ
```

---

## ✅ target に渡すのは「関数名のみ」

```python
# ✅ 正しい（関数オブジェクト）
threading.Thread(target=func)

# ❌ 間違い（関数を呼び出してしまっている）
threading.Thread(target=func())  # ← 実行して返り値を渡している
```

---

## ✅ 引数を渡したい場合：`args` を併用

```python
def greet(name):
    print(f"こんにちは、{name}さん")

t = threading.Thread(target=greet, args=("君",))
t.start()
```

---

## ✅ target が実行されるタイミング

* `t = Thread(target=...)` では**まだ何も実行されない**
* `t.start()` を呼ぶと → **その関数が別スレッドで非同期に走り出す**

---

## ✅ target は必須か？

| 状況                | 結果                              |
| ----------------- | ------------------------------- |
| `target=None`（省略） | 何も実行されない空スレッド（意味なし）             |
| サブクラス化した場合        | `run()` メソッドを自分で書けば `target` 不要 |

---

## ✔ まとめ

> **スレッドにおける `target` は、そのスレッドで動かす関数の指定。**
> 必ず関数オブジェクト（`func`）として渡す。
> 実行タイミングは `start()` を呼んだ瞬間。

---

