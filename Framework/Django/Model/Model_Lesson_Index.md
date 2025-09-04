---
title: 
keyword:
created: 2025-07-29 03:21
modified: 2025-07-29 03:21
vault: technology
catergory: Language
language: Python
area: 
identify:  
Type: Lesson...
Role: Index...
Order: 00...
---

# Django_Model_Lesson_Index

---

## 📍**第01部：Djangoモデルの基礎**

---

### 📄 **第01章：Djangoモデルの概要**

* ORMとモデルの関係
* モデルクラスの基本構造
* モデル定義のメリットと責務
* モデルとマイグレーションのつながり

---

### 📄 **第02章：モデルクラスの定義方法**

* `models.Model` の継承
* クラス定義とフィールド定義の書式
* コーディング規約と命名原則
* `__str__()` と `Meta` クラスの利用

---

### 📄 **第03章：主要なフィールド型**

* `CharField`, `TextField`, `IntegerField`, `FloatField`, `DecimalField`
* 引数（`max_length`, `null`, `blank`, `default`など）の意味
* バリデーションと保存処理の関係

---

### 📄 **第04章：主要なフィールド型**

* `DateField`, `TimeField`
* `DateTimeField`, `BooleanField`, `NullBooleanField`
* `auto_now`, `auto_now_add` の注意点
* タイムゾーンとの関係

---

### 📄 **第05章：リレーションフィールド**

* `ForeignKey`, `ManyToManyField`, `OneToOneField` の使い分け
* `on_delete` 引数の選択肢と影響
* 逆参照名の指定：`related_name`, `related_query_name`

---

## 📍**第02部：モデルの拡張と管理**

---

### 📄 **第06章：規定値・選択肢・ヘルプ**

* `default` による初期値指定
* `choices` による列挙定義
* `help_text`, `verbose_name` による管理画面の整備

---

### 📄 **第07章：Metaクラスの設定**

* `ordering`, `verbose_name`, `db_table`, `unique_together` など
* モデルクラスへの追加制御
* 管理者向け表記とデータベース構造の制御

---

### 📄 **第08章：モデルのマイグレーション**

* `makemigrations` と `migrate` の違い
* マイグレーションファイルの生成・内容確認
* 既存モデルの変更時の注意点
* マイグレーションの履歴管理とリセット

---

### 📄 **第09章：モデルのバリデーション**

* `clean()` メソッドの実装方法
* `full_clean()` を用いた全体検証
* `validators` の適用とカスタムバリデータの定義
* `ModelForm` との連携

---

### 📄 **第10章：モデルメソッドの活用**

* インスタンスメソッド（例：`get_full_name()`）の定義
* クラスメソッド（`@classmethod`）の設計例
* カスタム保存処理（`save()` のオーバーライド）

---

## 📍**第03部：DBの操作・最適化**

---

### 📄 **第11章：クエリセット関連の定義**

* `objects` マネージャとその役割
* カスタムマネージャとQuerySetの定義方法
* よく使うQuerySetフィルタ条件のまとめ
* 管理画面でのフィルタの活用

---

### 📄 **第12章：リレーション逆参照**

* `related_name` でのアクセス例（例：`user.profile`）
* `set.all()` でのManyToMany逆参照
* `select_related` と `prefetch_related` による効率化

---

### 📄 **第13章：ユニーク制約と複合制約**

* `unique=True` の使い方
* `unique_together` / `constraints` での複合制約
* バリデーションとの違いと目的の切り分け

---

### 📄 **第14章：プロパティ**

* `@property` を使った仮想属性の定義
* 読み取り専用属性
* 管理画面への表示と活用
* DBには保存しないが表示だけ必要なケースの対応

---

### 📄 **第15章：多対多関係の中間モデル**

* 中間テーブルを明示的に定義するパターン
* `through` 引数の使い方
* 中間モデルに追加属性を持たせる方法

---

### 📍**第04部：モデルの応用と管理画面**

---

### 📄 **第16章：ファイル・画像**

* `FileField`, `ImageField` の基本構文
* `MEDIA_ROOT`, `MEDIA_URL` 設定
* アップロード先の指定と管理
* ファイル削除のタイミングと副作用

---

### 📄 **第17章：自動フィールドとUUID**

* `AutoField`, `BigAutoField` の違い
* `UUIDField` による非連番ID設計
* `default=uuid.uuid4` の使い方とセキュリティ的意義

---

### 📄 **第18章：モデルにおける日時管理**

* 作成・更新日時を自動記録（`created_at`, `updated_at`）
* `auto_now`, `auto_now_add` の注意点
* ミドルウェアやモデル継承での共通化戦略

---

### 📄 **第19章：抽象基底クラスと継承**

* `abstract = True` による親クラス定義
* `Multi-table Inheritance` とその制限
* 抽象クラスのフィールド再利用と制御の分離

---

### 📄 **第20章：モデルレベルの最適化**

* データベースインデックスの設定方法
* `db_index=True` の効果と注意点
* `unique` との違いと共存の設計指針

---

### 📍**第05部：運用とテスト**

---

### 📄 **第21章：モデルシグナルの活用**

* `post_save`, `pre_save`, `post_delete` の基本構文
* 実用例：プロファイル自動作成・ログ記録など
* 注意点：非同期処理やパフォーマンス影響

---

### 📄 **第22章：初期データ投入**

* `fixtures` と `loaddata` の使い方
* JSON/YAML形式での定義
* Djangoコマンドからのバッチ登録方法

---

### 📄 **第23章：管理画面での表示設定**

* `list_display`, `search_fields`, `list_filter` の調整
* `readonly_fields` や `fieldsets` によるUI制御
* 管理画面でのリレーション表示の工夫

---

### 📄 **第24章：テストと保証**

* `TestCase` でのモデル検証
* 作成・更新・削除操作の自動テスト
* フィールド制約のテスト観点
* テストDBでのマイグレーション活用方法

---