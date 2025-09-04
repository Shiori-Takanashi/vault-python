---
title: 
keyword: 
created: <% tp.file.creation_date() %>
modified: <% tp.file.last_modified_date() %>
vault: technology
catergory: Language
language: Python
area: Standard_Library
identify:  
type: Lesson...
tole: Index
order: 00
---

# 📢Os_Lesson_Index

---

## 📍第01部：基礎ファイル操作

---

### 📝第01章_`os` モジュールとは何か

* Python標準ライブラリにおける役割
* プラットフォーム依存処理の抽象化
* 他モジュールとの補完関係（`shutil`, `pathlib` 等）

### 📝第02章_OS情報の取得

* `os.name` / `os.uname()` / `os.getlogin()`
* OS種別やユーザ情報の把握
* プラットフォーム判定の実用例

### 📝第03章_環境変数の取得と操作

* `os.environ` の辞書的操作
* `getenv()` / `putenv()` の違い
* セキュリティ上の注意点と活用場面

### 📝第04章_カレントディレクトリの取得と変更

* `os.getcwd()` / `os.chdir()`
* 実行パスとモジュールパスの区別
* スクリプト内での相対移動制御

### 📝第05章_ファイル・ディレクトリの存在確認と作成

* `os.path.exists()` / `os.makedirs()`
* `isfile()` / `isdir()` の判別
* 再帰的なディレクトリ生成の安全設計

### 📝第06章_ファイルの削除・移動・名前変更

* `os.remove()` / `os.rename()` / `os.replace()`
* エラー処理と例外対応の設計
* `shutil.move()` との違い

---

## 📍第02部：パス操作と情報取得

---

### 📝第07章_パス結合と正規化

* `os.path.join()` / `normpath()` の活用
* `abspath()` / `realpath()` の違い
* マルチOS対応の構文設計

### 📝第08章_パスの分解と要素取得

* `basename()` / `dirname()` / `splitext()`
* 拡張子・親ディレクトリ・ファイル名の抽出
* 拡張子判定処理の実用例

### 📝第09章_ディレクトリ内の一覧取得

* `os.listdir()` の基本と応用
* `.`, `..` の排除処理
* `scandir()` / `walk()` による拡張

### 📝第10章_ファイルサイズと属性の取得

* `os.path.getsize()` / `getmtime()` など
* タイムスタンプの種類と変換
* ファイルの更新順に並べる処理例

### 📝第11章_一時ディレクトリとユーザパスの取得

* `os.getenv("TEMP")` / `gettempdir()`（`tempfile`）
* `os.path.expanduser()` による `~` 解決
* クロスプラットフォーム環境への配慮

### 📝第12章_シンボリックリンクと実パスの扱い

* `os.symlink()` / `readlink()` の構文
* WindowsとUnixの違い
* 実パス解決のユースケース

---

## 📍第03部：プロセス・システム関連操作

---

### 📝第13章_プロセスの起動と実行

* `os.system()` の基本と限界
* `os.spawn*()` 系の使い分け
* `subprocess` との比較と推奨構成

### 📝第14章_プロセスIDと制御

* `os.getpid()` / `getppid()` の取得
* 実行中プロセスの識別
* 親子関係のトレース設計

### 📝第15章_終了ステータスと例外終了

* `os._exit()` と `sys.exit()` の違い
* 終了コードの明示と運用
* 実行環境との連携（CI/CD）

### 📝第16章_ユーザー・グループ情報

* `os.getuid()` / `getgid()` / `geteuid()`
* 権限の動的確認
* 非Unix系での代替処理設計

### 📝第17章_CPU数とリソース情報の取得

* `os.cpu_count()` / `os.sysconf()`
* 並列処理計画への応用
* マシン依存コードの排除方法

### 📝第18章_カレント実行ファイル情報の取得

* `os.path.realpath(__file__)`
* スクリプトの配置情報と実行環境の特定
* フレームワーク内での応用事例

---

## 📍第04部：実践応用と注意点

---

### 📝第19章_os.walk() を用いた再帰探索

* ファイル再帰走査のパターン
* 特定拡張子の抽出処理
* 処理効率と構文簡潔性の両立

### 📝第20章_ファイル操作時の例外対策

* `FileNotFoundError`, `PermissionError` 等の検知
* `try` / `except` の適用例
* 破壊的操作の安全性設計

### 📝第21章_権限変更とモード設定

* `os.chmod()` の基本構文と数値指定
* 読み書き実行のビット演算
* クロスプラットフォーム時の制限

### 📝第22章_一時ファイルの作成と削除

* `tempfile` との連携例
* `delete=False` の運用
* 一時保存と明示削除の責務設計

### 📝第23章_低レベル操作と安全性

* `os.open()` / `read()` / `write()` の非推奨理由
* 高レベルAPIとの使い分け
* 文字コードとバイナリの落とし穴

---

## 📍第05部：他モジュールとの連携と比較

---

### 📝第24章_`os` モジュールの限界と代替

* `pathlib` / `shutil` との役割分担
* 推奨構文への移行指針
* `os` 使用判断基準の整理

### 📝第25章_`os` と `pathlib` の基本構文比較

* `os.path.join()` vs `Path() /`
* `os.path.exists()` vs `Path.exists()`
* 可読性と構文の自然さの違い

### 📝第26章_ファイル・ディレクトリ操作の比較

* `os.mkdir()` / `os.makedirs()` vs `Path.mkdir()`
* `os.remove()` vs `Path.unlink()`
* 例外処理と明示的制御のしやすさ

### 📝第27章_パス操作における違いと統一的記述

* パス分割：`os.path.basename()` vs `Path.name`
* 拡張子処理：`os.path.splitext()` vs `Path.suffix`
* パスオブジェクト化による一貫したインターフェース

### 📝第28章_`shutil` による高水準ファイル操作

* `os.rename()` / `os.replace()` vs `shutil.move()`
* `shutil.copy() / copy2()` の使い分け
* 再帰コピーや削除：`rmtree()` との連携

### 📝第29章_使い分けの原則と推奨

* `os` の利用は必要最小限にとどめる
* path/ファイル操作：`pathlib` / `shutil` 優先
* クロスプラットフォーム・可読性・保守性の観点からの指針

---