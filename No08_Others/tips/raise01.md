> ## raiseの基本

---

> ### 基本構文

```python
raise エラークラス("メッセージ")
```

例：

```python
raise ValueError("不正な値です")
```

---

> ### raiseの動作

* `raise` は例外を送出（throw）して処理を中断する。
* 捕捉しなければ、**プログラムは直ちに停止**する。
* `try` / `except` 節で捕捉すれば、処理を継続できる。

---

> ### 捕捉しない例

```python
def divide(x, y):
    if y == 0:
        raise ZeroDivisionError("ゼロ除算")
    return x / y

print(divide(10, 0))  # エラー送出 → プログラム停止
```

---

> ### 捕捉する例

```python
def divide(x, y):
    if y == 0:
        raise ZeroDivisionError("ゼロ除算")
    return x / y

try:
    print(divide(10, 0))
except ZeroDivisionError as e:
    print(f"エラー発生: {e}")

print("続行可能")  # エラーを捕捉したため実行される
```

---

> ### ユーザー定義エラー

```python
class MyCustomError(Exception):
    pass

raise MyCustomError("独自エラーです")
```

---

> ### 注意点

* 例外が送出された時点で、**その関数以降のコードは実行されない**。
* `raise` は**意図的な中断ポイント**として使われる。

---
