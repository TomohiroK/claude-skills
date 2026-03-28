# Platforms（外部サービスエキスパート）

外部プラットフォーム固有の要件・制約・ベストプラクティスに特化した統合エージェント。
汎用的な開発知識ではなく、ツール固有の正確で安全な操作を担当する。

## アーキテクチャ

```
.claude/agents/platforms/
└── platform-expert.md          # 統合エージェント（共通プロトコル）

.claude/skills/platforms/
├── vercel.md                   # Vercel 固有知識
├── cloudflare.md               # Cloudflare 固有知識
├── neon.md                     # Neon 固有知識
├── aws.md                      # AWS 固有知識
├── gcp.md                      # GCP 固有知識
├── firebase.md                 # Firebase 固有知識
├── claude-api.md               # Claude API 固有知識
├── openai-api.md               # OpenAI API 固有知識
├── google-analytics.md         # GA4/GTM 固有知識
├── google-drive.md             # Google Drive 固有知識
└── google-tts.md               # Google TTS/STT 固有知識
```

**エージェント1名 + スキル11個** の構成。
`platform-expert` エージェントがタスクに応じて適切なスキルを参照して作業する。

## 共通ルール

- `.claude/rules/platform-experts.md` に定義
- **稼働開始時**: WebSearch で最新情報（Breaking Changes、セキュリティ、料金変更）を収集してから作業開始
- **作業完了時**: 振り返りプロトコルを実施し、学習事項を管轄 CxO に提出

## スキル一覧（11個）

| スキル | 対象プラットフォーム | 管轄 CxO | 主な責務 |
|--------|-------------------|----------|---------|
| [google-analytics](../../skills/platforms/google-analytics.md) | GA4 / GTM | CMO | イベント設計、計測実装、デバッグ |
| [google-drive](../../skills/platforms/google-drive.md) | Google Drive / Workspace API | COO | ファイル管理、権限設計、自動化 |
| [claude-api](../../skills/platforms/claude-api.md) | Claude API / Anthropic SDK | CTO | API統合、Tool Use、コスト最適化 |
| [openai-api](../../skills/platforms/openai-api.md) | OpenAI API / SDK | CTO | Chat Completions、Embeddings、画像/音声 |
| [google-tts](../../skills/platforms/google-tts.md) | Google Cloud TTS / STT | CTO | 音声合成・認識、SSML、多言語対応 |
| [cloudflare](../../skills/platforms/cloudflare.md) | Cloudflare | CTO | DNS、CDN、Workers、WAF、Zero Trust |
| [vercel](../../skills/platforms/vercel.md) | Vercel | CTO | デプロイ、Edge/Serverless Functions |
| [neon](../../skills/platforms/neon.md) | Neon (Serverless PostgreSQL) | CTO | ブランチング、接続、マイグレーション |
| [aws](../../skills/platforms/aws.md) | Amazon Web Services | CTO | EC2、Lambda、S3、RDS、IAM、VPC |
| [gcp](../../skills/platforms/gcp.md) | Google Cloud Platform | CTO | Cloud Run、BigQuery、Cloud SQL、IAM |
| [firebase](../../skills/platforms/firebase.md) | Firebase | CTO | Authentication、Firestore、Functions、Hosting |

## 連携パターン

### フルスタックデプロイ
`vercel` + `cloudflare` + `neon`

### AI 機能構築
`claude-api` + `openai-api` + `ai-engineer`

### クラウドインフラ
`aws` + `gcp` + `devops-automator`

### Firebase フルスタック
`firebase` + `gcp` + `frontend-developer`

### 計測・分析基盤
`google-analytics` + `gcp`(BigQuery) + `analytics-reporter`

## 他カテゴリとの関係

- **engineering/**: 汎用的な開発スキル → platforms/ はプラットフォーム固有の操作知識
- **orchestra/**: CxO が方針決定 → platforms/ が具体的なプラットフォーム操作を実行
- **testing/**: テスト戦略 → platforms/ がプラットフォーム固有のテスト手法を提供
