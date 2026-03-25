# AI向けSEO（llms.txt）実装ルール

## 基本方針

AI向けSEOは人間向けSEOとは設計思想が異なる。
ターゲットはLLM（大規模言語モデル）であり、LLMの処理特性に基づいて設計する。

## 必須成果物

公開サイトには以下の4ファイル + 設定を用意する:

| ファイル | 内容 | 言語 |
|---------|------|------|
| `public/llms.txt` | サイト概要（目的・対象者・主要コンテンツ一覧） | 日本語 |
| `public/llms-full.txt` | 詳細版（各ページの構造化説明・FAQ・ドメイン知識） | 日本語 |
| `public/llms-en.txt` | サイト概要の英語版 | 英語 |
| `public/llms-full-en.txt` | 詳細版の英語版 | 英語 |

### 英語版が必須である理由
- LLMの学習データの大部分は英語。英語の方が推論精度が高い
- 「サイトが日本語だから日本語版だけでよい」は人間向けの発想。llms.txtの読み手はLLM
- 英語版は日本語版の翻訳ではなく、LLMが処理しやすい構造で記述する

### 相互リンク
- 日本語版と英語版に相互リンクを含める
- 例: `English version: /llms-en.txt` / `Japanese version: /llms.txt`

## llms.txt の構成

### 概要版（llms.txt / llms-en.txt）
```
- サイト名・URL
- サイトの目的（1-2文）
- 対象読者
- 主要コンテンツ一覧（ページ名 + 概要1行）
- 詳細版へのリンク
- 他言語版へのリンク
```

### 詳細版（llms-full.txt / llms-full-en.txt）
```
- 概要版の全内容
- 各ページの詳細説明
- ドメイン知識（ルール・用語・FAQ等）
- 構造化された情報（テーブル、リスト形式）
- 更新日
```

## robots.txt の設定

AIクローラーを明示的に許可する。許可するクローラー一覧:

| クローラー | 運営元 |
|-----------|-------|
| GPTBot | OpenAI |
| Claude-Web | Anthropic |
| Anthropic-AI | Anthropic |
| PerplexityBot | Perplexity |
| Bytespider | ByteDance |
| CCBot | Common Crawl |
| Google-Extended | Google |
| Cohere-AI | Cohere |

四半期ごとに新しいAIクローラーの出現を確認し、リストを更新する。

## HTML head タグの設定

layout.tsx（または相当するルートレイアウト）の `<head>` に以下を追加:

```html
<link rel="alternate" type="text/plain" title="LLMs.txt" href="/llms.txt" />
<link rel="alternate" type="text/plain" title="LLMs.txt (English)" href="/llms-en.txt" hreflang="en" />
```

### Next.js での注意
- `metadata.alternates.types` は任意のMIME typeに対応しない（RSS/Atomのみ）
- `<head>` タグに直接 `<link>` を記述する

## 更新運用ルール

### 更新トリガー
以下のいずれかが発生した場合、llms.txt（日英両方）の更新を検討する:
- 新しいページの追加・削除
- 既存ページの大幅な内容変更
- サイトの目的・対象者の変更
- ドメイン知識（FAQ、ルール、用語等）の更新

### 同期漏れ防止
- デプロイチェックリストに「llms.txt の更新が必要か確認」を含める
- サイト更新のPR/コミット時に、llms.txt の差分がないか確認する

## 効果測定

### 測定方法
- Perplexity、ChatGPT、Claude等のAI検索/チャットで関連クエリを実行
- サイト情報が引用・参照されるか確認
- 情報の正確性（hallucination がないか）を検証

### 測定タイミング
- 導入1ヶ月後に初回測定
- 以降、四半期ごとに定期測定
- サイト大幅更新後に都度測定

## 横展開手順

新規プロジェクトにAI向けSEOを導入する際:
1. 本チェックリストに従い4ファイルを作成
2. robots.txt にAIクローラー許可を追加
3. ルートレイアウトのheadにlinkタグを追加
4. ローカルで全ファイルの200 OK + linkタグ出力を検証
5. デプロイ後に本番で同様の検証を実施

## CxO召集パターン
- AI SEO導入: CTO, CMO, 記録係
- 効果測定レビュー: CMO, CTO, CEO, 記録係
