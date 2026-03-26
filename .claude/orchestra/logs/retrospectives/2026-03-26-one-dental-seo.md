# 振り返り（レトロスペクティブ） - 2026-03-26

## セッション概要

**テーマ**: ワンデンタル (one-dental.jp) SEO / AI SEO 改善
**参加CxO**: CTO, CMO, COO, 記録係
**対象作業**: 既存歯科求人サイトへのAI SEO導入 + 従来SEO改善（7項目）

---

## 作業サマリ

| # | 内容 | 種別 | ステータス |
|---|------|------|-----------|
| 1 | llms.txt 4ファイル新規作成（日英 x 概要/詳細） | AI SEO | ✅ 完了 |
| 2 | layout.tsx に llms.txt link タグ追加 | AI SEO | ✅ 完了 |
| 3 | robots.txt に AI クローラー8種の許可追加 | AI SEO | ✅ 完了 |
| 4 | `lang="jp"` → `lang="ja"` 修正 | 基本SEO | ✅ 完了 |
| 5 | site.webmanifest の name/short_name 設定 | PWA/SEO | ✅ 完了 |
| 6 | WebSite JSON-LD に SearchAction 追加 | 構造化データ | ✅ 完了 |
| 7 | Twitter Card メタタグ追加 | SNS共有 | ✅ 完了 |
| 8 | next.config.js に rewrites 追加（動的ルート回避） | インフラ | ✅ 完了（ユーザー対応） |

---

## Keep（良かった点）

### K1. 前回プロジェクト（3x3-portal）の学習が活かされた [担当: CTO/CMO]
- AI SEO の4ファイル構成（日英 x 概要/詳細）を初回から用意した
- 前回は英語版を後追いで追加したが、今回は最初から日英同時作成
- `ai-seo.md` ルールが横展開テンプレートとして正しく機能した

### K2. metadata API の既知問題を回避した [担当: CTO]
- 前回の教訓（Next.js metadata APIはtext/plain linkタグを生成しない）を踏まえ、最初からHTML直書きを採用
- 不要なデプロイ→修正サイクルを1回分回避できた

### K3. 既存SEOの改善点を網羅的に検出した [担当: CMO]
- AI SEOだけでなく、既存の問題（lang属性誤り、manifest空値、Twitter Card欠如等）も併せて検出・修正
- 特に `lang="jp"` → `lang="ja"` は検索エンジンの言語判定に影響する重要な修正

### K4. 並列作業による効率的な実装 [担当: COO]
- 4つのllms.txtファイルを2回のバッチで作成、5つの既存ファイル編集を並列実行
- 計9ファイルの変更を1セッションで完了

---

## Problem（問題点）

### P1. JSX属性のキャメルケース規則を見落とした [担当: CTO]
- **事象**: `hreflang="en"` と記述したが、JSXでは `hrefLang="en"` が正しい
- **影響**: ステージングのDockerビルドで型エラーが発生し、デプロイ失敗
- **根本原因**: HTMLの属性名をそのままJSXに持ち込んだ。JSXではキャメルケース変換が必要（`class` → `className`, `hreflang` → `hrefLang` 等）
- **再発防止**: JSXでHTML属性を書く際は、キャメルケース変換を意識する

### P2. 動的ルートによる静的ファイルの遮蔽を予測できなかった [担当: CTO]
- **事象**: `public/llms.txt` がステージングで404を返した
- **影響**: ユーザーが `next.config.js` に `rewrites` を追加して対応
- **根本原因**: `app/(serverLayout)/[jobType]/` の動的ルートが `/llms.txt` をキャッチし、public/ の静的ファイルより優先された
- **再発防止**: Next.js App Router で `[slug]` 動的ルートがある場合、public/ の新規ファイルがルートに遮蔽されないか確認する

### P3. ローカルdevサーバーでの検証ができなかった [担当: CTO/COO]
- **事象**: `.env` 未設定 + `next.config.js` の既存警告により `npm run dev` が起動せず、ローカル検証をスキップした
- **影響**: P1, P2 がデプロイ後に発覚
- **根本原因**: 本プロジェクトのローカル開発環境がセットアップされていない状態で作業を開始した
- **再発防止**: 外部プロジェクトの変更時、最低限のローカル検証環境（.env.example、ビルド通過）を事前に確認する

---

## Try（改善策）

### T1. Next.js 動的ルートと静的ファイルの競合チェックをルール化
- `[slug]` / `[...catchAll]` 動的ルートがあるプロジェクトで `public/` にファイルを追加する場合、ルート競合の有無を確認する手順をルールに追加する

### T2. JSXのHTML属性変換チェックリスト
- `hreflang` → `hrefLang`、`tabindex` → `tabIndex` 等、頻出する変換をnextjs.mdに追記する

### T3. 外部プロジェクト変更時のローカル検証必須化
- `.env` がない場合でも `npm run build` でビルド検証を試みる（env未設定でもTypeScriptの型チェックは通る）

---

## 決定事項

| 決定内容 | 担当CxO | 期限 |
|---------|---------|------|
| Next.js ルールに動的ルートと静的ファイル競合の注意を追加 | CTO | 即日 |
| Next.js ルールにJSX属性キャメルケース変換の注意を追加 | CTO | 即日 |
| AI SEO導入チェックリストに「動的ルート競合確認」を追加 | CMO | 即日 |
| 1ヶ月後にAI検索での引用状況を測定 | CMO | 2026-04-26 |

---

## 参照

- 変更リポジトリ: `/Users/tomohirok/Documents/Github/Career-Bridge/jobseeker-front/`
- 本番URL: https://one-dental.jp/
- 前回AI SEO振り返り: `.claude/orchestra/logs/retrospectives/2026-03-25-3x3-ai-seo-v2.md`
