# Claude Skills

## このリポジトリについて
Claude Code のサブエージェント組織（70名）を管理するリポジトリ。
CxOボード（orchestra）による振り返り・改善サイクルを持つ。

## ディレクトリ構成
```
.claude/
├── agents/                    # エージェント定義（70名）
│   ├── engineering/           # 10名
│   ├── platforms/             # 11名（外部サービスエキスパート）
│   ├── service-account-manager.md  # 独立遊軍（CTO直轄）
│   ├── product/               # 4名
│   ├── marketing/             # 12名
│   ├── design/                # 6名
│   ├── project-management/    # 3名
│   ├── studio-operations/     # 6名
│   ├── testing/               # 5名
│   └── orchestra/             # 11名（CxOボード）
├── rules/                     # テーマ別ルールファイル
├── commands/                  # カスタムスラッシュコマンド
├── skills/                    # 自動トリガーワークフロー
├── orchestra/logs/            # 議事録・学習ログ
├── settings.json              # チーム権限設定
└── launch.json                # プレビューサーバー設定

brand/                         # ブランド素材（Ledgea）
```

## エージェント共通仕様
- YAML frontmatter: name, description, model, tools, permissionMode: acceptEdits
- orchestra CxO は model: opus、それ以外は model: sonnet
- tools フィールドでエージェントのツールアクセスを制限する
- 各エージェントは 作業開始プロトコル / アウトプットテンプレート / 完了条件 / 他エージェント連携 を持つ

## カスタムコマンド
- `/project:retrospective [name]` — 振り返りの実施
- `/project:board-meeting [agenda]` — ボードミーティング開催
- `/project:new-project [name]` — 新規プロジェクト開始

## platforms フェーズ投入（2026-03-27 ボード決定）
- **Phase 1（即稼働）**: vercel-expert, cloudflare-expert, neon-expert, claude-api-expert, google-analytics-expert, service-account-manager
- **Phase 2（実需発生時稼働）**: aws-expert, gcp-expert, firebase-expert, openai-api-expert, google-tts-expert, google-drive-expert

## クレデンシャル管理
- Vault: ~/.claude-vault/（service-account-manager 専用、他エージェントアクセス禁止）
- クレデンシャルは Claude 会話上で受け渡さない。inbox/ フォルダ経由で暗号化する
- 詳細: `.claude/rules/credential-management.md`

## 作業ディレクトリ
- 全作業: /Users/tomohirok/Documents/Github/claude-works/{project}/
- ブランド素材: /Users/tomohirok/Documents/Github/claude-skills/brand/
