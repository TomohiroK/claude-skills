# CxOボードミーティング議事録

- **日付**: 2026-03-27
- **アジェンダ**: 新メンバー（platforms 10名 + service-account-manager）のオンボーディング検証
- **参加者**: CEO, CTO, COO, CMO, CFO, CHRO, Secretary

---

## 1. 各CxOの主要意見サマリ

### CEO（戦略）
- platforms カテゴリの設立は戦略方針に合致。実需（Ledgea: Vercel+Neon、3x3-portal: Vercel+GA4）に基づく拡張
- **全10名同時稼働ではなくフェーズ投入を推奨**。aws-expert / gcp-expert は実需発生まで待機可
- CTOへの管轄集中（18名以上）が最大のボトルネック
- 69名規模での「適切なエージェント発見性」が課題に近づいている

### CTO（技術）
- engineering/ との役割重複は限定的。「プラットフォーム固有知識 vs 汎用開発スキル」の軸で合理的に分離済み
- Vault 設計は堅牢。inbox パターンによる平文露出時間の最小化を評価
- **master.key のバックアップ戦略が未定義**（単一障害点）
- **各エキスパートに公式 changelog URL をハードコード**すべき（WebSearch 依存の回避）
- CI/CD クレデンシャル注入フロー（GitHub Actions Secrets 等）が未定義

### COO（運用）
- inbox フローは堅実だが、ユーザーのCLI操作スキルに依存する摩擦がある
- 連携パターンに**コーディネーター役が未定義**（3名並列稼働時の統括者が不明）
- WebSearch の冗長実行（10名同時稼働時に40回以上）がオーバーヘッド
- **「エキスパート不要な軽微操作」の判断基準**が未定義

### CMO（マーケティング）
- google-analytics-expert と seo-specialist の境界は明確
- **analytics-reporter エージェントが未作成**で「計測→分析→レポート」パイプラインに穴
- **Google Consent Mode v2** への対応記述がない

### CFO（財務）
- 料金変更発見時の**CFOへの報告フローが未設定**（CTO経由のみ）
- 従量課金サービス（Claude API / OpenAI API）の**月額予算ハードリミットが未設定**
- 月次コストレポートの提出義務が未定義
- **コスト異常アラートの閾値**（前月比+20%等）が標準化されていない

### CHRO（組織）
- CTO管理スパン問題（20名前後が直轄）は要注視
- オンボーディングチェックリストが**制度化されていない**（今回は個別対応）
- **監査ログレビューの独立性**が不足（CTO単独 → CTO+CLO複数管轄にすべき）
- 四半期組織レビューの制度化を提案

---

## 2. 合意形成：クロスカッティング論点

### 論点A: CTO管轄の集中問題
- **全CxOが指摘**。CEO/COO/CHROが同一問題を認識
- **合意**: 即座にVP新設は過剰。まずは devops-automator をインフラ系エキスパート（aws/gcp/cloudflare/vercel/neon）の一次窓口とし、CTOへのエスカレーションを二段階にする
- **フォローアップ**: 四半期レビューで管理スパンを再評価。改善しなければVP of Platforms を検討

### 論点B: WebSearch の冗長実行
- **CTO/COO/CFOが指摘**。トークンコストと実効性の両面で懸念
- **合意**: platform-experts.md に「前回確認から24時間以内の再召集ではトレンドチェックをスキップ可」のルールを追加。各エキスパートに公式 changelog URL を追記し、WebSearch ではなく WebFetch で直接取得する方式に段階移行

### 論点C: フェーズ投入
- **CEO提案、CTO/CHROが支持**
- **合意**: 先行稼働（Phase 1）と待機（Phase 2）に分ける
  - Phase 1（即稼働）: vercel-expert, cloudflare-expert, neon-expert, claude-api-expert, google-analytics-expert, service-account-manager — 実需あり
  - Phase 2（実需発生時稼働）: aws-expert, gcp-expert, openai-api-expert, google-tts-expert, google-drive-expert

### 論点D: CFO報告ラインの確立
- **CFO指摘、CEO承認**
- **合意**: 全プラットフォームエキスパートの「他エージェントとの連携」に CFO を追加。料金変更発見時は CTO + CFO に同時報告。finance-tracker による月次コスト集計を義務化

---

## 3. 決定事項

| # | 決定内容 | 担当CxO | 期限 |
|---|---------|---------|------|
| D1 | platform-experts.md に「24時間以内再召集時のトレンドチェックスキップ」ルールを追加 | CTO | 2026-03-28 |
| D2 | 各エキスパートに公式 changelog URL を追記（WebFetch 優先方式に移行） | CTO | 2026-04-03 |
| D3 | devops-automator をインフラ系エキスパート5名の一次窓口に設定（CTO管理スパン緩和） | CHRO+CTO | 2026-04-03 |
| D4 | 全プラットフォームエキスパートの連携先に CFO を追加、料金変更の同時報告ルール化 | CFO | 2026-03-31 |
| D5 | credential-management.md に master.key バックアップ戦略を追記 | CTO | 2026-03-31 |
| D6 | analytics-reporter エージェントの作成（計測パイプライン完成） | CMO | 2026-04-10 |
| D7 | google-analytics-expert に Consent Mode v2 対応を追記 | CMO | 2026-04-10 |
| D8 | 連携パターンにコーディネーター役を明記（platform-experts.md） | COO | 2026-04-03 |
| D9 | CI/CD クレデンシャル注入フローを credential-management.md に追記 | CTO | 2026-04-10 |
| D10 | オンボーディングチェックリストのルール化（.claude/rules/onboarding.md 新設） | CHRO | 2026-04-10 |
| D11 | 監査ログレビュー権限を CTO 単独 → CTO+CLO 複数管轄に変更 | CHRO+CLO | 2026-04-03 |
| D12 | フェーズ投入: Phase 1 (6名即稼働) / Phase 2 (5名待機) を CLAUDE.md に反映 | CEO | 2026-03-28 |
| D13 | 四半期組織レビュー（69名体制の棚卸し）の制度化 | CHRO | 2026-04-10 |

---

## 4. 未解決事項（次回持ち越し）

- 従量課金サービスの月額予算ハードリミットの具体値設定（CFO + 各エキスパートで協議）
- platforms 間の統合テストパターンの設計（CTO + qa-engineer で検討）
- Server-Side GTM + Cloudflare Workers の連携手順具体化（CMO + CTO で検討）

---

## 5. 総合評価

**組織として回ると判断する。ただし以下の条件付き:**

1. D1-D5（即時対応事項）が完了するまで Phase 1 エージェントの本番稼働を開始しない
2. フェーズ投入（D12）を採用し、全10名同時稼働は行わない
3. 四半期組織レビュー（D13）で管理スパン・コスト・連携品質を継続監視する
4. service-account-manager の Vault 設計はセキュリティ面で十分と判断。master.key バックアップ（D5）の完了後に本番稼働可

**次回ボードミーティング**: 2026-04-10（D6-D10, D13 の進捗確認）
