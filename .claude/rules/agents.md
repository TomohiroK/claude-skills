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

## CxO召集パターン
- キャンペーン/プロモーション: CLO, CTO, CVO, CMO, COO, 記録係（金銭が絡む施策はCLO必須）
- ブランディング: CVO, CMO, CEO, 記録係
- 技術設計: CTO, COO, 記録係
- セキュリティ監査: CTO, CLO, 記録係（書面許可必須。ルール: `.claude/rules/security-audit.md`）
- コードレビュー: code-reviewer, （必要に応じて）tech-standards-architect, CTO
