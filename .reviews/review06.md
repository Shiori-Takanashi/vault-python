# ディレクトリのNo順序に関する判断

## 対象ディレクトリ
1. **Standard_Library**
   - サブディレクトリ: `asyncio`, `Logging`, `os`, `Pathlib`, `re`, `Shutil`, `Sys`, `Theareding`, `Typing`, `Unnittest`
   - **内容**: Python標準ライブラリに関する学習資料。

2. **Third_Party_Library**
   - サブディレクトリ: `Beautifulsoup`, `Playwright`, `Pytest`, `click`
   - **内容**: サードパーティライブラリに関する学習資料。

3. **Framework**
   - サブディレクトリ: `Django`
   - **内容**: Djangoフレームワークに特化した学習資料。

4. **Development**
   - サブディレクトリ: `.mypy_cache`（キャッシュ）、`BKP_GitHub_Actions.md`, `GitHub_Actions.md`, `format.py`
   - **内容**: 開発関連の設定やスクリプト。

---

## 順序の提案
### 基準
- **一般性**: より多くのプロジェクトで利用される内容を優先。
- **依存関係**: 標準ライブラリ → サードパーティ → フレームワーク → 開発ツールの順。

### 提案する順序
1. **No01_Standard_Library**
   - 理由: Pythonの基礎であり、他のライブラリやフレームワークの基盤となる。

2. **No02_Third_Party_Library**
   - 理由: 標準ライブラリを補完する形で利用されることが多い。

3. **No03_Framework**
   - 理由: 特定の用途に特化した内容であり、サードパーティライブラリの後に位置付けるのが自然。

4. **No04_Development**
   - 理由: 開発環境や設定に関する内容は、他の学習資料の後に参照されることが多い。

---

## 結論
- 上記の順序でディレクトリをリネームすることを推奨します。
