---
description: Google Analytics 4 / Google Tag Manager の設計・実装知識。GA4プロパティ設定、GTMコンテナ管理、カスタムイベント、コンバージョン計測、dataLayer設計、デバッグを含む。
---

# Google Analytics スキル

## 作業開始チェックリスト

1. 測定 ID（G-XXXXXXXXXX）、ストリーム設定、既存イベント一覧を把握する
2. コンテナ ID（GTM-XXXXXXX）、既存タグ/トリガー/変数の構成を把握する
3. SPA/MPA の区別、使用フレームワーク（Next.js, Nuxt 等）、ルーティング方式を確認する
4. 現在計測中のイベント・コンバージョン、カスタムディメンション/メトリクスを確認する
5. Cookie 同意バナーの有無、GDPR/個人情報保護法への対応状態を確認する

## コーディングルール

### dataLayer 設計
- dataLayer.push のスキーマは事前に型定義する（TypeScript 推奨）
- イベント名は snake_case で統一する（GA4 の推奨命名規則に準拠）
- eコマースイベントは GA4 の標準スキーマに準拠する（`view_item`, `add_to_cart`, `purchase` 等）
- `dataLayer.push({ ecommerce: null })` で eコマースオブジェクトをクリアしてから新しいイベントを送信する

### GTM 設計
- タグの命名規則: `[種類] - [目的] - [トリガー条件]`（例: `GA4 - purchase - thank_you_page`）
- 全タグにバージョンメモを記載する（いつ・誰が・なぜ追加/変更したか）
- カスタム HTML タグは最終手段。GA4 タグ・カスタムイベントタグを優先する
- Lookup Table / RegEx Table 変数を活用し、タグの数を最小化する

### 計測精度
- `debug_mode: true` パラメータを開発環境で有効化し、DebugView で検証する
- 本番デプロイ前に GTM プレビューモードで全イベントの発火を確認する
- 内部トラフィック除外フィルタを設定する（IP / GTM 変数ベース）
- クロスドメイン計測が必要な場合、ドメインリストを明示的に設定する

### SPA 対応
- `page_view` イベントはルート変更時に手動送信する（History API / Router イベント）
- SPA では GTM の「履歴の変更」トリガーまたはカスタムイベントトリガーを使用する
- Next.js の場合、`next/router` の `routeChangeComplete` で計測する

## セキュリティ

- GA4 測定 ID / API Secret をソースコードにハードコードしない
- Measurement Protocol の API Secret は環境変数で管理する
- GTM コンテナのアクセス権限は最小権限の原則に従う（閲覧のみ / 編集 / 公開 を分離）
- GTM カスタム HTML 内でユーザー入力を直接使用しない（XSS 防止）
- サーバーサイド GTM のエンドポイントにレート制限を設定する

## テスト

- **義務**: 全イベントの発火を GTM プレビューモード + GA4 DebugView で検証する
- **テスト種類**:
  - ユニットテスト: dataLayer.push の出力値を検証する
  - 統合テスト: GTM プレビューモードで タグ発火 → GA4 DebugView 到達を確認する
  - E2E テスト: コンバージョンファネル全体の計測を通しで確認する
  - リグレッション: タグ変更後、既存イベントが引き続き正常に発火することを確認する
- **完了前必須**: DebugView でイベントパラメータの値が期待通りであることをスクリーンショット付きで記録する

## 完了条件

- [ ] 全イベントが GA4 DebugView で正常に記録されている
- [ ] GTM プレビューモードで全タグの発火条件が正しい
- [ ] eコマースイベントのスキーマが GA4 標準に準拠している
- [ ] 内部トラフィック除外が設定されている
- [ ] Cookie 同意との連携が正しく動作している（該当する場合）
- [ ] イベント設計ドキュメントが作成されている

## 連携先エージェント

- **CMO**: 計測要件の受領、KPI に基づくイベント設計の方針決定
- **analytics-reporter**: 計測データの分析・レポート作成を引き継ぐ
- **frontend-developer**: dataLayer.push の実装箇所・タイミングの調整
- **seo-specialist**: 構造化データとの整合性確認
- **legal-compliance-checker**: プライバシーポリシー・Cookie 同意の法的要件確認
- **cloudflare → platforms/cloudflare**: サーバーサイド GTM のプロキシ設定
