# スケジュールタスク設計ルール

## 第一原則: タスクは壊れる前提で設計する

スケジュールタスクは「次も正常に動く」前提で設計してはならない。
CLI アップデート、認証切れ、ネットワーク障害など、外部依存は常に壊れうる。

## 必須設定

### notifyOnCompletion: true（例外なし）
- タスク作成時に `notifyOnCompletion: true` を必ず設定する
- 失敗を防ぐことはできなくても、失敗に気づくことはできる
- 通知なしのタスクは「動いているかどうか誰も知らないタスク」と同義

### 完了条件 = 本番反映
- タスクの完了条件は「成果物が本番環境に反映されること」
- ブランチへの push は中間成果物であり、完了ではない
- タスク定義には以下の全ステップを含める:
  1. 成果物の生成
  2. git commit
  3. git push
  4. PR 作成（`gh pr create`）
  5. PR マージ（`gh pr merge`）
  6. デプロイ確認（Vercel 自動デプロイの場合はマージで完了）

## ブランチ戦略

### 毎回 main から新規ブランチを作成する
- 前回のブランチが残っている前提で設計しない
- `feature/request` 等の固定名ブランチを使い回さない（stale state の原因）

```bash
# Good: 毎回クリーンに作成
git checkout main && git pull origin main
git checkout -b feature/request
# ... 作業 ...
git push -u origin feature/request
gh pr create --title "..." --body "..."
gh pr merge --merge

# Bad: 既存ブランチがある前提
git checkout feature/request  # ブランチが存在しなければ失敗
```

## 認証切れへの対応

### CLI アップデートでセッションは切れる
- Claude Code のアップデートは頻繁に行われる
- アップデート時にログインセッションが無効化されることがある
- これは「異常」ではなく「日常」

### SKILL.md レベルでは解決不能
- 認証はタスクランナーのインフラ層で処理される
- タスク定義（SKILL.md）に認証リカバリーロジックを書いても意味がない
- 対策は `notifyOnCompletion` による検知と、手動リカバリー

## 発生事例（2026-03-30）

### weekly-article-publisher 未実行
- CLI が 2.1.85 → 2.1.87 にアップデートされ、ログインセッション無効化
- タスクは起動されたが `Not logged in · Please run /login` で即座に失敗
- `notifyOnCompletion` 未設定のため、ユーザーが自分で気づくまで発覚しなかった
- 手動実行後も `feature/request` への push で終了し、PR マージを忘れて本番未反映

### 教訓
- 失敗の検知（通知）がなければ、タスクは「動いているフリ」をし続ける
- push で満足せず、本番反映まで追うのがタスクの責務
