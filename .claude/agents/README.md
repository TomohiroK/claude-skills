# エージェント定義

Claude Code サブエージェント組織（69名）の定義ファイル。

## カテゴリ一覧

| カテゴリ | 人数 | 責務 |
|---------|------|------|
| [engineering/](engineering/) | 10名 | ソフトウェア開発（フロント/バック/AI/DevOps等） |
| [platforms/](platforms/) | 10名 | 外部サービスエキスパート（AWS/GCP/Vercel/Cloudflare等） |
| [product/](product/) | 4名 | プロダクト企画（PM、リサーチ、フィードバック、スプリント） |
| [marketing/](marketing/) | 12名 | マーケティング（SNS、SEO、コンテンツ、グロース等） |
| [design/](design/) | 6名 | デザイン（UI/UX、ビジュアル、ブランド、モーション） |
| [project-management/](project-management/) | 3名 | プロジェクト管理（リリース、実験、スタジオ運営） |
| [studio-operations/](studio-operations/) | 6名 | スタジオ運営（インフラ、財務、法務、サポート、分析、人事） |
| [testing/](testing/) | 5名 | テスト・品質保証（API、パフォーマンス、ワークフロー） |
| [orchestra/](orchestra/) | 11名 | CxOボード（CEO/CTO/CFO/CMO等 + orchestrator + secretary） |

## 独立遊軍

| エージェント | 管轄 | 責務 |
|-------------|------|------|
| [service-account-manager.md](service-account-manager.md) | CTO直轄 | クレデンシャル暗号化管理（`~/.claude-vault/`） |

独立遊軍はカテゴリに属さず、専用の作業ディレクトリを持つ。他エージェントからの直接アクセスは禁止。

## 共通仕様

- YAML frontmatter 必須: `name`, `description`, `model`, `tools`, `permissionMode: acceptEdits`
- orchestra CxO は `model: opus`、その他は `model: sonnet`
- `tools` フィールドでツールアクセスを明示的に制限
- 各エージェントが持つセクション:
  - 作業開始プロトコル
  - コーディングルール（該当する場合）
  - セキュリティ
  - テスト
  - 完了条件
  - 他エージェントとの連携

## 関連ルール

- `.claude/rules/agents.md` — エージェント管理ルール
- `.claude/rules/platform-experts.md` — プラットフォームエキスパート共通ルール
- `.claude/rules/credential-management.md` — クレデンシャル管理ルール
- `.claude/rules/development-workflow.md` — 開発ワークフロー（フェーズゲート）
