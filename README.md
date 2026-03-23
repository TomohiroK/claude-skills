# Claude Skills - AI Agent Organization

Claude Code のサブエージェントとして動作する、55名のAIエージェント組織。
各エージェントは専門分野を持ち、CxOボードによる統括のもとで連携して業務を遂行する。

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

経営判断・戦略策定を行うボードメンバー。振り返りを通じて組織全体を進化させる。

| 役職 | ファイル | 役割 |
|------|---------|------|
| **Orchestrator** | `orchestra/orchestrator.md` | 全体指揮・CxO召集・ボードミーティング運営 |
| **CEO** | `orchestra/ceo.md` | ビジョン策定・最終意思決定・全体統括 |
| **CTO** | `orchestra/cto.md` | 技術戦略・アーキテクチャ決定・技術的負債管理 |
| **COO** | `orchestra/coo.md` | オペレーション最適化・プロセス改善・チーム間調整 |
| **CFO** | `orchestra/cfo.md` | 財務戦略・予算管理・コスト最適化・投資判断 |
| **CMO** | `orchestra/cmo.md` | マーケティング戦略・ブランド構築・顧客獲得 |
| **CLO** | `orchestra/clo.md` | 法的リスク管理・コンプライアンス・契約レビュー |
| **CSO** | `orchestra/cso.md` | 中長期戦略・事業開発・新規事業探索 |
| **CVO** | `orchestra/cvo.md` | ブランドビジョン・デザイン品質・UX統括 |
| **CHRO** | `orchestra/chro.md` | 組織設計・人材配置・オンボーディング・評価設計 |
| **Secretary** | `orchestra/secretary.md` | 議事録・決定事項追跡・学習ログ管理 |

---

## Engineering（8名） - CTO 管轄

技術的な設計・実装を担当するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| Frontend Developer | `engineering/frontend-developer.md` | React/Vue/Next.js等でのUI構築・コンポーネント設計 |
| Backend Architect | `engineering/backend-architect.md` | API設計・DB設計・マイクロサービス構成 |
| Mobile App Builder | `engineering/mobile-app-builder.md` | React Native/Flutter/Swift/KotlinでのiOS/Android開発 |
| AI Engineer | `engineering/ai-engineer.md` | LLM統合・プロンプトエンジニアリング・RAG構築 |
| DevOps Automator | `engineering/devops-automator.md` | CI/CD構築・Docker・IaC・Kubernetes |
| Rapid Prototyper | `engineering/rapid-prototyper.md` | 高速プロトタイピング・PoC作成・技術検証 |
| QA Engineer | `engineering/qa-engineer.md` | 品質保証戦略・テスト計画・バグトリアージ |
| Data Engineer | `engineering/data-engineer.md` | データパイプライン・ETL/ELT・データ品質管理 |

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

マーケティング戦略の立案と実行を担当するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| Growth Hacker | `marketing/growth-hacker.md` | AARRR分析・グロース実験・ファネル最適化 |
| Content Creator | `marketing/content-creator.md` | ブログ・ニュースレター・ホワイトペーパー執筆 |
| Copywriter | `marketing/copywriter.md` | キャッチコピー・広告文・LPコピー・シナリオ作成 |
| Trend Analyst | `marketing/trend-analyst.md` | バズ分析・トレンド再現・季節イベント予測 |
| Brand Ambassador | `marketing/brand-ambassador.md` | ブランドの顔・ストーリーテリング・インフルエンサー連携 |
| Community Manager | `marketing/community-manager.md` | Discord/Slack運営・イベント企画・ユーザー交流促進 |
| SEO Specialist | `marketing/seo-specialist.md` | テクニカルSEO・キーワード戦略・サイト構造最適化 |
| TikTok Strategist | `marketing/tiktok-strategist.md` | TikTokトレンド活用・ショート動画コンテンツ企画 |
| Instagram Curator | `marketing/instagram-curator.md` | フィード設計・ストーリーズ/リール戦略 |
| Twitter Engager | `marketing/twitter-engager.md` | ツイート・スレッド企画・リアルタイムマーケティング |
| Reddit Community Builder | `marketing/reddit-community-builder.md` | サブレディット参入戦略・AMA企画 |
| App Store Optimizer | `marketing/app-store-optimizer.md` | ASO・キーワード最適化・ストアページ改善 |

---

## Design（6名） - CVO 管轄

デザイン品質とユーザー体験を統括するチーム。

| エージェント | ファイル | 役割 |
|-------------|---------|------|
| UI Designer | `design/ui-designer.md` | 画面設計・デザインシステム構築・インタラクション設計 |
| UX Researcher | `design/ux-researcher.md` | ユーザビリティ調査・ペルソナ作成・ジャーニーマップ |
| Brand Guardian | `design/brand-guardian.md` | ブランドガイドライン策定・整合性チェック・違反検出 |
| Visual Storyteller | `design/visual-storyteller.md` | データビジュアライゼーション・プレゼンテーション設計 |
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
├── agents/
│   ├── engineering/          # 技術チーム（8名）
│   ├── product/              # プロダクトチーム（4名）
│   ├── marketing/            # マーケティングチーム（12名）
│   ├── design/               # デザインチーム（6名）
│   ├── project-management/   # プロジェクト管理チーム（3名）
│   ├── studio-operations/    # スタジオ運営チーム（6名）
│   ├── testing/              # テスト・品質チーム（5名）
│   └── orchestra/            # CxOボード + 記録係（11名）
│
└── orchestra/
    └── logs/                 # ボードミーティング記録
        ├── meetings/         # 議事録
        ├── decisions/        # 意思決定ログ
        ├── learnings/        # 学習・教訓
        ├── retrospectives/   # 振り返り結果
        └── c{x}o/            # 各CxO個別ログ
```

---

## 使い方

### 個別エージェントの呼び出し

Claude Code で `@` を使ってエージェントを指定:

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
@cfo この投資のROIを分析して
```

---

## 設計思想

- **全エージェントに共通基盤**: 作業開始プロトコル・完了条件・他エージェントとの連携を定義
- **具体的な行動指針**: 「ベストプラクティスに従う」ではなく具体的なルール・テンプレート・チェックリスト
- **エージェント間連携**: 孤立したサイロではなく、インプット/アウトプットの受け渡しを明示
- **CxOによる統括**: 各専門領域をCxOが統括し、ボードミーティングで横断的な意思決定
- **学習と進化**: 記録係が議事録・学習ログを蓄積し、振り返りを通じて組織を改善
- **permissionMode: acceptEdits**: 全エージェントで編集承認を自動化
