## poetryとは何か

`poetry` は、**Pythonプロジェクトの依存管理・ビルド・パッケージングを統合的に行うツール**である。以下の機能を一元的に提供する：

* パッケージの依存解決（`pip` と `pip-tools` の役割を統合）
* 仮想環境の自動管理（`venv` 相当）
* パッケージのビルド・公開（`setuptools`, `twine` の代替）
* `pyproject.toml` に基づいた設定（PEP 517/518 準拠）

---

## poetryの目的と特徴

| 項目          | 内容                                            |
| ----------- | --------------------------------------------- |
| **一元管理**    | `pip`, `virtualenv`, `setuptools` 等の機能を統合     |
| **明確な定義**   | `pyproject.toml` にすべての設定を記述（`setup.py` 等不要）   |
| **ロックファイル** | `poetry.lock` により、**再現可能な環境構築**が可能（npmのような挙動） |
| **軽量なCLI**  | `poetry add` や `poetry run` など直感的コマンドで操作できる   |

---

## pip / venv との違い

| 観点   | poetry                          | pip + venv                            |
| ---- | ------------------------------- | ------------------------------------- |
| 仮想環境 | 自動管理（`poetry shell`）            | 手動で作成（`python -m venv .venv`）         |
| 依存解決 | 厳格な解決アルゴリズム（最新と互換の両立）           | 基本は最新を導入、互換性の保証は弱い                    |
| ファイル | `pyproject.toml`, `poetry.lock` | `requirements.txt`, `setup.py` など複数分散 |
| 再現性  | 高い（ロックファイル強制）                   | 低い（`requirements.txt` による明示が必要）       |
| 実務利用 | 徐々に普及、特に**ライブラリ開発**で有用          | 依然として多数の現場で採用されている標準的手法               |

---

## poetryの基本コマンド

```bash
# 初期化
poetry init

# 依存追加（本番）
poetry add requests

# 依存追加（開発用）
poetry add --dev pytest

# 仮想環境でコマンド実行
poetry run python main.py

# 仮想環境へ入る（bash）
poetry shell

# lockファイルからインストール
poetry install
```

---

## 実務面での扱い

* **導入している企業**は一部に限られる（特にWebアプリ系よりもライブラリ開発系で多い）
* **チーム開発では周囲とバージョンを揃えることが重要**であり、`poetry` が使えない現場もある
* `.venv` の場所が分かりづらいというデメリットもある
* ただし、**再現性の高いビルドとパッケージ公開には圧倒的に有利**

---

## 注意点

* Pythonのパスと仮想環境のパスが衝突することがある（`poetry config virtualenvs.in-project true` 推奨）
* 初学者が使うと中で何が起きているのか理解しづらいことがある（透明性が低い）
* Dockerなど他ツールとの統合がpip/venvより複雑になる場合がある

---

## 補足：pyproject.toml の例

```toml
[tool.poetry]
name = "poke-pipeline"
version = "0.1.0"
description = "ポケモンデータのパイプライン構築"
authors = ["君 <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.12"
aiohttp = "^3.9.1"

[tool.poetry.dev-dependencies]
pytest = "^8.1.1"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

