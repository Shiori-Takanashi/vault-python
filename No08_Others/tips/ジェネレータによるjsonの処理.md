> ## ジェネレーターを用いて大量のJSONファイルを逐次処理する

---

> ### 背景と目的

* 一度にすべてのファイルをメモリに載せると **RAM不足や処理遅延**の原因になる
* **ジェネレーターで1件ずつ読み込み・処理・解放**することで安定した動作と高速化を図る

---

> ### 前提条件

* ディレクトリ内に `data_0001.json` ～ `data_1000.json` が存在すると仮定
* ファイル構造は以下のような単純なJSON：

```json
{
  "id": 1,
  "name": "sample"
}
```

---

> ### ジェネレーター定義

```python
import json
from pathlib import Path
from typing import Generator, Dict

def json_file_generator(dir_path: Path) -> Generator[Dict, None, None]:
    for file in sorted(dir_path.glob("data_*.json")):
        with file.open(encoding="utf-8") as f:
            try:
                yield json.load(f)
            except json.JSONDecodeError:
                continue  # 無効なJSONをスキップ
```

---

> ### 利用例：逐次的にデータを処理する

```python
DATA_DIR = Path("./json_data")  # ディレクトリ指定

for json_obj in json_file_generator(DATA_DIR):
    print(json_obj["id"], json_obj["name"])
```

📌 メモリには1件ずつしか載らないため、大規模処理にも対応可能。

---

> ### 応用：条件に一致したものだけ抽出

```python
results = (
    obj for obj in json_file_generator(DATA_DIR)
    if obj.get("name") == "target_name"
)

for match in results:
    print(match)
```

---

> ### 特徴と利点

| 項目     | 内容                         |
| ------ | -------------------------- |
| メモリ使用量 | **低い**（一度に1ファイルだけ）         |
| 拡張性    | サブディレクトリ走査、拡張子フィルタなどが容易    |
| 応答性    | 最初のファイルをすぐに処理できる           |
| 保守性    | `glob()` によりファイル順を明示的に制御可能 |

---
