> # `random` モジュールの基本

---

### インポート

```python
import random
```

---

> ## 主な関数

### 1. `random.random()`

* 0.0以上1.0未満の**浮動小数点数**を返す。

```python
random.random()  # 例: 0.3748...
```

---

### 2. `random.randint(a, b)`

* `a` 以上 `b` 以下の**整数**を返す（両端を含む）。

```python
random.randint(1, 10)  # 例: 7
```

---

### 3. `random.uniform(a, b)`

* `a` 以上 `b` 以下の**浮動小数点数**を返す。

```python
random.uniform(1.0, 5.0)  # 例: 3.1415...
```

---

### 4. `random.choice(seq)`

* シーケンス（リスト、文字列、タプルなど）から**ランダムに1つ選ぶ**。

```python
random.choice(['a', 'b', 'c'])  # 例: 'b'
```

---

### 5. `random.choices(seq, k=n)`

* **重複ありでn個選ぶ**（`k=3` などで指定）。

```python
random.choices([1, 2, 3, 4], k=2)  # 例: [3, 1]
```

---

### 6. `random.sample(seq, k=n)`

* **重複なしでn個選ぶ**（`seq` の長さ以下）。

```python
random.sample([1, 2, 3, 4], k=2)  # 例: [4, 1]
```

---

### 7. `random.shuffle(seq)`

* リストを\*\*その場でシャッフル（破壊的変更）\*\*する。

```python
lst = [1, 2, 3, 4]
random.shuffle(lst)
# lst はランダムな順序に変更される
```

---

### 8. シード固定（再現性の確保）

```python
random.seed(42)
```

---

用途が決まっている場合（例：ガチャの当選確率、分布に従った乱数、スリープ時間のばらつきなど）、追加で説明可能です。要望があれば明示してください。
