---
title: 
keyword:
created: 2025-07-29 06:26
modified: 2025-07-29 06:26
vault: technology
catergory: Language
language: Python
area: Django
identify: Admin
Type: Lesson
Role: Index
Order: 00
---


# 📢GitHub\_Actions\_Python\_Lesson\_Index

<hr>

## 📍第1部 基礎理解

<hr>

### 📝第01章 GitHub Actionsの全体像

* ワークフロー ジョブ ステップ ランナーの関係
* eventsとfilters push pull\_request workflow\_dispatch schedule
* ホストランナーとセルフホストの使い分け
* 成果物 アーティファクトとキャッシュの違い

### 📝第02章 最小CIの設計指針

* 目的を明確化 テスト 静的解析 配布 自動実行
* 失敗の定義と終了コード
* ステージ分割と依存関係 needs の基本
* ジョブ単位の失敗隔離

### 📝第03章 uv環境の前提整理

* uvによるピン留め pyproject.toml uv.lock
* .venv運用とランナー上のパス
* キャッシュ対象と非対象 .venv uvのhttpキャッシュ wheels

### 📝第04章 秘密情報の扱い

* repository secrets と environment secrets
* OpenAI APIキーの注入方法 env か with か
* マスクとログ汚染対策 set -x 禁止 原則print禁止
* 機密の境界と責任分界

### 📝第05章 ブランチ戦略と保護

* 保護対象ブランチへの必須チェック required status checks
* 強制push禁止 レビュールール 併用の現実

<hr>

## 📍第2部 実装テンプレート

<hr>

### 📝第06章 Python CIベースライン

* 目的 ユニットテストとコード品質の担保
* サンプル ci.yml



### 📝第07章 マトリクス戦略の導入

* 3.10 3.11 3.12を並列検証
* allow-failureの線引き



### 📝第08章 OpenAI統合の安全実行

* secrets.OPENAI\_API\_KEY の注入
* 統合テストをPRで回さない選別



### 📝第09章 速度最適化

* ruff の差分実行 ruff --force-exclude
* pytestのテスト選別 -m small -k expr
* キャッシュヒット率向上の鍵の設計 hashFilesの粒度調整

### 📝第10章 静的解析と型検査

* ruff mypy banditの役割分担
* エラー基準の合意と段階的厳格化



### 📝第11章 成果物の保存

* アーティファクト保存と保持期間



### 📝第12章 スケジュール実行

* cronで夜間ジョブを実行
* OpenAI連携スクリプトの定期実行



### 📝第13章 条件分岐とスキップ

* paths-ignore
* 変更ファイル検知でテスト縮小



### 📝第14章 再利用可能ワークフロー

* .github/workflows/reusable-ci.yml の切り出し
* 呼び出し側で入力を渡す



### 📝第15章 コンポジットアクション

* .github/actions/setup-uv の自作でDRY化
* inputsとoutputsの設計

<hr>

## 📍第3部：運用・実践

<hr>

### 📝第16章 セキュリティ強化

* permissions最小化 contents readのみから開始
* GITHUB\_TOKENの権限制御



### 📝第17章 環境保護ルール

* environmentsで本番鍵を保護 reviewers 必須待機時間
* PRからの本番鍵アクセス禁止

### 📝第18章 シークレット漏洩防止

* ログ出力の抑制 set -eux 禁止
* 例外時に環境変数を表示しない
* マスキングと静的検査 trufflehog の導入判断

### 📝第19章 リトライとフレーク対処

* pytest-rerunfailures
* ネットワーク外部要因と再試行戦略
* OpenAIレート制限時の待機

### 📝第20章 リリース自動化

* タグトリガ release.yml
* バージョン刻みとCHANGELOG生成
* PyPIや内部配布の判断

### 📝第21章 モノレポ運用

* パスフィルタで影響範囲のみビルド
* 並列化とキャッシュ分離

### 📝第22章 コストと実行時間

* ジョブ短縮の打ち手 依存の段階化
* 秘密鍵の使用最小化で監査容易化

### 📝第23章 障害対応手順

* ワークフロー再実行方針
* 緊急停止 concurrencyでのキャンセル
* 直近変更のロールバック基準

### 📝第24章 監査ログと可視化

* Job Summary出力で要点を残す



<hr>

