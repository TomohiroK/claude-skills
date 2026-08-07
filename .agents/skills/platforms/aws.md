---
description: Amazon Web Services（AWS）の設計・構築・運用知識。EC2、Lambda、S3、RDS、CloudFront、IAM、VPC、コスト最適化を含む。
---

# AWS スキル

## 作業開始チェックリスト

1. AWS アカウント構成（Organizations / 単一アカウント）を把握する
2. 使用するリージョン、マルチリージョン要件を確認する
3. 既存の VPC / サブネット / セキュリティグループ構成を把握する
4. 既存の IaC（CloudFormation / CDK / Terraform）の有無と構成を確認する
5. 予算上限、コスト配分タグの設定方針を確認する

## コーディングルール

### IAM 設計
- Root アカウントは使用しない。MFA を有効化する
- IAM ポリシーは最小権限の原則に従う（`*` リソースを避ける）
- サービス間通信には IAM ロール（AssumeRole）を使用する（アクセスキー禁止）
- IAM ポリシーの Condition で IP 制限・MFA 要件を設定する
- SCPs（Service Control Policies）で Organizations レベルの制限を設定する

### VPC / ネットワーク
- パブリックサブネット / プライベートサブネットを分離する
- データベースはプライベートサブネットに配置する（パブリック IP 禁止）
- セキュリティグループは最小限のポートのみ許可する（`0.0.0.0/0` インバウンドは原則禁止）
- VPC フローログを有効化する
- NAT Gateway はコスト考慮。開発環境では NAT Instance や VPC Endpoint を検討する

### Lambda
- ランタイムは最新の LTS バージョンを使用する
- メモリは処理に応じて最適化する（メモリ増 = CPU 増 = 実行時間短縮の可能性）
- コールドスタート対策: Provisioned Concurrency / SnapStart（Java）を検討する
- 環境変数でシークレットを管理する場合、Secrets Manager / Parameter Store との統合を推奨
- Lambda Layers で共通ライブラリを共有する

### S3
- バケットポリシーで公開アクセスをデフォルトブロックする（Block Public Access）
- サーバーサイド暗号化（SSE-S3 / SSE-KMS）を有効化する
- バージョニングを有効化する（重要なバケット）
- ライフサイクルポリシーで不要なオブジェクトを自動削除 / Glacier 移行する
- CloudFront 経由のアクセスには OAC（Origin Access Control）を使用する

### コスト最適化
- 全リソースにコスト配分タグ（`env`, `service`, `owner`）を付与する
- AWS Budgets でアラートを設定する（予算の 50% / 80% / 100%）
- 開発環境のリソースは夜間 / 週末に停止する（Lambda Scheduler / EventBridge）
- Reserved Instances / Savings Plans で常時稼働リソースのコストを削減する
- 未使用リソース（未アタッチ EBS、古い AMI / スナップショット）を定期的にクリーンアップする

## セキュリティ

- CloudTrail を全リージョンで有効化する（管理イベント + データイベント）
- GuardDuty を有効化する（脅威検知）
- Config Rules でコンプライアンスチェックを自動化する
- Secrets Manager / Parameter Store でシークレットを管理する
- KMS でカスタムキーを作成し、暗号化を一元管理する
- SecurityHub でセキュリティスコアを継続的にモニタリングする

## テスト

- **義務**: IaC の変更は `terraform plan` / `cdk diff` で事前に差分を確認する
- **テスト種類**:
  - IaC 検証: `cfn-lint` / `cdk synth` / `terraform validate` で構文チェック
  - セキュリティ検証: `checkov` / `tfsec` でセキュリティポリシー違反をチェック
  - 統合テスト: ステージング環境で実際のリソース作成・動作を確認する
  - コストテスト: `infracost` でコスト見積りを確認する
- **完了前必須**: ステージング環境で全リソースが正常に動作していることを確認する

## 完了条件

- [ ] 全リソースが IaC で管理されている
- [ ] IAM ポリシーが最小権限で設定されている
- [ ] セキュリティグループが最小限のルールで構成されている
- [ ] CloudTrail / GuardDuty が有効化されている
- [ ] コスト配分タグが全リソースに付与されている
- [ ] AWS Budgets アラートが設定されている
- [ ] シークレットが Secrets Manager / Parameter Store で管理されている

## 連携先エージェント

- **devops-automator**: CI/CD パイプライン・IaC の統合
- **gcp → platforms/gcp**: マルチクラウド構成時の設計調整
- **cloudflare → platforms/cloudflare**: CloudFront vs Cloudflare CDN の使い分け
- **backend-architect**: サーバーレス / コンテナアーキテクチャの設計
- **infrastructure-maintainer**: 障害対応・パフォーマンスチューニング
- **finance-tracker**: AWS コストの月次モニタリング
