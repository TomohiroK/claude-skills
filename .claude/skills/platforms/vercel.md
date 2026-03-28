---
description: Vercel プラットフォームの設計・デプロイ・運用知識。デプロイ設定、Edge Functions、Serverless Functions、ドメイン管理、ISR/SSR/SSG、パフォーマンス最適化を含む。
---

# Vercel スキル

## 作業開始チェックリスト

1. フレームワーク（Next.js, Nuxt, SvelteKit, Astro 等）とバージョンを確認する
2. `vercel.json` の存在と設定内容を確認する
3. 必要な環境変数が Vercel ダッシュボードに設定されているか確認する
4. カスタムドメインの設定状態、SSL 証明書を確認する
5. Hobby / Pro / Enterprise のプランを確認し、利用可能な機能を把握する

## コーディングルール

### デプロイ設定
- `vercel.json` でビルドコマンド・出力ディレクトリ・リダイレクト/リライトを管理する
- プレビューデプロイ / プロダクションデプロイの環境変数を分離する
- ビルドキャッシュを活用する（`node_modules`, `.next/cache` 等）
- Monorepo の場合、Root Directory の設定を明示する

### Edge Middleware
- Edge Middleware は軽量に保つ（実行時間制限あり）
- 地理情報（`request.geo`）/ ユーザーエージェント / Cookie に基づくルーティングに使用する
- 重い処理は Serverless Functions に委譲する
- `matcher` config でミドルウェアの適用パスを制限する（全パス適用を避ける）

### Serverless Functions
- API Routes は `/api/` ディレクトリに配置する
- コールドスタートを考慮し、関数サイズを最小化する
- リージョン設定: ユーザーの地理に近いリージョンを選択する
- タイムアウト: Hobby 10秒 / Pro 60秒 / Enterprise 900秒 を考慮する
- Function の最大サイズ: 50MB（依存関係含む）を超えないようバンドルを最適化する

### パフォーマンス
- 静的ページは SSG / ISR を優先する（SSR は最終手段）
- ISR の `revalidate` 期間はコンテンツの更新頻度に合わせる
- 画像は `next/image`（Next.js）で自動最適化する
- `vercel.json` の `headers` でキャッシュヘッダーを適切に設定する

### 環境変数
- `NEXT_PUBLIC_` プレフィックスはクライアント公開可能な値のみに使用する
- シークレット（API キー等）は `NEXT_PUBLIC_` プレフィックスを付けない
- Production / Preview / Development の環境変数を適切に分離する

## セキュリティ

- 環境変数に API キー・DB 接続文字列を正しく設定する（ソースコードにハードコードしない）
- Edge Middleware で認証・認可のゲートを実装する（該当する場合）
- `X-Frame-Options`, `Content-Security-Policy` 等のセキュリティヘッダーを設定する
- プレビューデプロイへのアクセス制限を検討する（Vercel Authentication / Password Protection）
- Serverless Functions のエラーレスポンスに内部情報を含めない

## テスト

- **義務**: プロダクションデプロイ前にプレビューデプロイで全機能を検証する
- **テスト種類**:
  - ローカルテスト: `vercel dev` でローカル環境での動作確認
  - プレビューテスト: PR ごとのプレビューデプロイでの E2E テスト
  - パフォーマンステスト: Vercel Speed Insights / Lighthouse でスコア確認
  - Edge テスト: Edge Middleware / Functions の動作を複数リージョンから確認
- **完了前必須**: プレビューデプロイで全ページの表示・機能が正常であることを確認する

## 完了条件

- [ ] デプロイが正常に完了している
- [ ] カスタムドメインが正しく設定されている
- [ ] 環境変数が適切に設定されている（本番 / プレビュー分離）
- [ ] セキュリティヘッダーが設定されている
- [ ] Lighthouse スコアが目標値を満たしている（該当する場合）
- [ ] Edge Middleware / Serverless Functions が正常に動作している（該当する場合）

## 連携先エージェント

- **devops-automator**: CI/CD パイプラインとの統合（GitHub Actions → Vercel）
- **frontend-developer**: Next.js / フレームワーク固有の設定最適化
- **cloudflare-expert → platforms/cloudflare**: Cloudflare CDN + Vercel オリジンの構成設計
- **neon-expert → platforms/neon**: Vercel Postgres (Neon) との接続設定
- **seo-specialist**: ISR / SSR / SSG がSEOに与える影響の確認
