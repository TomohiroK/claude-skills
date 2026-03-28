---
description: Neon（サーバーレス PostgreSQL）の設計・運用知識。ブランチング、接続プーリング（PgBouncer）、Vercel統合、Auto-scaling、スキーマ設計、マイグレーション管理を含む。
---

# Neon スキル

## 作業開始チェックリスト

1. Neon プロジェクト ID、リージョン、プランを確認する
2. 既存ブランチ（main, dev, preview 等）の構成を把握する
3. Direct Connection / Pooled Connection / Serverless Driver のどれを使うか確認する
4. 使用する ORM / クエリビルダー（Prisma, Drizzle, Kysely 等）を確認する
5. Auto-scaling の最小/最大 CU（Compute Units）を確認する

## コーディングルール

### 接続管理
- Serverless 環境（Vercel Functions 等）では Neon Serverless Driver（`@neondatabase/serverless`）を使用する
- 長時間接続が必要な場合は Pooled Connection（`:5432` → PgBouncer）を使用する
- Direct Connection（`:5432` 直結）はマイグレーション実行時のみ使用する
- 接続文字列は環境変数 `DATABASE_URL`（pooled）/ `DIRECT_URL`（direct）で管理する

### ブランチング
- 本番ブランチ（`main`）から開発ブランチを作成する（データのスナップショット付き）
- PR ごとにプレビューブランチを自動作成する（Vercel Integration）
- 不要になったブランチは速やかに削除する（ストレージコスト削減）
- ブランチ名は `dev/{feature-name}` / `preview/pr-{number}` で統一する

### マイグレーション
- Prisma: `prisma migrate dev`（開発）/ `prisma migrate deploy`（本番）
- Drizzle: `drizzle-kit push`（開発）/ `drizzle-kit migrate`（本番）
- マイグレーションは Direct Connection で実行する（PgBouncer 経由ではトランザクション DDL が不安定）
- 破壊的マイグレーション（カラム削除、型変更等）は2段階で実施する

### パフォーマンス
- Auto-suspend: アイドル状態での自動停止時間を設定する（開発: 5分、本番: 無効 or 長め）
- Auto-scaling: ワークロードに応じた CU 範囲を設定する
- コールドスタート（Auto-suspend からの復帰）時間を考慮する（~500ms）
- インデックス設計: `EXPLAIN ANALYZE` で実行計画を確認し、適切なインデックスを作成する
- `pg_stat_statements` で低速クエリを特定する

### Prisma / Drizzle 統合
- Prisma: `datasources.db.url` に Pooled URL、`directUrl` に Direct URL を設定する
- Drizzle: `drizzle-orm/neon-serverless` アダプターを使用する
- コネクションプールのサイズはサーバーレス環境に合わせて小さく設定する（`connection_limit=1` 等）

## セキュリティ

- 接続文字列（`DATABASE_URL`）をソースコードにハードコードしない
- Neon のロール（ユーザー）は最小権限で作成する（読み取り専用ロール等）
- IP 許可リスト（IP Allow）を設定する（Pro プラン以上）
- SSL 接続を強制する（`sslmode=require`）
- ブランチ削除時にセンシティブデータが残存しないことを確認する

## テスト

- **義務**: マイグレーションは開発ブランチで検証してから本番に適用する
- **テスト種類**:
  - マイグレーションテスト: 開発ブランチでスキーマ変更を検証する
  - 接続テスト: Pooled / Direct / Serverless の各接続方式を検証する
  - パフォーマンステスト: クエリの実行計画と実行時間を計測する
  - フェイルオーバーテスト: Auto-suspend → Resume の復帰時間を確認する
- **完了前必須**: 本番ブランチへのマイグレーションを開発ブランチで事前検証する

## 完了条件

- [ ] DB 接続が正常に動作している（Pooled / Direct / Serverless）
- [ ] マイグレーションが正常に適用されている
- [ ] ブランチ戦略が設計通りに構成されている
- [ ] Auto-scaling / Auto-suspend が適切に設定されている
- [ ] 接続文字列がソースコードにハードコードされていない
- [ ] インデックスが主要クエリに対して適切に設定されている

## 連携先エージェント

- **backend-architect**: スキーマ設計、ORM 選定、クエリ最適化
- **vercel → platforms/vercel**: Vercel Integration の設定、環境変数の連携
- **devops-automator**: マイグレーションの CI/CD パイプライン統合
- **data-engineer**: データパイプライン・ETL の設計
