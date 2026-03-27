# エージェント管理ルール

## エージェント作成・編集
- エージェントファイルは `.claude/agents/{category}/` に配置
- YAML frontmatter 必須: name, description, model, tools, permissionMode: acceptEdits
- orchestra CxO は model: opus、その他は model: sonnet
- 各エージェントには 作業開始プロトコル / アウトプットテンプレート / 完了条件 / 他エージェント連携 を記載
- tools フィールドで各エージェントが使えるツールを明示的に制限する

## ボードミーティング
- 議事録は `.claude/orchestra/logs/meetings/` に日付ファイルで保存
- 決定事項テーブルには「決定内容・担当CxO・期限」の3列を必須とする
- 決定事項は `.claude/orchestra/logs/decisions/` に記録
- 学習事項は `.claude/orchestra/logs/learnings/` に記録
- 振り返りは `.claude/orchestra/logs/retrospectives/` に記録
- 振り返りから得たルールは .claude/rules/ に反映する

## プラットフォームエキスパート
- `.claude/agents/platforms/` に配置（10名）
- 外部サービス固有の要件・制約・ベストプラクティスに特化
- 稼働開始時に WebSearch で最新情報（Breaking Changes、セキュリティ、料金変更）を収集する
- 作業完了時に振り返りプロトコルを実施し、学習事項をCTOに提出する
- 共通ルール: `.claude/rules/platform-experts.md`

| エキスパート | 対象 | 管轄 CxO |
|-------------|------|----------|
| google-analytics-expert | GA4 / GTM | CMO |
| google-drive-expert | Google Drive / Workspace API | COO |
| claude-api-expert | Claude API / Anthropic SDK | CTO |
| openai-api-expert | OpenAI API / SDK | CTO |
| google-tts-expert | Google Cloud TTS / STT | CTO |
| cloudflare-expert | Cloudflare (DNS/CDN/Workers/WAF) | CTO |
| vercel-expert | Vercel (Deploy/Edge/Serverless) | CTO |
| neon-expert | Neon (Serverless PostgreSQL) | CTO |
| aws-expert | Amazon Web Services | CTO |
| gcp-expert | Google Cloud Platform / Firebase | CTO |

## CxO召集パターン
- キャンペーン/プロモーション: CLO, CTO, CVO, CMO, COO, 記録係（金銭が絡む施策はCLO必須）
- ブランディング: CVO, CMO, CEO, 記録係
- 技術設計: CTO, COO, 記録係
- セキュリティ監査: CTO, CLO, 記録係（書面許可必須。ルール: `.claude/rules/security-audit.md`）
- コードレビュー: code-reviewer, （必要に応じて）tech-standards-architect, CTO
- プラットフォーム操作: 該当エキスパート + 管轄CxO（ルール: `.claude/rules/platform-experts.md`）
