---
description: OpenAI API / SDK の統合知識。Chat Completions、Assistants API、Embeddings、Image Generation、Whisper、TTS、Function Calling、Fine-tuningを含む。
---

# OpenAI API スキル

## 作業開始チェックリスト

1. `OPENAI_API_KEY` の管理方法を確認する
2. 組織 ID、プロジェクト ID、利用上限の設定を確認する
3. ユースケースに最適なモデルを選定する（gpt-4o / gpt-4o-mini / o3 / o4-mini 等）
4. 現在のティアのレート制限（RPM / TPM / RPD）を把握する
5. 想定リクエスト量とトークン使用量からコストを事前試算する

## コーディングルール

### API 呼び出し
- Chat Completions API（`/v1/chat/completions`）を標準とする
- モデル ID は定数化する。文字列リテラルで散在させない
- `response_format: { type: "json_object" }` で JSON 出力を強制する（該当する場合）
- Structured Outputs（`response_format: { type: "json_schema" }`）を活用して型安全な出力を得る

### Function Calling
- 関数定義の `description` は明確に記述する
- `parameters` は JSON Schema で厳密に型定義する
- `strict: true` で Structured Outputs を有効化し、スキーマ準拠を保証する
- 関数実行結果は `tool` ロールのメッセージで必ず返す

### Embeddings
- テキスト埋め込みには `text-embedding-3-small`（コスト効率）または `text-embedding-3-large`（精度重視）を使用する
- `dimensions` パラメータでベクトル次元数を削減してコスト・速度を最適化する
- 大量テキストの埋め込みはバッチで処理する（1 リクエスト最大 2048 入力）

### Streaming
- ユーザー向けレスポンスには Streaming を使用する
- Server-Sent Events のパースを正しく実装する
- Function Calling + Streaming の組み合わせ時、`tool_calls` のチャンク結合を正しく処理する

### コスト最適化
- タスクの複雑さに応じてモデルを使い分ける
- 不要に長い `max_tokens` を指定しない
- Embeddings のキャッシュ（同一テキストの再埋め込み防止）を実装する
- バッチ API（`/v1/batches`）で大量処理のコストを50%削減する

## セキュリティ

- `OPENAI_API_KEY` をソースコード・ログ・エラーメッセージに含めない
- ユーザー入力を System prompt に直接埋め込まない（Prompt Injection 対策）
- Function Calling でのユーザー入力は必ずバリデーションする
- 画像生成のプロンプトに不適切コンテンツフィルタを実装する
- API レスポンスに含まれるユーザーデータのログ出力を制限する

## テスト

- **義務**: Function Calling の入出力を全パターンテストする
- **テスト種類**:
  - ユニットテスト: プロンプト生成ロジック・レスポンスパーサーの検証
  - 統合テスト: 実 API を使った E2E テスト
  - Eval テスト: プロンプト品質の回帰テスト
  - コストテスト: トークン使用量が想定範囲内であることを確認する
- **完了前必須**: `finish_reason` の全パターンのハンドリングを確認する

## 完了条件

- [ ] API 呼び出しが正常に動作している
- [ ] エラーハンドリング（400, 401, 429, 500, 503）が実装されている
- [ ] API キーがソースコードにハードコードされていない
- [ ] Streaming が正しく動作している（該当する場合）
- [ ] Function Calling が正常に動作し、エラーケースも処理されている
- [ ] コスト最適化が適用されている

## 連携先エージェント

- **ai-engineer**: RAG / エージェントアーキテクチャ全体の設計
- **claude-api → platforms/claude-api**: マルチプロバイダー構成時のフォールバック設計
- **backend-architect**: API ラッパー層の設計
- **finance-tracker**: API 利用コストの月次モニタリング
- **google-tts → platforms/google-tts**: 音声処理パイプラインでの TTS/Whisper 連携
