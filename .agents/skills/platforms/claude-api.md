---
description: Claude API / Anthropic SDK / Claude Agent SDK の統合知識。Messages API、Tool Use、Prompt Caching、Batch API、Streaming、コスト最適化を含む。
---

# Claude API スキル

## 作業開始チェックリスト

1. `ANTHROPIC_API_KEY` の管理方法（環境変数、Secret Manager 等）を確認する
2. ユースケースに最適なモデルを選定する（opus: 高品質推論、sonnet: バランス、haiku: 高速低コスト）
3. 現在のティアのレート制限（RPM / TPM）を把握する
4. 既にある Claude API 統合コードがないか確認する
5. 想定リクエスト量とトークン使用量からコストを事前試算する

## コーディングルール

### API 呼び出し
- 最新の Messages API（`/v1/messages`）を使用する。旧 Completions API は使用しない
- モデル ID は定数化する（`claude-sonnet-4-6` 等）。文字列リテラルで散在させない
- `max_tokens` を必ず指定する。省略時のデフォルトに依存しない
- レスポンスの `stop_reason` を必ず確認する（`end_turn` / `max_tokens` / `tool_use`）

### Tool Use 設計
- ツール定義の `description` は LLM が理解しやすい明確な記述にする
- `input_schema` は JSON Schema で厳密に型定義する（required フィールドを明示）
- ツール実行結果は `tool_result` で必ず返す（成功・失敗両方）
- ツールの副作用（DB 書き込み、外部 API 呼び出し等）は明示的に文書化する

### Prompt 設計
- System prompt でエージェントの役割・制約・出力フォーマットを明示する
- 長い context は Prompt Caching を活用する（`cache_control: { type: "ephemeral" }` ）
- マルチターン会話では不要な履歴を適切にトリミングする
- 構造化出力が必要な場合、JSON モードまたは Tool Use で強制する

### Streaming
- ユーザー向けレスポンスには Streaming を使用する（体感速度向上）
- `message_start` → `content_block_delta` → `message_stop` のイベント順序を正しく処理する
- Streaming 中のエラーハンドリング（接続切断、タイムアウト）を実装する

### コスト最適化
- Prompt Caching を積極的に活用する（繰り返し使う System prompt / 大量 context）
- 大量処理には Batch API を使用する（50% コスト削減）
- タスクの複雑さに応じてモデルを使い分ける（分類: haiku、生成: sonnet、推論: opus）
- 不要に長い出力を要求しない（`max_tokens` を適切に制限）

### エラーハンドリング
- 400（不正リクエスト）/ 401（認証失敗）/ 429（レート制限）/ 529（API 過負荷）を区別する
- 429 / 529 は `retry-after` ヘッダーを尊重してリトライする
- Overloaded エラー時はモデルのフォールバック（opus → sonnet → haiku）を検討する

## セキュリティ

- `ANTHROPIC_API_KEY` をソースコード・ログ・エラーメッセージに含めない
- ユーザー入力を System prompt に直接埋め込まない（Prompt Injection 対策）
- Tool Use でのユーザー入力は必ずバリデーションする（コマンドインジェクション防止）
- API レスポンスに含まれるユーザーデータのログ出力を制限する
- Batch API のジョブ結果は適切な保持期間後に削除する

## テスト

- **義務**: Tool Use の入出力を全パターンテストする
- **テスト種類**:
  - ユニットテスト: プロンプト生成ロジック・レスポンスパーサーの検証
  - 統合テスト: 実 API を使った E2E テスト（テスト用 API キーで実行）
  - Eval テスト: プロンプト品質の回帰テスト（期待出力との一致率）
  - コストテスト: トークン使用量が想定範囲内であることを確認する
- **完了前必須**: `stop_reason` の全パターン（`end_turn`, `max_tokens`, `tool_use`）のハンドリングを確認する

## 完了条件

- [ ] API 呼び出しが正常に動作している
- [ ] エラーハンドリング（400, 401, 429, 529）が実装されている
- [ ] API キーがソースコードにハードコードされていない
- [ ] Streaming が正しく動作している（該当する場合）
- [ ] Tool Use の全ツールが正常に動作し、エラーケースも処理されている
- [ ] コスト最適化が適用されている（Prompt Caching / モデル選択 / Batch API）
- [ ] Prompt Injection 対策が実装されている

## 連携先エージェント

- **ai-engineer**: RAG / エージェントアーキテクチャ全体の設計
- **openai-api → platforms/openai-api**: マルチプロバイダー構成時のフォールバック設計
- **backend-architect**: API ラッパー層の設計、キュー/バッチ処理のアーキテクチャ
- **finance-tracker**: API 利用コストの月次モニタリング
