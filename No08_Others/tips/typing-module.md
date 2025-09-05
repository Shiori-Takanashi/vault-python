# ❓質問
> `typing`モジュールとは？

# ❗回答
> Pythonにおける\*\*型ヒント（type hint）\*\*を記述するための標準ライブラリです。静的型チェックや補完支援などを強化する目的で使われます。Python 3.5で導入され、後にPEP 484やPEP 585などで段階的に機能が拡張されています。

---

> ## 基本的な役割

* **関数引数や戻り値に型を明示する**
* **構造化された型（List, Dict, Tupleなど）を表現する**
* **ジェネリック型やUnionなど複雑な型を表現する**

---

> ## 主な型構文（旧式と新式）

| 型              | 旧式 (`typing.`)       | 新式（Python 3.9+）   |        |
| -------------- | -------------------- | ----------------- | ------ |
| リスト            | `List[int]`          | `list[int]`       |        |
| 辞書             | `Dict[str, int]`     | `dict[str, int]`  |        |
| タプル            | `Tuple[int, str]`    | `tuple[int, str]` |        |
| オプショナル（None許容） | `Optional[int]`      | \`int             | None\` |
| 複数型（Union）     | `Union[int, str]`    | \`int             | str\`  |
| ジェネリクス         | `TypeVar`, `Generic` | 同様に使用             |        |

---

> ## 使用例

```python
from typing import List, Dict, Optional

def greet(names: List[str], age: Optional[int]) -> Dict[str, str]:
    return {name: f"Hello, age {age}" for name in names}
```

---

> ## 補足

* Python 3.11から `typing` の多くの機能が `builtins` に移行され、記法が簡略化されている（例：`list[int]`, `str | None`）。
* `mypy` や `pyright` などの静的型チェッカーと併用することで、開発時の型安全性が向上する。
* 実行時には型情報は無視される（**注釈のみ**）。

---

必要であれば、`TypeVar`, `Callable`, `Protocol`, `Literal`, `Annotated` などの上級機能についても個別に解説可能です。
