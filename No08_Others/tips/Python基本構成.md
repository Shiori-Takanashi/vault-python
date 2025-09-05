> ## ✅ Python基本構成
---

### ◾ 言語・環境

* **Python 3.10 以上**

  * 構文の簡素化（例：パターンマッチング）
  * 型アノテーションの強化（`X | Y`, `Self` など）

* **venv / poetry / pipenv のいずれかで仮想環境を作成**
  → 推奨：`python -m venv .venv && source .venv/bin/activate`

---

### ◾ パッケージ管理（最低限）

```bash
pip install -U pip setuptools wheel
```

オプション：

```bash
pip install -U poetry  # pyproject.tomlベースの依存管理
```

---

### ◾ Lint／静的解析：`flake8`, `pylint`, `pyright`

```bash
pip install flake8 pylint
npm install -D pyright  # ※型検査はJSツールが強力
```

* **`flake8`**：スタイルと軽度な構文ミスの検出（軽量）
* **`pylint`**：設計レベルの警告、命名規則、unused検出など（厳格）
* **`pyright`**：型アノテーションの整合性チェック（VSCode / CLI）

```ini
# .flake8
[flake8]
max-line-length = 100
exclude = .venv
```

```ini
# pyrightconfig.json（TSに似た記法）
{
  "include": ["src"],
  "strict": true
}
```

---

### ◾ 自動整形：`black`, `isort`

```bash
pip install black isort
```

* **`black`**：文法準拠な整形（意図より正しさ優先）
* **`isort`**：import文の順序整列（blackと併用可）

```ini
# pyproject.toml
[tool.black]
line-length = 100

[tool.isort]
profile = "black"
```

---

### ◾ Gitフック（自動実行）

```bash
pip install pre-commit
pre-commit install
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks: [{ id: black }]
  - repo: https://github.com/PyCQA/isort
    rev: 5.12.0
    hooks: [{ id: isort }]
  - repo: https://github.com/PyCQA/flake8
    rev: 6.1.0
    hooks: [{ id: flake8 }]
```

---

### ◾ EditorConfig（VSCode / PyCharm整合）

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

---

### ◾ 最小構成まとめ（Py標準6点）

1. **flake8**（スタイル／構文Lint）
2. **pylint**（設計／冗長性チェック）
3. **black**（自動整形）
4. **isort**（import順整理）
5. **pyright**（型チェック）
6. **pre-commit**（Git hook連携）

---

### ◾ 推奨構成：pyproject.tomlベース（管理一元化）

```toml
[tool.black]
line-length = 100

[tool.isort]
profile = "black"

[tool.pylint.messages_control]
disable = "C0114,C0115,C0116"  # docstring警告を無効化
```

---
