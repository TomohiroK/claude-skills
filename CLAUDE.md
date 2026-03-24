# Claude Skills - プロジェクトメモ

## このリポジトリについて
Claude Code のサブエージェント組織（55名）を管理するリポジトリ。
CxOボード（orchestra）による振り返り・改善サイクルを持つ。

## ディレクトリ構成
```
.claude/agents/
├── engineering/       # 8名: frontend-developer, backend-architect, mobile-app-builder, ai-engineer, devops-automator, rapid-prototyper, qa-engineer, data-engineer
├── product/           # 4名: product-manager, trend-researcher, feedback-synthesizer, sprint-prioritizer
├── marketing/         # 12名: growth-hacker, content-creator, copywriter, trend-analyst, brand-ambassador, community-manager, seo-specialist, tiktok-strategist, instagram-curator, twitter-engager, reddit-community-builder, app-store-optimizer
├── design/            # 6名: ui-designer, ux-researcher, brand-guardian, visual-storyteller, motion-designer, whimsy-injector
├── project-management/ # 3名: studio-producer, project-shipper, experiment-tracker
├── studio-operations/ # 6名: support-responder, infrastructure-maintainer, analytics-reporter, finance-tracker, legal-compliance-checker, hr-coordinator
├── testing/           # 5名: api-tester, performance-benchmarker, test-results-analyzer, tool-evaluator, workflow-optimizer
└── orchestra/         # 11名: orchestrator, ceo, cto, coo, cfo, cmo, clo, cso, cvo, chro, secretary

.claude/orchestra/logs/  # 議事録・学習ログ
```

## エージェント共通仕様
- YAML frontmatter: name, description, model (sonnet/opus), permissionMode: acceptEdits
- orchestra CxO は model: opus、それ以外は model: sonnet
- 各エージェントは 作業開始プロトコル / アウトプットテンプレート / 完了条件 / 他エージェント連携 を持つ

---

# fixedassets プロジェクト情報

## 概要
ASEAN 11カ国 + 日本対応の固定資産管理 SaaS（クライアントサイド完結型）

## 現在の商品名（仮）
FixedAssets → **Ledgea（レジア）** に変更予定（ボード決定済み・未実装）

## ブランド戦略（ボード決定事項）
- 商品名: Ledgea（Ledger + Asia）
- タグライン EN: "The ledger built for Asia's complexity."
- タグライン JA: 「アジアの複雑さを、ひとつの台帳に。」
- ロゴ方向性: 3層カードモチーフ + 碇の安定感融合、Teal(#0D9488)基調
- ロゴフォント: Plus Jakarta Sans
- ターゲット: 日系企業のASEAN現地法人（経理・管理部門）
- 参入順序: 日本 → タイ → インドネシア → ベトナム

## 技術スタック
- **言語**: Rust + WebAssembly（Leptos 0.7 CSR）
- **ビルド**: Trunk
- **スタイル**: Tailwind CSS（mobile-first）
- **フォント**: Space Grotesk（見出し）、Inter（本文）
- **ストレージ**: IndexedDB v2（assets, photos）、localStorage（設定系）
- **精度**: rust_decimal（金融計算用高精度演算）
- **デプロイ**: Vercel（静的 SPA + Edge Middleware）
- **セキュリティ**: CSP, X-Frame-Options DENY, HSTS, SHA-256+salt認証

## 対応国（11カ国 + 日本）
| 国 | 通貨 | 減価償却方式 |
|----|------|-------------|
| Japan | JPY | 200%定率法 + 保証率切替 |
| Singapore | SGD | Capital Allowance IA+AA |
| Malaysia | MYR | Capital Allowance IA+AA |
| Thailand | THB | 二重定率法 |
| Indonesia | IDR | グループベース率（PMK-72/2023） |
| Philippines | PHP | 二重定率法 |
| Vietnam | VND | 150%/200%定率法 |
| Myanmar | MMK | 定額法のみ |
| Cambodia | KHR | 二重定率法 |
| Laos | LAK | 二重定率法 |
| Brunei | BND | 定額法 |

## 主要ソースファイル（/Users/tomohirok/Documents/Github/fixedassets/）
- `src/app.rs` - ルートコンポーネント
- `src/router.rs` - 20+ルート（国別SEOページ含む）
- `src/models/depreciation.rs` - 11カ国の減価償却計算（859行）
- `src/models/company.rs` - AseanCountry enum, Currency enum, CompanySetup
- `src/models/asset.rs` - Asset構造体（IFRS二重帳簿対応）
- `src/pages/asset_detail.rs` (34KB), `depreciation.rs` (48KB), `settings.rs` (30KB)
- `src/components/asset_form.rs` (36KB), `wizard_form.rs` (39KB)
- `src/stores/asset_store.rs` (30KB) - IndexedDB CRUD, CSV/JSON import/export
- `locales/en.json`, `locales/ja.json` - i18n（200+キー）
- `public/lp.html` - ランディングページ
- `middleware.js` - Vercel Edge Middleware（geo-redirect等）

## プラン体系
- Free: 5資産、1部門
- Paid: 無制限

## データ管理
- DATA_VERSION による自動リセット機構
- CSV 19列 / JSON インポート・エクスポート（5MB/10K件制限）
- デモアカウント: demo@example.com, admin@example.com, tanaka@example.com

## 注意: fixedassets-bn は別プロジェクト（対象外）
