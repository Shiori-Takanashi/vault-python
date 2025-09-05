`time` モジュールは、Pythonで最も基本的な時間管理ユーティリティです。典型的な使用例をいくつか挙げます。

---

### ✅ 1. 処理時間の計測（`time.time()`）

```python
import time

start = time.time()
# 処理（例: ループなど）
for _ in range(1000000):
    pass
end = time.time()

print(f"処理時間: {end - start:.5f} 秒")
```

---

### ✅ 2. 一定時間待つ（`time.sleep()`）

```python
import time

print("3秒待機中…")
time.sleep(3)
print("完了")
```

---

### ✅ 3. 現在時刻の取得（UNIXエポック秒）

```python
import time

now = time.time()
print(f"現在のUNIX時刻: {now}")
```

---

### ✅ 4. 現在時刻のローカル表記（`time.localtime()` / `time.strftime()`）

```python
import time

now = time.localtime()
formatted = time.strftime("%Y-%m-%d %H:%M:%S", now)
print(f"現在時刻: {formatted}")
```

---

### ✅ 5. タイムスタンプから文字列変換

```python
import time

timestamp = 1700000000
dt = time.localtime(timestamp)
print(time.strftime("%Y/%m/%d %H:%M", dt))  # → '2023/11/14 09:13' など
```

---

時間管理が単純で済むケースでは `datetime` よりも `time` が軽くて扱いやすいです。実用では、**処理時間の計測**と**スリープ**が最も頻繁に使われます。
