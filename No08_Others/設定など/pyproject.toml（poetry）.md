以下に、**Obsidian用ノート形式**で `pyproject.toml` を詳細に解説します。

---

## 📘 pyproject.toml とは

`pyproject.toml` は、Pythonプロジェクトの**ビルドや依存管理の標準構成ファイル**。
[PEP 518](https://peps.python.org/pep-0518/)、[PEP 621](https://peps.python.org/pep-0621/) などで標準化されている。

---

## 🎯 目的

* Pythonパッケージの **ビルド設定** を統一的に定義する
* `setuptools`, `poetry`, `flit`, `black`, `isort`, `mypy` などの**ツールの設定を一元化**
* `setup.py`, `setup.cfg`, `requirements.txt` の**混在を解消する方向性**

---

## 🔧 主な構造（例：poetryを使用）

```toml
[tool.poetry]
name = "poke-pipeline"
version = "0.1.0"
description = "ポケモンデータ処理パイプライン"
authors = ["君 <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.12"
aiohttp = "^3.9"
pydantic = "^2.7"

[tool.poetry.dev-dependencies]
pytest = "^8.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

## 🧱 セクションごとの意味

### \[build-system]

* ビルドに必要なツールを定義（必須）
* 例（`poetry` 使用時）：

```toml
[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

* 例（`setuptools` 使用時）：

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

---

### \[tool.poetry]（または \[tool.setuptools]）

* `poetry` が使うメタ情報や依存を定義
* `name`, `version`, `description`, `authors` は PEP 621 に準拠

---

### \[tool.poetry.dependencies]

* 本番環境で必要な依存パッケージ
* バージョン指定は `^`, `>=`, `<`, `~=`, `==` など使える

---

### \[tool.poetry.dev-dependencies]

* 開発時のみに必要な依存（pytest, black など）

---

### \[tool.black], \[tool.mypy] など

* コードフォーマッタ・型チェッカなどの**設定もここに統合可能**

```toml
[tool.black]
line-length = 88
target-version = ["py312"]

[tool.mypy]
strict = true
```

---

## 📦 poetry以外の例：setuptools利用時

```toml
[project]
name = "poke-pipeline"
version = "0.1.0"
description = "ポケモン処理"
authors = [{ name = "君", email = "you@example.com" }]
dependencies = [
  "aiohttp>=3.9,<4.0",
  "pydantic>=2.7,<3.0",
]

[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

---

## 🔁 poetry との関係

* `pyproject.toml` は **poetryの唯一の設定ファイル**
* `requirements.txt` は不要（`poetry.lock` によって代替）
* 依存追加は以下のように自動反映：

```bash
poetry add requests
```

→ `pyproject.toml` の `[tool.poetry.dependencies]` に反映される。

---

## 🧪 チェックリスト

| 確認項目                                   | 内容                   |
| -------------------------------------- | -------------------- |
| `build-system` は定義済みか？                 | PEP 518に必須           |
| `tool.poetry` または `project` セクションはあるか？ | ビルドや配布のために必要         |
| 開発用設定（mypy, blackなど）は統合されているか？         | `.toml` にまとめると保守性が高い |

---

## ✅ 結論

* `pyproject.toml` は **Pythonプロジェクトの統一的な設定の中心**
* `poetry` を使う場合は必須
* 将来的には **すべてのPythonパッケージが `pyproject.toml` ベースになる可能性が高い**

---

必要であれば `setup.cfg` との比較や、既存プロジェクトへの導入手順なども追記可能です。
