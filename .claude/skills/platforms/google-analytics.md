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

## MCP サーバー経由の GA4 データアクセス

### 概要
Google 公式 MCP サーバー（`analytics-mcp`）を使い、Claude Code から GA4 データに直接アクセスできる。
読み取り専用 — GA4 の設定変更はできない。

### 利用可能ツール

| ツール | 説明 | ユースケース |
|--------|------|-------------|
| `get_account_summaries` | アクセス可能な全 GA アカウント・プロパティの一覧 | 対象プロパティの特定、複数プロパティの横断把握 |
| `get_property_details` | 特定プロパティの詳細情報 | プロパティ設定の確認 |
| `run_report` | レポート実行（ディメンション、メトリクス、フィルタ、日付範囲） | トラフィック分析、コンバージョン分析、ユーザー行動分析 |
| `run_realtime_report` | リアルタイムレポート | 現在のアクティブユーザー、デプロイ直後の動作確認 |
| `get_custom_dimensions_and_metrics` | カスタムディメンション・メトリクス一覧 | 計測設計の確認、レポート設計の前準備 |
| `list_google_ads_links` | Google Ads 連携情報 | 広告連携状況の確認 |

### 複数アカウント対応

サービスアカウントに複数の GA4 プロパティの閲覧権限を付与することで、1つの MCP サーバーから全プロパティにアクセスできる。

作業フロー:
1. `get_account_summaries` で全プロパティを一覧取得する
2. 対象プロパティを特定する（名前やプロパティ ID で絞り込み）
3. `run_report` / `run_realtime_report` でデータを取得する

### 典型的なプロンプト例

```
# プロパティ一覧の取得
「アクセスできる GA4 プロパティを全て教えて」

# トラフィック分析
「3x3-portal の過去30日間のページビュー上位10ページを教えて」

# ユーザー行動分析
「Ledgea の過去7日間のイベント別発火回数を教えて」

# リアルタイム確認
「今のアクティブユーザー数を教えて」

# コンバージョン分析
「過去90日間のコンバージョンレートをチャネル別に教えて」

# カスタムイベント確認
「このプロパティのカスタムディメンション一覧を取得して」
```

### analytics-reporter との連携

MCP 経由で取得した GA4 データを analytics-reporter エージェントに渡し、定期レポートや KPI ダッシュボードの作成に活用する:

```
GA4 MCP (データ取得) → analytics-reporter (分析・可視化) → CMO (意思決定)
```

### セットアップ前提条件

- GCP プロジェクトで Google Analytics Admin API / Data API を有効化済み
- サービスアカウント作成済み（Viewer ロール）
- 対象 GA4 プロパティにサービスアカウントのメールアドレスを閲覧者として追加済み
- `~/.claude/.mcp.json` に `analytics-mcp` サーバーが設定済み
- 環境変数 `GOOGLE_APPLICATION_CREDENTIALS`（サービスアカウント JSON キーのパス）を設定済み

## 連携先エージェント

- **CMO**: 計測要件の受領、KPI に基づくイベント設計の方針決定
- **analytics-reporter**: 計測データの分析・レポート作成を引き継ぐ
- **frontend-developer**: dataLayer.push の実装箇所・タイミングの調整
- **seo-specialist**: 構造化データとの整合性確認
- **legal-compliance-checker**: プライバシーポリシー・Cookie 同意の法的要件確認
- **cloudflare → platforms/cloudflare**: サーバーサイド GTM のプロキシ設定
