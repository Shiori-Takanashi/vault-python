## 🧷 pip + pyproject.toml の関係

---
### ✔ `pip` でも `pyproject.toml` を**必ず読む**

* **PEP 517/518 により、ビルドのためのメタデータとして `pyproject.toml` が標準化された**
* `pip install .` を行うと、`pyproject.toml` を読み、`[build-system]` に従ってパッケージをビルドする

---

## 📘 最低限必要な構成

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

これがあると、`setup.py` ベースのプロジェクトでも `pip` は `pyproject.toml` を使ってビルドする。
この定義が**無いと将来的に警告または失敗**する。

---

## 🧱 setuptools を使う場合（pip互換）

```toml
[build-system]
requires = ["setuptools>=61", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "poke-pipeline"
version = "0.1.0"
description = "ポケモン処理パイプライン"
authors = [{ name = "君", email = "you@example.com" }]
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "aiohttp>=3.9,<4.0",
    "pydantic>=2.7,<3.0",
]
```

これは `setup.py`, `requirements.txt` を不要にする構成。
**Python 3.12 + setuptools 61.0 以降で正式サポートされた構文（PEP 621）**

---

## ✅ pip install の使い方

### ローカルインストール（`pyproject.toml` を元に）

```bash
pip install .
```

→ `pyproject.toml` に基づいて依存がインストールされ、パッケージがビルドされる。

### ソース配布用のwheel生成（オプション）

```bash
python -m build
```

> `build` モジュールが必要：

```bash
pip install build
```

---

## 📌 注意点

| 注意事項                                            | 内容                                    |
| ----------------------------------------------- | ------------------------------------- |
| `setup.py` は省略可能になった                            | `pyproject.toml` と `setuptools` だけでOK |
| `requirements.txt` は残しても良い                      | `pip install -r` 用に維持可能（テスト用、CI用）     |
| `pyproject.toml` に書いた依存しか `pip install .` で入らない | 開発用パッケージ（pytest等）は別途扱う                |

---

## 💬 よくある混乱

| 誤解                                            | 実際                               |
| --------------------------------------------- | -------------------------------- |
| pip は `pyproject.toml` を使わない？                 | → **使う**（ビルド時に必須）                |
| `pyproject.toml` を書いても `pip install` だけで使えない？ | → `pip install .` で反映される         |
| `poetry` が必要？                                 | → **不要**。pip + setuptools で十分使える |

---

## ✅ 結論

* `pip` + `pyproject.toml` は **モダンPython開発の標準**
* `setup.py` を廃止し、`pyproject.toml` + `setuptools` に移行するのが現実的
* `build-system` セクションは**最低限必須**

---