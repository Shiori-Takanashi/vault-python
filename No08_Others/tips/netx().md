> ## `next()` 関数について

---

> ### 基本構文

```python
next(イテレーター [, デフォルト値])
```

* **イテレーターから次の要素を1つ取り出す**
* 要素がなくなった場合：

  * デフォルト値があればそれを返す
  * なければ `StopIteration` 例外を送出

---

> ### 使用例：基本

```python
it = iter([10, 20, 30])

next(it)  # 10
next(it)  # 20
next(it)  # 30
next(it)  # StopIteration 発生
```

---

> ### 使用例：デフォルト値あり

```python
it = iter([1, 2])

next(it)       # 1
next(it)       # 2
next(it, -1)   # -1（StopIterationを回避）
```

---

> ### 注意点

* 対象は **イテレーター** に限る
  → イテラブル（例：list）には直接使えない

```python
next([1, 2, 3])  # TypeError
next(iter([1, 2, 3]))  # OK
```

---

> ### for文との関係

* `for` 文は内部で `iter()` + `next()` を使っている
* `next()` は**手動で繰り返し処理を行いたいとき**に便利

---

> ### StopIterationの捕捉

```python
it = iter("ab")

try:
    while True:
        x = next(it)
        print(x)
except StopIteration:
    print("終端")
```

---

> ### 応用：ジェネレーターとの併用

```python
def gen():
    yield "a"
    yield "b"

g = gen()
next(g)  # "a"
next(g)  # "b"
next(g)  # StopIteration
```

---
