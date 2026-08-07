# Dynamic Workflows ライブラリ

CxOボード組織（69名）の協働パターンを **Dynamic Workflow** として再現するためのライブラリと設計仕様。

## このディレクトリの責務

- `*.js` … 保存済みワークフロースクリプト。`/<name>` で起動する（リポジトリ共有）
- `README.md`（本ファイル）… ライブラリ全体のアーキテクチャと各ワークフローの**設計仕様（spec）**

このREADMEは「Claude にワークフロースクリプトを書かせるときの設計図」であり、エージェント定義（`.claude/agents/`）と並ぶ唯一の真実（single source of truth）として機能する。

---

## 前提

| 項目 | 内容 |
|------|------|
| Dynamic Workflows リリース | 2026-05-28（research preview） |
| 必要バージョン | Claude Code v2.1.154 以上 |
| 当環境の状態 | v2.1.159 / 無効化設定なし（稼働可能） |
| 公式ドキュメント | https://code.claude.com/docs/en/workflows |

### Dynamic Workflow とは
Claude がタスクに応じて書く **JSオーケストレーションスクリプト**。ランタイムが背景で実行し、最大1000体（同時16体）のサブエージェントを並列に回す。計画・ループ・分岐・中間結果は**スクリプトが保持**し、Claudeのcontextには最終結果だけが返る。

---

## なぜ「ワークフロー・ライブラリ」なのか（設計判断）

### モノリスにしない
全69名を1本の巨大ワークフローにするのは反パターン。理由は2つ:
1. **組織規律違反のリスク** — 巨大化すると不可逆操作の混入を制御できない
2. **workflowの想定用途外** — workflowは「1タスク1オーケストレーション」が単位

→ **召集パターン（`orchestrator.md` の召集パターン表）ごとに独立したワークフロー**を作り、段階的に拡張する。

### 多段委譲の制約を回避する
公式制約: **「サブエージェントは別のサブエージェントを起動できない（Subagents cannot spawn other subagents）」**。

現組織の「orchestrator → CxO → worker」という多段委譲は、サブエージェント単体では不可能。
**ワークフローはこれを解く** — スクリプト自身がオーケストレーターとなり、1階層のfan-outを**フェーズで多段化**する。これがライブラリ方式が機能する技術的根拠。

---

## アーキテクチャ：共通5フェーズパターン

全ワークフローは原則として以下の構造を取る（不要フェーズは省略可）。

```
Phase 1  Prepare      直列1体（secretary）   入力整理・論点生成（args を受ける）
Phase 2  Fan-out      並列N体                各エージェントが独立に分析（1階層）
Phase 3  Cross-check  並列（任意）            相互の敵対的レビュー（公式の品質パターン）
Phase 4  Synthesize   直列1体（ceo 等）       統合・優先順位付け
Phase 5  Record       直列1体（secretary）    logs/ へ保存＋活動ログ追記
```

- **Fan-out** がワークフローの主戦場。独立・並列・読み取り中心のタスクで最大効果
- **Cross-check** は公式が推奨する品質パターン（"independent agents adversarially review each other's findings"）。重要な意思決定で有効
- **Synthesize / Record** で人間に返す単一の成果物を生成する

---

## 共通ガードレール（組織規律のコード化）

全ワークフローに必ず焼き込む。スコープ逸脱はワークフロー設計の不合格条件。

| 規律 | 参照ルール | ワークフローでの強制方法 |
|------|-----------|------------------------|
| スコープ限定 | `local-first-then-push.md` / `deploy-target.md` | **「分析・生成まで」**に限定。`git push`/`merge`/`deploy`/`commit`/`rm`/`DROP` 等の不可逆操作を**一切含めない** |
| DRY | `agents.md` | 各エージェントは `.claude/agents/*.md` を参照。システムプロンプトを script に複製しない |
| 活動追跡 | `agent-activity-tracking.md` | Record フェーズで `logs/agent-activity/YYYY-MM.md` に召集記録を追記 |
| ルール反映は人間確認 | `conversation-close.md` | 学びの**「提案」まで**。`.claude/rules/` の実書き換えはユーザーに返す |
| 自動承認リスクの封じ込め | — | workflowのサブエージェントは acceptEdits + allowlist継承。書き込み先を `.claude/orchestra/logs/` 等の**成果物ディレクトリに限定**し、コード本体・本番に触れさせない |

### なぜスコープ限定が必須か
Dynamic Workflow は ①実行中ユーザー入力不可 ②ファイル編集が自動承認（acceptEdits）。
よって CEO決裁・ユーザー確認・デプロイ先確認といった**人間ゲートが効かない**。`local-first-then-push.md` で `git push`/`merge` は Claude に永久禁止されているため、ワークフローはこれらを**構造的に含めてはならない**。実反映（commit/push/deploy）は常に人間に返す。

---

## エージェント参照原則（DRY）

- ワークフローが起動するエージェントは、`.claude/agents/{category}/*.md` の定義を唯一の真実とする
- spec（本README）には「どのエージェントを・どのフェーズで・何の入力で呼ぶか」だけを書く
- エージェントのシステムプロンプト本文を spec やスクリプトに複製しない（定義変更時の不整合を防ぐ）

---

## ワークフロー一覧（ロードマップ）

| コマンド | 再現する召集パターン | Fan-out対象 | 状態 |
|---------|--------------------|------------|------|
| `/retrospective` | 全体振り返り | 8 CxO（CEO除く） | **パイロット（設計済・未実装）** |
| `/board-meeting` | ボードミーティング | アジェンダ依存のCxO | 未着手 |
| `/org-audit` | 四半期組織レビュー | 全69エージェント定義 | 未着手 |
| `/deep-review` | コード敵対レビュー | code-reviewer / qa-engineer / security-officer | 未着手 |
| `/campaign-lp-review` | キャンペーンLPレビュー | CLO / CMO / CVO / security-officer | 未着手 |

横展開は**パイロットで型を実証してから**順次行う（`no-shortcuts.md`）。

---

## パイロット仕様：`/retrospective`

`orchestrator.md` の振り返りフロー（記録整理 → 各CxO報告 → KPT → CEO優先順位 → 記録）を、逐次から**並列フェーズ**に再構成する。

### args（起動時入力）
```js
args = {
  period: "2026-Q2",         // 対象期間
  projects: ["..."],          // 対象プロジェクト（任意。空なら全体）
  includeCrossCheck: false    // Phase 3 を実行するか
}
```
`args` 省略時は直近期間・全プロジェクト・Cross-checkなしをデフォルトとする。

### フェーズ定義

| Phase | エージェント | 並列 | 入力 | 出力 |
|-------|------------|------|------|------|
| 1 Prepare | `orchestra/secretary` ×1 | 直列 | `logs/meetings` `logs/decisions` `logs/learnings` `logs/agent-activity` | 各CxO向けの論点リスト |
| 2 Fan-out | `cto` `coo` `cfo` `cmo` `clo` `cso` `cvo` `security-officer` ×8 | **並列** | Phase1の論点 | `[{ cxo, keep[], problem[], try[] }]` |
| 3 Cross-check（任意） | Phase2の8体 | 並列 | 他CxOのTry | Try相互の重複・矛盾・実現性指摘 |
| 4 Synthesize | `orchestra/ceo` ×1 | 直列 | 全KPT（+ Cross-check結果） | 優先順位付きアクションリスト |
| 5 Record | `orchestra/secretary` ×1 | 直列 | Phase4の成果 | 下記の保存物 |

### Fan-out 対象の根拠
- CEO は議長・最終決裁役のため Fan-out から除外し Phase 4（Synthesize）に置く
- security-officer は CTO から独立した立場のため Fan-out に含める
- worker層（69名）は含めない。振り返りはCxOボードの活動であるため

### 出力先
- `.claude/orchestra/logs/retrospectives/YYYY-MM-DD.md` … 振り返り結果（KPT統合＋アクション）
- `.claude/orchestra/logs/agent-activity/YYYY-MM.md` … 召集記録を追記
- **ルール化すべき学び**は「提案リスト」として出力するに留め、`.claude/rules/` の改訂はユーザーに返す（`conversation-close.md`）

### 規模・リスク
- 実質 約11体（8並列 + prepare + synthesize + record）。1000上限・16並列に対し極小 → トークン実証に最適
- 読み取り + `logs/` 書き込みのみ。コード本体・本番に非接触。最も安全なパイロット

---

## ビルド経路（spec → 実装 → 検証 → 保存）

workflowスクリプトの**正確なspawn APIは研究プレビューで未公開**のため、`.js` を手書きしない。以下の経路を取る。

1. **spec を本READMEに確定**（このファイル）
2. **Claude にスクリプトを書かせる** — 起動例:
   ```
   Run a workflow for our Q2 retrospective, following the spec in .claude/workflows/README.md
   ```
3. **生成スクリプトを検証** — 承認前に `View raw script`（`Ctrl+G`）で確認:
   - CxOを名前で呼べているか / `.md` 参照になっているか（spawn機構の実証）
   - ガードレール（不可逆操作の不在・出力先限定）を満たすか
   - まず小さいスコープで1回回し、トークン消費を把握
4. **保存** — `/workflows` で run を選び `s` キー → `.claude/workflows/retrospective.js`
5. **横展開** — 実証できた型を `/board-meeting` → `/org-audit` → `/deep-review` … へ

---

## 関連ルール

- `.claude/rules/agents.md` — エージェント管理・召集パターン
- `.claude/rules/agent-activity-tracking.md` — 活動追跡
- `.claude/rules/local-first-then-push.md` — push/merge は Claude 永久禁止
- `.claude/rules/deploy-target.md` — デプロイ先確認
- `.claude/rules/conversation-close.md` — 学びはルールに落とす（人間確認）
- `.claude/rules/no-shortcuts.md` — ワークフローを端折らない
- `.claude/agents/orchestra/orchestrator.md` — 召集パターンの原典
