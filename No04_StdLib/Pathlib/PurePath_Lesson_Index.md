---
title: 
keyword: 
created: 2025-07-30 04:55
modified: 2025-07-30 04:55
vault: Python
catergory: Language
language: Python
area: Standard_Library
identify:  
type: Lesson...
tole: Index
order: 00
---

以下に、`PurePath_Obj_Use_Index` の第01部「パス情報の取得（プロパティ）」に該当する全10章の目次を展開します。`Path` と共通ですが、**PurePath はファイルシステム非依存**である点を前提に構成しています。

---

## 📍第01部：パス情報の取得（プロパティ）

---

### 📝第01章\_`.name`

* パスの末尾要素（ファイル名またはディレクトリ名）を取得
* 例：`PurePath("a/b/file.txt").name == "file.txt"`
* ファイル名の取得やログ出力に使用される

---

### 📝第02章\_`.stem`

* `.name` から最後の拡張子を除いたベース名を返す
* 拡張子が複数ある場合も末尾のみ除去
* 例：`PurePath("file.tar.gz").stem == "file.tar"`

---

### 📝第03章\_`.suffix`

* `.name` の末尾にある拡張子（ピリオド付き）を返す
* 拡張子による分類や条件分岐に活用
* 例：`PurePath("file.tar.gz").suffix == ".gz"`

---

### 📝第04章\_`.suffixes`

* `.name` のすべての拡張子をリスト形式で返す
* `.tar.gz` → `[".tar", ".gz"]` のように多段拡張子を解析
* 拡張子の判別や安全なファイル変換処理に役立つ

---

### 📝第05章\_`.parts`

* パス全体をルートから順に分割したタプルで返す
* 例：`PurePath("/a/b/file.txt").parts == ("/", "a", "b", "file.txt")`
* 各階層の要素アクセスや比較処理に有効

---

### 📝第06章\_`.parent`

* 1階層上の親パスを返す
* 例：`PurePath("a/b/c").parent == PurePath("a/b")`
* 階層操作や相対参照の処理に使用される

---

### 📝第07章\_`.parents`

* すべての親階層を `PurePath` オブジェクトで列挙する
* リスト形式ではなくインデックスアクセス可能な専用オブジェクト
* 例：`PurePath("a/b/c").parents[0] == PurePath("a/b")`

---

### 📝第08章\_`.anchor`

* `.drive` + `.root` を連結した文字列を返す
* Windows: `"C:\"`、POSIX: `"/"`
* ルート基準でのパス判定に活用できる

---

### 📝第09章\_`.drive`

* Windows専用：パスのドライブ名（例："C:"）を返す
* POSIXでは空文字 `""` を返す
* OS間の差異を処理する前提情報として有用

---

### 📝第10章\_`.root`

* パスのルート（例：`"/"`）を返す
* 絶対パスと相対パスの判定や正規化処理に活用
* Windows: `"\"`、POSIX: `"/"`、相対パス: `""`

---

必要であれば、第02部以降の展開も可能です。指定してください。
以下に、`PurePath_Obj_Use_Index` の第02部「パス判定・比較系（メソッド）」の全目次（第11章〜第14章）を展開します。すべて**ファイルシステムに依存せず構文的に処理可能**なメソッドです。

---

## 📍第02部：パス判定・比較系（メソッド）

---

### 📝第11章\_`.is_absolute()`

* パスが絶対パスかどうかを論理的に判定する
* ルートやドライブを含んでいれば `True`
* 例：`PurePath("/a/b").is_absolute() == True`

---

### 📝第12章\_`.match(pattern)`

* パス（通常は `.name`）が指定パターンに一致するかを判定
* ワイルドカード（`*`, `?`, `**` など）対応
* 例：`PurePath("a/b/file.txt").match("*.txt") == True`

---

### 📝第13章\_`.relative_to(other)`

* 指定パス `other` からの相対パスを返す
* 対象が親階層にない場合は `ValueError` を発生させる
* 例：`PurePath("a/b/c").relative_to("a") == PurePath("b/c")`

---

### 📝第14章\_`.is_relative_to(other)`（Python 3.9+）

* `.relative_to()` が例外になるかどうかを事前に判定する
* 相対的な関係であれば `True`、そうでなければ `False`
* 例：`PurePath("a/b/c").is_relative_to("a") == True`

---

続けて「第03部：パス変換・構成操作（メソッド）」の展開を希望される場合はご指示ください。
以下に、`PurePath_Obj_Use_Index` の第03部「パス変換・構成操作（メソッド）」の全目次（第15章〜第17章）を展開します。これらはすべて構文操作のみで完結し、ファイルの存在に依存しません。

---

## 📍第03部：パス変換・構成操作（メソッド）

---

### 📝第15章\_`.joinpath(*other)`

* 複数のパス要素を結合して新しいパスを構成する
* `/` 演算子と等価な機能
* 例：`PurePath("a").joinpath("b", "c") == PurePath("a/b/c")`

---

### 📝第16章\_`.with_name(name)`

* 末尾の名前部分（ファイル名）だけを置き換えた新しいパスを返す
* ディレクトリ末尾には使用不可（`ValueError`）
* 例：`PurePath("a/b.txt").with_name("c.md") == PurePath("a/c.md")`

---

### 📝第17章\_`.with_suffix(suffix)`

* 末尾ファイル名の拡張子だけを変更する
* `suffix` はピリオド付きで渡す必要がある（例：`.md`）
* 例：`PurePath("a/b.txt").with_suffix(".csv") == PurePath("a/b.csv")`

---

続けて「第04部：文字列表現・変換（メソッド）」を展開する場合は指示してください。

以下に、`PurePath_Obj_Use_Index` の第04部「文字列表現・変換（メソッド）」の全目次（第18章〜第19章）を展開します。これらは `PurePath` を文字列形式に変換するためのユーティリティメソッドです。

---

## 📍第04部：文字列表現・変換（メソッド）

---

### 📝第18章\_`.as_posix()`

* 現在のパスを POSIX 形式（スラッシュ区切り）で文字列化する
* Windows 環境でも `/` 区切りに統一されるため、クロスプラットフォーム対応に有用
* 例：`PurePath("a\\b\\c").as_posix() == "a/b/c"`（Windows）

---

### 📝第19章\_`.as_uri()`

* 絶対パスを `file://` スキームの URI 形式に変換して返す
* **絶対パスでなければ例外 `ValueError` が発生する点に注意**
* 例（POSIX）：`PurePath("/home/user/file.txt").as_uri() == "file:///home/user/file.txt"`
* 例（Windows）：`PureWindowsPath("C:/Users/User/file.txt").as_uri() == "file:///C:/Users/User/file.txt"`

---

続けて「第05部：演算子と比較（オーバーロード）」を展開する場合は指示してください。
以下に、`PurePath_Obj_Use_Index` の第05部「演算子と比較（オーバーロード）」の全目次（第20章〜第22章）を展開します。これらは `PurePath` オブジェクトに対して Python の演算子がどのように働くかを扱います。

---

## 📍第05部：演算子と比較（オーバーロード）

---

### 📝第20章\_`/` 演算子（パス結合）

* `PurePath` オブジェクトと文字列または他のパスを結合する
* `joinpath()` の構文糖衣（シンタックスシュガー）
* 例：`PurePath("a") / "b" / "c"` → `PurePath("a/b/c")`

---

### 📝第21章\_`==` / `!=` / `<` / `>`（パス比較）

* `PurePath` 同士の辞書順比較や同値判定に使用される
* ファイルシステム上の実体とは無関係に「文字列としての順序」を評価
* 例：`PurePath("a/b") < PurePath("a/c") == True`

---

### 📝第22章\_`in` 演算子（部分一致）

* `in` によって `PurePath` の一部として他のパスが含まれているかを確認したい場合がある
* ただし、`in` は文字列のようには使えず、明示的な `.parts` による判定が必要
* 誤用例：`PurePath("a/b/c") in PurePath("a/b/c/d")` → `False`（期待動作ではない）

---

以上で「PurePath\_Obj\_Use\_Index」は**全5部・全22章**の展開が完了しました。

必要に応じて Obsidian 用に YAML メタ情報を付与したインデックス形式や、全体一括出力も対応可能です。指示があれば実行します。
