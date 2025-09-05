## .toml とは何か

`.toml` は **設定ファイル用の軽量マークアップ形式**で、
**"TOML = Tom's Obvious, Minimal Language"** の略称。

> 主に `pyproject.toml` や `poetry.toml` などで使用される。

---

## 目的と特徴

| 項目    | 内容                                                                 |
| ----- | ------------------------------------------------------------------ |
| 用途    | 設定ファイル（構成定義）                                                       |
| 主な利用先 | `pyproject.toml`, `Cargo.toml`（Rust）, `poetry`, `Black`, `mypy` など |
| 仕様    | 正式仕様: [toml.io](https://toml.io)                                   |
| 特徴    | シンプル、明快、インデント自由（YAMLより堅牢）                                          |

---

## 文法の基本

### ✔︎ キーと値（key = value）

```toml
name = "poke-pipeline"
version = "0.1.0"
debug = true
count = 42
```

### ✔︎ 文字列・数値・真偽値

```toml
title = "サンプル"
pi = 3.1415
active = false
```

### ✔︎ 配列（リスト）

```toml
keywords = ["python", "pokeapi", "pipeline"]
```

### ✔︎ テーブル（セクション）

```toml
[database]
host = "localhost"
port = 5432
```

これは以下と等価：

```json
{
  "database": {
    "host": "localhost",
    "port": 5432
  }
}
```

---

## pyproject.toml での例（Python）

```toml
[tool.poetry]
name = "poke-pipeline"
version = "0.1.0"
description = "ポケモンデータのパイプライン"
authors = ["君 <example@example.com>"]

[tool.poetry.dependencies]
python = "^3.12"
aiohttp = "^3.9"

[tool.poetry.dev-dependencies]
pytest = "^8.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

## JSON / YAML / TOML の比較

| 比較軸         | JSON  | YAML        | TOML                  |
| ----------- | ----- | ----------- | --------------------- |
| 構文エラーの出やすさ  | 少ない   | 多い（インデント依存） | 少ない                   |
| 読みやすさ       | 並     | 高           | 高                     |
| コメント        | ❌     | ✅           | ✅                     |
| 設定用途での使われ方  | 少     | 多           | 増加中（Python公式準拠）       |
| Pythonとの親和性 | 高（標準） | 中           | 高（`tomli`, `tomllib`） |

---

## Pythonでの読み込み

Python 3.11 以降：

```python
import tomllib

with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)
```

Python 3.10 以前では `tomli` をインストールして代用：

```bash
pip install tomli
```

---

## 結論

* `.toml` は構成情報を記述するのに適したフォーマット。
* Pythonでは **`pyproject.toml` による標準化が進行中**。
* `.toml` を編集できることは **モダンなPython環境の前提スキル**になりつつある。

---
