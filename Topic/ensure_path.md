
## 📍01章：パスが正規であるか（正規化の確認）

---

### 📝解説

**パスが正規であるか**の確認とは、相対パスや記号（`..` や `.`）を含んだパスを、**絶対パスに変換し、冗長性のない形**にする処理である。
Python では `pathlib.Path` を使い、`resolve()` メソッドを用いることで、シンボリックリンクの解決やカレントディレクトリとの統合を行った、**正規化された絶対パス**を取得できる。

注意点として、対象ファイルやディレクトリが**実在しない場合にエラーを出すかどうか**は、`strict` 引数の設定に依存する（`strict=False` であれば存在確認せずに正規化する）。

---

### ✅ 使用例（パスの正規化）

```python
from pathlib import Path

p = Path("./.././folder/../file.txt")
print(p.resolve(strict=False))  # → /home/user/file.txt（存在しなくてもOK）
```

---

### ⚠️ 注意点

* `resolve(strict=True)` は対象が存在しないと `FileNotFoundError` を発生させる。
* `resolve()` はシンボリックリンクをたどるため、実ファイルの場所が変わる可能性がある。
* 正規化が目的であれば `strict=False` を推奨。

---

## 📍02章：パスに対象が存在するか（存在確認）

---

### 📝解説

**指定されたパスにファイルやディレクトリが存在するか**を確認するには、`pathlib.Path.exists()` を用いる。
これは、シンボリックリンクを含めたパスが**実在するかどうか**を判定し、結果を真偽値で返す。

存在する対象がファイルなのか、ディレクトリなのかを区別したい場合は、`is_file()` や `is_dir()` を使う。

---

### ✅ 使用例（存在確認）

```python
from pathlib import Path

p = Path("example.txt")

if p.exists():
    print("存在します")
else:
    print("存在しません")
```

---

### 🔍 より詳細な確認

```python
if p.is_file():
    print("これはファイルです")
elif p.is_dir():
    print("これはディレクトリです")
else:
    print("どちらでもありません（存在しない可能性）")
```

---

### ⚠️ 注意点

* `exists()` はシンボリックリンクの実体が存在しない場合、`False` を返す。
* 隠しファイルやアクセス権の制限には注意が必要。
* 事前の `resolve()` は存在確認には不要であり、むしろ例外を出す可能性がある。

---

### 🧩 補足：正規化と存在確認の順序

正規化 → 存在確認という順に行う場合は、**`resolve(strict=False)` → `exists()`** のように使うとよい。
ただし `resolve()` はディスクI/Oが発生するため、**処理性能が重要な場面では慎重に使うべき**である。用途に応じた選択が望ましい。

以下に、`Path.resolve()` における **エラーキャッチの方法**を含めた実用的な解説を、いつものフォーマットで追補します。特に `strict=True` の場合に注意が必要です。

---

## 📍03章：resolve() による例外の発生と対処

---

### 📝解説

`pathlib.Path.resolve(strict=True)` は、**対象のファイルやディレクトリが実在しない場合に `FileNotFoundError` を発生させる**。
これにより、パスの正規化と同時に、実在確認を**例外ベースで強制的に行いたい場面**で有効である。

一方、対象の存在に関係なくパスのみ正規化したい場合は `strict=False`（デフォルト）を使用する。

---

### ✅ 基本構文（try–except）

```python
from pathlib import Path

p = Path("not_exist.txt")

try:
    real_path = p.resolve(strict=True)
    print(f"正規パス: {real_path}")
except FileNotFoundError:
    print("対象が存在しません")
```

---

### 🔍 エラーの具体例

```python
# 存在しないパスに strict=True を指定すると:
Path("unknown.txt").resolve(strict=True)
# → FileNotFoundError: [Errno 2] No such file or directory: 'unknown.txt'
```

---

### 🧪 補足：使用意図による選択

| 用途               | 推奨設定                 |
| ---------------- | -------------------- |
| 存在チェックを兼ねた絶対パス取得 | `strict=True`（例外で処理） |
| 存在不要の正規化（構文目的のみ） | `strict=False`（例外回避） |

---

### ⚠️ 注意点

* `resolve(strict=True)` はシンボリックリンクの不整合でも `FileNotFoundError` を投げる。
* 対象が**ネットワークドライブや外部ボリューム**であれば、IOエラーが出る可能性もあるため、`OSError` 全体を補足する方が安全な場合もある。

```python
try:
    path.resolve(strict=True)
except (FileNotFoundError, OSError) as e:
    logging.warning(f"解決失敗: {e}")
```

---

### ✅ 統合：パスが正規かつ存在するかを安全に確認する関数

```python
def get_safe_path(p: Path) -> Path | None:
    try:
        return p.resolve(strict=True)
    except FileNotFoundError:
        logging.warning(f"{p} は存在しません")
        return None
```

---
