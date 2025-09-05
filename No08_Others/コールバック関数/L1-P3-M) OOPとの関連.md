## 9章: メソッドをコールバックとして渡す

---

### 解説

コールバック関数として渡す対象は、通常の関数に限らず、クラスインスタンスのメソッドも利用できる。Pythonにおいては、`self.method` のように記述することで、そのインスタンスの状態を保持したままメソッドを関数オブジェクトとして他の関数に渡すことができる。

これにより、状態に依存した処理や、同じインターフェースで異なる挙動を実現する設計が可能になる。ただし、クラスメソッドを渡す際には、バインド済み（`self`付き）かどうかに注意する必要がある。

---

### メソッドをそのまま渡す

```python
class Greeter:
    def __init__(self, name):
        self.name = name

    def say_hello(self):
        print(f"{self.name}さん、こんにちは")

def run(callback):
    callback()

g = Greeter("田中")
run(g.say_hello)  # => 田中さん、こんにちは
```

ここでは `g.say_hello` が「インスタンスにバインドされたメソッド」として関数と同様に渡されている。

---

### 状態の保持とコールバックの関係

メソッドは内部的に `self` を通じてインスタンスの状態を参照しているため、コールバックとして渡しても「状態付き処理」が自然に実現できる。

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        print(f"カウント: {self.count}")

def trigger_n_times(callback, n):
    for _ in range(n):
        callback()

c = Counter()
trigger_n_times(c.increment, 3)
# カウント: 1
# カウント: 2
# カウント: 3
```

このように、バインドされたメソッドを渡すことで、状態の累積・管理を保ちながら繰り返し処理が可能になる。

---

### `functools.partial` との違いと併用可能性

メソッドは `self` にバインドされた状態で渡されるが、他の引数を事前に固定して渡したい場合には `functools.partial()` を併用する。

```python
from functools import partial

class Logger:
    def log(self, level, msg):
        print(f"[{level}] {msg}")

logger = Logger()
info_logger = partial(logger.log, "INFO")
info_logger("初期化完了")  # => [INFO] 初期化完了
```

このように、メソッドに特定の引数を固定しておくことで、柔軟にコールバック化することができる。

---

### 用語と構造の整理

| 用語                   | 内容                                  |
| -------------------- | ----------------------------------- |
| バインドメソッド             | `self.method` のように、インスタンスに結びついたメソッド |
| 関数オブジェクト             | 関数やメソッドを「引数として渡せる値」として扱う構文          |
| `partial(func, ...)` | 関数に一部の引数を事前に固定した新しい関数オブジェクトを生成する    |

---

### 利用例と実用パターン

| 利用場面           | 内容                                            |
| -------------- | --------------------------------------------- |
| GUIイベントの状態管理   | インスタンスに紐づいた状態（例: 入力内容）をボタン処理で活用               |
| 非同期コールバック登録    | 状態保持オブジェクトのメソッドを `add_done_callback()` に渡す    |
| 高階APIへの登録      | `sorted(key=...)` や `map(...)` にインスタンスメソッドを渡す |
| 関数的スタイルとOOPの融合 | OOPで設計された処理を関数スタイルで柔軟に組み立て可能にする               |

---
