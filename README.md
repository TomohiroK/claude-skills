# Claude Skills - AI Agent Organization

Claude Code のサブエージェントとして動作する、57名のAIエージェント組織。
各エージェントは専門分野を持ち、CxOボードによる統括のもとで連携して業務を遂行する。
リサーチ駆動・品質ゲート・パフォーマンス責任を柱とした、プロダクション品質のエージェントアーキテクチャ。

---

## 組織図

```
                            ┌─────────────┐
                            │ Orchestrator │
                            │  (指揮者)     │
                            └──────┬──────┘
                                   │
        ┌──────────┬───────┬───────┼───────┬───────┬───────┬───────┬────────┐
        │          │       │       │       │       │       │       │        │
     ┌──┴──┐  ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴──┐ ┌──┴───┐
     │ CEO │  │ CTO │ │ COO │ │ CFO │ │ CMO │ │ CLO │ │ CSO │ │ CVO │ │ CHRO │
     └──┬──┘  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬───┘
        │        │       │       │       │       │       │       │       │
     Secretary   │       │       │       │       │       │       │       │
     (記録係)     │       │       │       │       │       │       │       │
                 │       │       │       │       │       │       │       │
```

---

## CxO ボードメンバー（Orchestra）

経営判断・戦略策定を行うボードメンバー。Claude Architect レベルの知識を持ち、振り返りを通じて組織全体を進化させる。

| 役職 | ファイル | 役割 |
|------|---------|------|
| **Orchestrator** | `orchestra/orchestrator.md` | 全体指揮・CxO召集・ボードミーティング運営 |
| **CEO** | `orchestra/ceo.md` | ビジョン策定・最終意思決定・マルチエージェントパイプライン設計・品質ゲート管理 |
| **CTO** | `orchestra/cto.md` | 技術戦略・エージェントアーキテクチャ設計・Claude Code環境構成・コンテキスト管理最適化 |
| **COO** | `orchestra/coo.md` | オペレーション最適化・ワークフロー品質監視・エージェント実行効率・エラー伝播管理 |
| **CFO** | `orchestra/cfo.md` | 財務戦略・予算管理・コスト最適化・投資判断 |
| **CMO** | `orchestra/cmo.md` | データ駆動リサーチパイプライン・コンテンツ制作パイプライン・Weapons Check品質管理 |
| **CLO** | `orchestra/clo.md` | 法的リスク管理・コンプライアンス・契約レビュー |
| **CSO** | `orchestra/cso.md` | 中長期戦略・事業開発・新規事業探索 |
| **CVO** | `orchestra/cvo.md` | ビジュアル品質ゲート・動画/写真ディレクション・ビュー数/CTR/CVR責任 |
| **CHRO** | `orchestra/chro.md` | 組織設計・人材配置・オンボーディング・評価設計 |
| **Secretary** | `orchestra/secretary.md` | 議事録・決定事項追跡・学習ログ管理 |

---

## Engineering（10名） - CTO 管轄

技術的な設計・実装・品質管理を担当するチーム。tech-standards-architect がルールを策定し、code-reviewer がそのルールに基づいてレビューを行うフィードバックループを持つ。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| **Tech Standards Architect** | `engineering/tech-standards-architect.md` | **セキュリティ・トレンド調査→技術選定基準策定→コーディングルール制定** |
| **Code Reviewer** | `engineering/code-reviewer.md` | **策定ルールに基づくコードレビュー→繰り返し指摘のルール化フィードバック** |
| Frontend Developer | `engineering/frontend-developer.md` | React/Vue/Next.js等でのUI構築・コンポーネント設計 |
| Backend Architect | `engineering/backend-architect.md` | API設計・DB設計・マイクロサービス構成 |
| Mobile App Builder | `engineering/mobile-app-builder.md` | React Native/Flutter/Swift/KotlinでのiOS/Android開発 |
| AI Engineer | `engineering/ai-engineer.md` | LLM統合・プロンプトエンジニアリング・RAG構築 |
| DevOps Automator | `engineering/devops-automator.md` | CI/CD構築・Docker・IaC・Kubernetes |
| Rapid Prototyper | `engineering/rapid-prototyper.md` | 高速プロトタイピング・PoC作成・技術検証 |
| QA Engineer | `engineering/qa-engineer.md` | 品質保証戦略・テスト計画・バグトリアージ |
| Data Engineer | `engineering/data-engineer.md` | データパイプライン・ETL/ELT・データ品質管理 |

### 開発ワークフロー（フェーズゲート）

コード変更を伴う開発は以下のフェーズを必ず経る（詳細: `.claude/rules/development-workflow.md`）。

```
要件定義 → 技術選定 → 設計 → 技術選定見直し → 設計完了 → コーディング → コードレビュー → テスト
                ↑                                                        │
                └────── ルール改訂フィードバック ──────────────────────────┘
```

- **tech-standards-architect** が Phase 2（技術選定）と Phase 4（見直し）を担当
- **code-reviewer** が Phase 7（レビュー）を担当。繰り返し指摘 → ルール追加提案
- Phase 7（コードレビュー）と Phase 8（テスト）は省略不可

---

## Product（4名） - CSO 管轄

プロダクトの方向性と優先度を決定するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| Product Manager | `product/product-manager.md` | PRD作成・ユーザーストーリー定義・ロードマップ管理 |
| Trend Researcher | `product/trend-researcher.md` | 市場・技術トレンド調査・競合分析 |
| Feedback Synthesizer | `product/feedback-synthesizer.md` | ユーザーフィードバック分析・傾向分析・優先度付け |
| Sprint Prioritizer | `product/sprint-prioritizer.md` | スプリント計画・RICE/MoSCoWスコアリング・OKR紐付け |

---

## Marketing（12名） - CMO 管轄

マーケティング戦略の立案と実行を担当するチーム。リサーチ駆動・品質ゲート（Weapons Check）・パフォーマンス責任を全員が共有。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| Growth Hacker | `marketing/growth-hacker.md` | AARRR分析・グロース実験・ファネル最適化 |
| Content Creator | `marketing/content-creator.md` | ブログ・ニュースレター・ホワイトペーパー執筆・パイプライン制作 |
| Copywriter | `marketing/copywriter.md` | キャッチコピー・広告文・LPコピー・シナリオ・Weapons Check |
| Trend Analyst | `marketing/trend-analyst.md` | バズ分析・トレンド再現・季節イベント予測 |
| Brand Ambassador | `marketing/brand-ambassador.md` | ブランドの顔・ストーリーテリング・インフルエンサー連携 |
| Community Manager | `marketing/community-manager.md` | Discord/Slack運営・イベント企画・ユーザー交流促進 |
| SEO Specialist | `marketing/seo-specialist.md` | テクニカルSEO・キーワード戦略・サイト構造最適化 |
| TikTok Strategist | `marketing/tiktok-strategist.md` | ショート動画企画・フック設計・サムネイルA/Bテスト |
| Instagram Curator | `marketing/instagram-curator.md` | フィード設計・ストーリーズ/リール戦略・Weapons Check |
| Twitter Engager | `marketing/twitter-engager.md` | ツイート・スレッド企画・引用ツイート比率分析 |
| Reddit Community Builder | `marketing/reddit-community-builder.md` | サブレディット参入・ダウンボート分析・痛点の構造化提供 |
| App Store Optimizer | `marketing/app-store-optimizer.md` | ASO・キーワード最適化・ストアページ改善 |

---

## Design（7名） - CVO 管轄

デザイン品質とユーザー体験を統括するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| UI Designer | `design/ui-designer.md` | 画面設計・デザインシステム構築・インタラクション設計 |
| UX Researcher | `design/ux-researcher.md` | ユーザビリティ調査・ペルソナ作成・ジャーニーマップ |
| Brand Guardian | `design/brand-guardian.md` | ブランドガイドライン策定・整合性チェック・違反検出 |
| Visual Storyteller | `design/visual-storyteller.md` | データビジュアライゼーション・プレゼンテーション設計 |
| Key Visual Producer | `design/key-visual-producer.md` | ヒーロー画像・人物写真ディレクション・動画素材の企画・選定 |
| Motion Designer | `design/motion-designer.md` | 動画・アニメーション・モーショングラフィックス設計 |
| Whimsy Injector | `design/whimsy-injector.md` | マイクロインタラクション・イースターエッグ・遊び心 |

---

## Project Management（3名） - COO 管轄

プロジェクト進行と実験管理を担当するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| Studio Producer | `project-management/studio-producer.md` | 複数プロジェクト横断管理・週次レポート・リソース配分 |
| Project Shipper | `project-management/project-shipper.md` | リリース管理・チェックリスト・ロールバック判断 |
| Experiment Tracker | `project-management/experiment-tracker.md` | A/Bテスト管理・実験計画書・統計分析・実験台帳 |

---

## Studio Operations（6名） - COO / CFO / CLO / CHRO 管轄

組織の基盤業務を支えるチーム。

| エージェント | ファイル | 管轄 | 役割 |
|-------------|---------|------|------|
| Support Responder | `studio-operations/support-responder.md` | COO | 問い合わせ対応・トリアージ（P0-P3）・FAQ整備 |
| Infrastructure Maintainer | `studio-operations/infrastructure-maintainer.md` | COO | サーバー監視・障害対応・ランブック整備 |
| Analytics Reporter | `studio-operations/analytics-reporter.md` | CFO | KPIレポート・データ分析・ダッシュボード設計 |
| Finance Tracker | `studio-operations/finance-tracker.md` | CFO | 予算管理・コスト分析・キャッシュフロー予測 |
| Legal Compliance Checker | `studio-operations/legal-compliance-checker.md` | CLO | 法規制対応・利用規約レビュー・コンプライアンスチェック |
| HR Coordinator | `studio-operations/hr-coordinator.md` | CHRO | 採用プロセス・オンボーディング・評価テンプレート |

---

## Testing（5名） - CTO 管轄

品質検証と業務最適化を担当するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| API Tester | `testing/api-tester.md` | API機能テスト・テスト計画書・プライバシー保護 |
| Performance Benchmarker | `testing/performance-benchmarker.md` | パフォーマンス測定・ベンチマーク・ボトルネック特定 |
| Test Results Analyzer | `testing/test-results-analyzer.md` | テスト結果集計・失敗パターン分析・リリース判断 |
| Tool Evaluator | `testing/tool-evaluator.md` | ツール比較・TCO算定・ベンダーロックインリスク評価 |
| Workflow Optimizer | `testing/workflow-optimizer.md` | 業務プロセス分析・ボトルネック特定・自動化提案 |

---

## ディレクトリ構成

```
.claude/
├── agents/                    # エージェント定義（57名）
│   ├── engineering/           # 技術チーム（10名）
│   ├── product/               # プロダクトチーム（4名）
│   ├── marketing/             # マーケティングチーム（12名）
│   ├── design/                # デザインチーム（7名）
│   ├── project-management/    # プロジェクト管理チーム（3名）
│   ├── studio-operations/     # スタジオ運営チーム（6名）
│   ├── testing/               # テスト・品質チーム（5名）
│   └── orchestra/             # CxOボード + 記録係（11名）
│
├── rules/                     # テーマ別ルールファイル（パススコープ対応）
│   ├── work-style.md          # 作業スタイル・要件確認
│   ├── agents.md              # エージェント管理・ボードミーティング
│   ├── campaign-lp.md         # キャンペーンLP制作チェックリスト
│   ├── branding.md            # ブランディング標準フロー
│   ├── development-workflow.md # 開発フェーズゲート（必須ワークフロー）
│   ├── project-setup.md       # プロジェクト事前確認
│   ├── fixedassets.md         # Ledgea プロジェクト固有情報
│   ├── rust-conventions.md    # Rust コーディング規約（パススコープ: src/**/*.rs）
│   ├── locales.md             # i18n ルール（パススコープ: locales/**/*.json）
│   ├── landing-pages.md       # LP/静的HTML ルール（パススコープ: public/**/*.html）
│   └── orchestra-logs.md      # 議事録ルール（パススコープ: .claude/orchestra/logs/**/*.md）
│
├── commands/                  # カスタムスラッシュコマンド
│   ├── retrospective.md       # /project:retrospective [name] — 振り返り実施
│   ├── board-meeting.md       # /project:board-meeting [agenda] — ボードミーティング
│   └── new-project.md         # /project:new-project [name] — 新規プロジェクト開始
│
├── skills/                    # 自動トリガーワークフロー（今後追加）
│
├── orchestra/logs/            # ボードミーティング記録
│   ├── meetings/              # 議事録
│   ├── decisions/             # 意思決定ログ
│   ├── learnings/             # 学習・教訓
│   └── retrospectives/        # 振り返り結果
│
├── settings.json              # チーム権限設定（allow/deny）
├── settings.local.json        # 個人権限オーバーライド（gitignored）
└── launch.json                # プレビューサーバー設定

brand/                         # ブランド素材（Ledgea）
CLAUDE.md                      # プロジェクトメモ（200行以下）
```

---

## 使い方

### カスタムコマンド

```
/project:new-project my-campaign      # 新規プロジェクト開始（要件確認フロー付き）
/project:retrospective my-campaign    # 振り返りの実施
/project:board-meeting 新機能の検討     # ボードミーティング開催
```

### 個別エージェントの呼び出し

```
@frontend-developer ログインページのUIを実装して
@copywriter この機能のキャッチコピーを10案考えて
@api-tester このエンドポイントのテスト計画を作って
```

### CxOボードミーティング

```
@orchestrator 新規プロジェクトの立ち上げについてボードミーティングを開催して
@orchestrator 前回からの振り返りを行って
```

### 特定CxOへの相談

```
@cto 技術スタックの選定について相談したい
@cmo マーケティング戦略をレビューして
@cvo このLPのビジュアル品質をチェックして
```

---

## 設計思想

- **1エージェント1責務**: 1人に全てをやらせない。専門エージェントが専門領域で品質を追求する
- **リサーチ駆動**: 制作前に徹底的なリサーチ。天井/底分析で成功パターンを抽出
- **品質ゲート（Weapons Check）**: 全成果物を Invention Novelty × Copy Intensity の2軸で評価。基準未満は差し戻し
- **パフォーマンス責任**: ビュー数・CTR・CVRの結果に責任を持つ。作って終わりではない
- **要件確認ファースト**: 推測で進めない。作業前に依頼者に質問し、ファクトチェックを行う
- **パススコープルール**: `.claude/rules/` でファイルパターンに応じたルールを自動適用。トークン効率を最適化
- **エージェント間連携**: 孤立したサイロではなく、構造化データでインプット/アウトプットを受け渡し
- **CxOによる統括**: Claude Architect レベルの知識を持つCxOが各専門領域を統括
- **学習と進化**: 記録係が議事録・学習ログを蓄積し、振り返りを通じてルールを更新
- **ツール制限**: `tools` フィールドで各エージェントのツールアクセスを最小権限に制限
- **permissionMode: acceptEdits**: 全エージェントで編集承認を自動化

---

## エージェント共通仕様

```yaml
---
name: agent-name
description: エージェントの説明
model: sonnet          # orchestra CxO は opus
tools: Read, Write, Edit, Glob, Grep  # 役割に応じて制限
permissionMode: acceptEdits
---
```

| カテゴリ | tools |
|---------|-------|
| engineering | Read, Write, Edit, Glob, Grep, Bash |
| design | Read, Write, Edit, Glob, Grep |
| marketing / product | Read, Write, Edit, Glob, Grep, WebSearch, WebFetch |
| project-management / studio-operations | Read, Write, Edit, Glob, Grep |
| testing | Read, Glob, Grep, Bash |
| orchestra (CxO) | Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch |
