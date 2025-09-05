# Black ― Pythonコードフォーマッタ

---

## 1. Blackとは何か

**Black** は Python 用の**非可逆コードフォーマッタ**。PEP8準拠のコード整形を行い、
**「スタイル議論を排除し、効率と一貫性を最大化する」** という思想に基づいている。

* キャッチコピー: *The uncompromising Python code formatter*
* 設定自由度が極めて低く、整形ルールはほぼ固定
* 手動整形不要 → 保守性・レビュー効率向上

---

## 2. 特徴

| 特徴               | 内容                                |
| ---------------- | --------------------------------- |
| 非可逆（destructive） | 整形前のスタイルは失われ、整形後の形が唯一の正解          |
| 高速処理             | ファイル単位で並列整形                       |
| 安定した整形ルール        | PEP8 + 独自ルール（末尾カンマ・括弧の補完など）       |
| 自動化しやすい          | VSCode保存時整形・pre-commit・CIに容易に組込可能 |

---

## 3. 導入

```powershell
py -m pip install black
```

* `py` は Windows標準のPythonランチャー（`py` → `python` でも可）
* 仮想環境（`python -m venv .venv` → `.\.venv\Scripts\activate`）の使用を推奨

---

## 4. 基本的な使い方（Windows）

```powershell
black .                          # カレントディレクトリ以下を整形
black script.py                  # 単一ファイル
black --check --diff script.py   # 差分のみ確認（書き換えなし）
```

---

## 5. 主なオプション

| オプション                    | 内容                   |
| ------------------------ | -------------------- |
| `--check`                | 整形せず、整形が必要かどうかのみ確認   |
| `--diff`                 | 差分を表示（実際の書き換えは行わない）  |
| `--line-length 88`       | 行の最大長（既定は88、PEP8は79） |
| `--target-version py312` | 対応Python構文バージョン（複数可） |
| `--exclude`              | 除外ファイル・ディレクトリ（正規表現）  |

---

## 6. pyproject.toml 設定例

```toml
[tool.black]
line-length = 88
target-version = ["py312"]
exclude = '''
/(
    \.git
  | \.venv
  | build
  | dist
  | __pycache__
  | .*_pb2\.py
)/
'''
```

> ※ `.toml` ファイルはプロジェクトルートに設置

---

## 7. 整形ルール例（before → after）

| 内容     | 整形前                     | 整形後                              |
| ------ | ----------------------- | -------------------------------- |
| 関数定義   | `def f(a,b):return a+b` | `def f(a, b):\n    return a + b` |
| リスト記述  | `['a','b','c']`         | `["a", "b", "c"]`                |
| 複数文1行  | `x=1; y=2`              | `x = 1\ny = 2`                   |
| 引数複数行  | `f(1,2,\n3)`            | カンマ補完 + 整形される                    |
| クォート統一 | `'hello'`               | `"hello"`                        |

---

## 8. 保存時に自動整形（VS Code）

`.vscode/settings.json`（Windows対応）：

```json
{
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter"
  }
}
```

### 必須拡張機能：

* `ms-python.black-formatter`（VS Code Marketplace）

> `.venv` を使っている場合、自動で検出される。未検出時は `Python: Select Interpreter` を選択し、`.venv\Scripts\python.exe` を指定。

---

## 9. pre-commit hook で自動整形

`.pre-commit-config.yaml`：

```yaml
- repo: https://github.com/psf/black
  rev: 24.3.0
  hooks:
    - id: black
      language_version: python3.12
```

### セットアップ（Windows PowerShell）：

```powershell
py -m pip install pre-commit
pre-commit install
```

→ Git のコミット前に `black` が自動実行される。

---

## 10. CI組込例（GitHub Actions）

`.github/workflows/format.yml`：

```yaml
name: Check Formatting

on:
  push:
    paths: ["**/*.py"]
  pull_request:

jobs:
  black-check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.12"
      - name: Install Black
        run: pip install black
      - name: Run Black Check
        run: black --check .
```

---

## 11. よくある誤解

| 誤解               | 実際                             |
| ---------------- | ------------------------------ |
| Black は linter   | ✕ → 形式整形のみで、エラー検出はしない          |
| Black は可逆（元に戻せる） | ✕ → 完全に上書き。元のスタイルは失われる         |
| 柔軟な整形スタイルが選べる    | ✕ → 事実上 `line-length` しか指定できない |
| PEP8を完全に遵守している   | △ → 一部PEP8より厳しく、一部では独自仕様       |

---

## 12. ワークフローへの組込み位置

| フェーズ     | 推奨処理                    |
| -------- | ----------------------- |
| 実装中      | 保存時に VSCode で自動整形       |
| ステージング直前 | `pre-commit` で自動整形      |
| CIチェック   | `black --check` による強制確認 |
| 本番ビルド前   | フォーマット済み状態を前提として確認      |

---

## 13. 結論

* `black` は柔軟性を排除し、**コード統一と自動整形**に特化したツール
* Windows でも **VSCode + `.venv` + pre-commit** の組合せで簡単に運用可能
* `black` 単独ではスタイル整形しか行わないため、必要なら `ruff` などの静的解析と組み合わせて使う

---
