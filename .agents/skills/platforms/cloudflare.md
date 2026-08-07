---
description: Cloudflare プラットフォームの設計・運用知識。DNS管理、CDN設定、Workers、Pages、R2、D1、WAF、Zero Trust、SSL/TLS設定を含む。
---

# Cloudflare スキル

## 作業開始チェックリスト

1. 対象の Cloudflare アカウント ID、ゾーン ID、ドメインを把握する
2. Free / Pro / Business / Enterprise のプランを確認し、利用可能な機能を把握する
3. DNS レコード、Page Rules、WAF ルール、Workers の現状を確認する
4. SSL モード（Flexible / Full / Full Strict）、証明書の状態を確認する
5. オリジンサーバーの構成（Vercel, AWS, GCP 等）を把握する

## コーディングルール

### DNS 管理
- DNS レコードの変更前に既存レコードを全てバックアップする
- プロキシ状態（オレンジ雲 / グレー雲）を意図的に設定する
- MX / TXT レコードはプロキシ不可。グレー雲で設定する
- TTL は Auto を基本とし、移行時のみ短縮する

### Workers
- Wrangler CLI を使用してデプロイする（`wrangler.toml` で構成管理）
- Worker のサイズ制限（1MB / Free、10MB / Paid）を考慮する
- Workers KV は結果整合性。強整合が必要な場合は Durable Objects を使用する
- Worker 内で `await` を適切に使い、CPU 時間制限（10ms / Free、30s / Paid）を超えない
- エラーハンドリングで `event.passThroughOnException()` を適切に使用する

### キャッシュ戦略
- `Cache-Control` ヘッダーをオリジンで正しく設定する
- 静的アセット: `max-age=31536000, immutable`（ハッシュ付きファイル名前提）
- HTML: `no-cache` または短い `max-age`（動的コンテンツ）
- API: `no-store`（キャッシュしない）
- Cloudflare の Cache Rules でパスベースのキャッシュ制御を行う

### セキュリティ設定
- SSL/TLS: Full (Strict) モードを推奨。Flexible は MITM リスクがあるため非推奨
- HSTS を有効化する（`max-age=31536000; includeSubDomains; preload`）
- WAF マネージドルールを有効化する（OWASP Core Ruleset）
- Bot Management: 既知の良性ボット（Google, Bing 等）を許可し、悪性ボットをブロック
- Rate Limiting: API エンドポイントにレート制限を設定する

### R2 / D1
- R2: S3 互換 API。既存の S3 SDK でアクセス可能
- R2 のパブリックアクセスはカスタムドメイン経由で提供する
- D1: SQLite ベース。読み取り多 / 書き込み少のワークロードに最適
- D1 のマイグレーションは Wrangler CLI で管理する

## セキュリティ

- Cloudflare API トークンは最小権限で発行する（ゾーン単位 / 権限単位）
- グローバル API キーは使用しない。スコープ付き API トークンを使用する
- WAF ルールの変更はステージング環境で検証してから本番に適用する
- Zero Trust Access ポリシーの変更は影響範囲を事前に確認する
- SSL 証明書の有効期限を監視し、期限切れ前に更新する

## テスト

- **義務**: DNS / SSL / キャッシュの変更は本番適用前に検証する
- **テスト種類**:
  - DNS 検証: `dig` / `nslookup` でレコードの解決を確認する
  - SSL 検証: `curl -vI` で証明書チェーン・TLS バージョンを確認する
  - キャッシュ検証: `cf-cache-status` ヘッダーで HIT / MISS を確認する
  - Workers 検証: `wrangler dev` でローカルテスト後、ステージングでデプロイテスト
  - WAF 検証: テスト用リクエストでルールの発火を確認する
- **完了前必須**: 本番ドメインで SSL / DNS / キャッシュが正常に動作していることを確認する

## 完了条件

- [ ] DNS レコードが正しく解決されている
- [ ] SSL/TLS が Full (Strict) で動作している
- [ ] キャッシュ戦略が設計通りに動作している（cf-cache-status で確認）
- [ ] WAF ルールが有効化されている
- [ ] API トークンが最小権限で設定されている
- [ ] Workers / Pages のデプロイが正常に動作している（該当する場合）

## 連携先エージェント

- **devops-automator**: CI/CD パイプラインとの統合（Wrangler デプロイ）
- **vercel → platforms/vercel**: Vercel オリジン + Cloudflare CDN の構成設計
- **aws/gcp → platforms/aws, platforms/gcp**: マルチクラウドオリジンの構成設計
- **seo-specialist**: キャッシュ設定が SEO に与える影響の確認
- **infrastructure-maintainer**: 障害時の DNS フェイルオーバー設計
