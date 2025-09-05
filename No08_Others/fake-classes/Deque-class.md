## Deque クラス

```python
class Deque:
    def __init__(self):
        # 内部リストで要素を管理（FIFOキューとして使用）
        self._items = []

    def append(self, x):
        # 要素を末尾に追加（enqueue）
        self._items.append(x)

    def popleft(self):
        # 先頭の要素を取り出して削除（dequeue）
        return self._items.pop(0)

    def __bool__(self):
        # 空でないかを真偽値で返す（if queue: のように使える）
        return bool(self._items)
```

---

