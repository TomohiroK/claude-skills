# Platforms（外部サービスエキスパート）

外部プラットフォーム固有の要件・制約・ベストプラクティスに特化したエージェント群。
汎用的な開発知識ではなく、ツール固有の正確で安全な操作を担当する。

## 共通ルール

- `.claude/rules/platform-experts.md` に定義
- **稼働開始時**: WebSearch で最新情報（Breaking Changes、セキュリティ、料金変更）を収集してから作業開始
- **作業完了時**: 振り返りプロトコルを実施し、学習事項を CTO に提出

## エージェント一覧（10名）

| エージェント | 対象プラットフォーム | 管轄 CxO | 主な責務 |
|-------------|-------------------|----------|---------|
| [google-analytics-expert](google-analytics-expert.md) | GA4 / GTM | CMO | イベント設計、計測実装、デバッグ |
| [google-drive-expert](google-drive-expert.md) | Google Drive / Workspace API | COO | ファイル管理、権限設計、自動化 |
| [claude-api-expert](claude-api-expert.md) | Claude API / Anthropic SDK | CTO | API統合、Tool Use、コスト最適化 |
| [openai-api-expert](openai-api-expert.md) | OpenAI API / SDK | CTO | Chat Completions、Embeddings、画像/音声 |
| [google-tts-expert](google-tts-expert.md) | Google Cloud TTS / STT | CTO | 音声合成・認識、SSML、多言語対応 |
| [cloudflare-expert](cloudflare-expert.md) | Cloudflare | CTO | DNS、CDN、Workers、WAF、Zero Trust |
| [vercel-expert](vercel-expert.md) | Vercel | CTO | デプロイ、Edge/Serverless Functions |
| [neon-expert](neon-expert.md) | Neon (Serverless PostgreSQL) | CTO | ブランチング、接続、マイグレーション |
| [aws-expert](aws-expert.md) | Amazon Web Services | CTO | EC2、Lambda、S3、RDS、IAM、VPC |
| [gcp-expert](gcp-expert.md) | Google Cloud Platform / Firebase | CTO | Cloud Run、BigQuery、Firestore、IAM |

## 連携パターン

### フルスタックデプロイ
`vercel-expert` + `cloudflare-expert` + `neon-expert`

### AI 機能構築
`claude-api-expert` + `openai-api-expert` + `ai-engineer`

### クラウドインフラ
`aws-expert` + `gcp-expert` + `devops-automator`

### 計測・分析基盤
`google-analytics-expert` + `gcp-expert`(BigQuery) + `analytics-reporter`

## 他カテゴリとの関係

- **engineering/**: 汎用的な開発スキル → platforms/ はプラットフォーム固有の操作知識
- **orchestra/**: CxO が方針決定 → platforms/ が具体的なプラットフォーム操作を実行
- **testing/**: テスト戦略 → platforms/ がプラットフォーム固有のテスト手法を提供
