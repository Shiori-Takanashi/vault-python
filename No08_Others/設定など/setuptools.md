
## 📦 setuptoolsとは何か

`setuptools` は、**Pythonパッケージの構築・インストール・配布を行う標準的なツール**。

> Pythonの公式なパッケージング方式であり、長年にわたって `pip` から使われている。

---

## 🧱 主な役割

| 用途      | 説明                              |
| ------- | ------------------------------- |
| ビルド     | パッケージを `.tar.gz` や `.whl` 形式に変換 |
| インストール  | `pip install .` でソースからパッケージを導入  |
| 配布      | `PyPI` などへのアップロードに対応            |
| メタデータ定義 | 名前・バージョン・依存パッケージなど              |

---

## 🧭 歴史的位置づけ

| 世代  | ツール              | 備考                              |
| --- | ---------------- | ------------------------------- |
| 古い  | `distutils`      | 廃止済み（Python 3.12で完全削除）          |
| 中央  | ✅ `setuptools`   | 依然として業界標準                       |
| 新しい | `poetry`, `flit` | より簡潔で高速なツール。`pyproject.toml` 前提 |

---

## 📝 基本構成（古い形式）

```python
# setup.py
from setuptools import setup

setup(
    name="poke-pipeline",
    version="0.1.0",
    packages=["src"],
    install_requires=[
        "aiohttp>=3.9,<4.0",
        "pydantic>=2.7,<3.0",
    ],
)
```

```bash
pip install .
```

---

## 🔧 モダン構成（pyproject.tomlのみ）

```toml
# pyproject.toml
[build-system]
requires = ["setuptools>=61", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "poke-pipeline"
version = "0.1.0"
dependencies = [
    "aiohttp>=3.9,<4.0",
    "pydantic>=2.7,<3.0"
]
```

この形式なら `setup.py` は不要。

---

## 🔨 よく使うコマンド

```bash
# パッケージをビルド
python -m build

# インストール（ローカルパッケージ）
pip install .

# 開発モード（ソース編集を即反映）
pip install -e .
```

---

## 🔁 setuptoolsとpipの関係

| 項目              | 内容                               |
| --------------- | -------------------------------- |
| pip             | パッケージのインストーラ（`pip install`）      |
| setuptools      | パッケージを**構築するための仕組み**             |
| `pip install .` | → `setuptools` を使ってビルドしてからインストール |

---

## ⚠️ 注意点

| 問題                  | 解説                                  |
| ------------------- | ----------------------------------- |
| `setup.py` の手書きは冗長  | `pyproject.toml` のみで記述する構成へ移行が進んでいる |
| `distutils` は非推奨    | Python 3.12で完全削除された                 |
| `setuptools` は低レイヤー | poetryやflitのような依存管理は提供しない           |

---

## ✅ 結論

* `setuptools` は **最も基本的なパッケージビルドツール**
* `pip` は `setuptools` を内部的に呼び出して使っている
* 今後は **`pyproject.toml` + `setuptools`** による記述が主流になる

---

