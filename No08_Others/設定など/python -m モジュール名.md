
## 🧭 `python -m` とは何か

`python -m` は、**「モジュールとして実行」するためのコマンドライン構文**。

```bash
python -m モジュール名
```

> `モジュール名` に対応する Python モジュールを探し、**その中の `__main__.py` を実行**する。

---

## 🧠 通常の `python file.py` との違い

| 項目        | `python file.py` | `python -m module` |
| --------- | ---------------- | ------------------ |
| 実行対象      | ファイルパス           | モジュール名（Pythonパス解決） |
| import 対応 | 相対インポートが壊れやすい    | **正しくインポートされる**    |
| 実行場所      | カレントディレクトリ依存     | `PYTHONPATH` 検出に従う |

---

## 🧪 使用例（実行系）

### ✅ 公式モジュールを実行

```bash
python -m http.server 8000
```

→ `http.server` モジュールをサーバーとして実行

```bash
python -m venv .venv
```

→ 仮想環境を作成（venvモジュールの `__main__.py` 実行）

---

### ✅ ユーザー定義モジュールの実行

構造：

```
mypkg/
├── __init__.py
├── __main__.py  ← ここが実行対象
└── core.py
```

実行：

```bash
python -m mypkg
```

→ `mypkg/__main__.py` が実行される

---

## 🔧 モジュール化の利点

* `python mypkg/__main__.py` では相対importが壊れるが、
* `python -m mypkg` なら `mypkg.core` 等のimportが機能する

---

## 🧱 よく使うモジュール

| コマンド                    | 説明                         |
| ----------------------- | -------------------------- |
| `python -m pip install` | pipをモジュール経由で使用（信頼性高）       |
| `python -m venv .venv`  | 仮想環境作成                     |
| `python -m http.server` | 簡易Webサーバ                   |
| `python -m unittest`    | テスト実行                      |
| `python -m build`       | pyproject.toml に従いパッケージビルド |

---

## ✅ 結論

* `python -m` は **「モジュールとして安全に実行する手段」**
* **importエラーを防ぎ、明確な構造を保てる**
* パッケージ開発者・CIツール・テスト実行者にとっては**事実上の標準**

---

