---
description: Google Cloud Platform（GCP）の設計・構築・運用知識。Cloud Run、Cloud Functions、Cloud Storage、Cloud SQL、BigQuery、IAM、VPC、モニタリングを含む。Firebase は platforms/firebase に委譲。
---

# GCP スキル

## 作業開始チェックリスト

1. GCP プロジェクト ID、組織構成、フォルダ階層を把握する
2. 使用するリージョン（asia-northeast1 等）を確認する
3. 既存のサービス・VPC・IAM 構成を把握する
4. 必要な API が有効化されているか確認する
5. 課金アカウント、予算アラートの設定を確認する

## コーディングルール

### IAM 設計
- サービスアカウントは最小権限の原則に従う
- 基本ロール（Owner / Editor / Viewer）は使用しない。事前定義ロールまたはカスタムロールを使用する
- サービスアカウントキー（JSON）の発行を最小化する。Workload Identity Federation を推奨する
- IAM Conditions でリソース・時間ベースの条件を設定する
- Organization Policy で組織レベルの制限を設定する

### Cloud Run
- コンテナイメージは Artifact Registry に保存する
- 最小インスタンス数: 開発 0 / 本番 1以上（コールドスタート回避）
- CPU / メモリの上限を適切に設定する
- 認証: `--no-allow-unauthenticated`（内部サービス）/ `--allow-unauthenticated`（公開 API）
- Cloud Run + Cloud SQL は Unix ソケット接続（Cloud SQL Auth Proxy）を使用する

### Cloud Functions
- 第2世代（Gen 2 = Cloud Run ベース）を推奨
- ランタイムは最新の LTS バージョンを使用する
- 環境変数でシークレットを管理する場合、Secret Manager との統合を使用する
- イベントトリガー: Pub/Sub, Cloud Storage, Firestore, HTTP
- 同時実行数とインスタンス数の上限を設定する

### Cloud Storage
- バケットの Uniform Bucket-level Access を有効化する（ACL を使用しない）
- デフォルトでパブリックアクセスを禁止する
- オブジェクトのライフサイクルルールを設定する（Nearline / Coldline / Archive への自動移行）
- 署名付き URL で一時的なアクセスを提供する（直接パブリックアクセスは避ける）

### Firebase 連携
- Firebase 固有の設計・運用は `platforms/firebase` スキルに委譲する
- GCP 側の設定（IAM、サービスアカウント、Cloud Run 連携等）は本スキルが対応する

### BigQuery
- パーティション分割テーブルでコストとパフォーマンスを最適化する
- クラスタリングで頻繁に使用するフィルタカラムを指定する
- `SELECT *` を避け、必要なカラムのみ選択する
- コスト管理: カスタムクォータで最大スキャン量を制限する
- スケジュールクエリで定期的なデータ集計を自動化する

### コスト最適化
- 全リソースにラベル（`env`, `service`, `owner`）を付与する
- Billing Budgets でアラートを設定する
- Committed Use Discounts（CUD）で常時稼働リソースのコストを削減する
- Recommender API の推奨事項を定期的に確認する
- 未使用リソース（未アタッチディスク、古いスナップショット）を定期的にクリーンアップする

## セキュリティ

- Cloud Audit Logs を全サービスで有効化する
- VPC Service Controls でデータの境界を設定する（該当する場合）
- Security Command Center でセキュリティの問題を継続的にモニタリングする
- Secret Manager でシークレットを管理する
- CMEK（Customer-Managed Encryption Keys）で暗号化を管理する（該当する場合）

## テスト

- **義務**: IaC の変更は `terraform plan` で事前に差分を確認する
- **テスト種類**:
  - IaC 検証: `terraform validate` / `checkov` でセキュリティポリシー違反をチェック
  - Firebase 検証: `platforms/firebase` スキルに委譲
  - 統合テスト: ステージングプロジェクトで実際のリソース作成・動作を確認する
  - コストテスト: BigQuery のドライランでスキャン量を事前確認する
- **完了前必須**: ステージング環境で全サービスが正常に動作していることを確認する

## 完了条件

- [ ] 全リソースが IaC で管理されている
- [ ] IAM が最小権限で設定されている
- [ ] Firebase 使用時は `platforms/firebase` スキルと連携済み
- [ ] Cloud Audit Logs が有効化されている
- [ ] ラベルが全リソースに付与されている
- [ ] Billing Budgets アラートが設定されている
- [ ] シークレットが Secret Manager で管理されている

## 連携先エージェント

- **devops-automator**: CI/CD パイプライン（Cloud Build）・IaC の統合
- **aws → platforms/aws**: マルチクラウド構成時の設計調整
- **firebase → platforms/firebase**: Firebase 固有の設計・運用
- **google-analytics → platforms/google-analytics**: GA4 + BigQuery 連携の設計
- **google-tts → platforms/google-tts**: GCP 上の音声 API サービスの構成
- **backend-architect**: サーバーレス / コンテナアーキテクチャの設計
- **infrastructure-maintainer**: 障害対応・パフォーマンスチューニング
- **finance-tracker**: GCP コストの月次モニタリング
