# コーディング品質ルール（言語横断）

## デバッグしやすさ

### エラーメッセージに文脈を含める
- Bad: `throw new Error("Not found")`
- Good: `throw new Error("User not found: userId=${id}, source=getProfile")`
- エラーメッセージには「何が」「どの値で」「どこで」の3点を含める

### 構造化ログ
- `console.log("here")` や `print("debug")` を使わない
- ログには操作名・入力値・結果を含める
- ログレベルを使い分ける（error/warn/info/debug）

### 早期リターン（Guard Clause）
- ネストを深くせず、異常系を先に弾いて正常系をフラットに書く
- Bad: `if (user) { if (user.active) { ... } }`
- Good: `if (!user) return; if (!user.active) return; ...`

### エラーの握りつぶし禁止
- 空の catch ブロック禁止。最低限ログを出す
- エラーを再スローする場合は元のエラーを cause に含める

## メンテしやすさ

### 関数は1つの責務
- 1関数 = 1つのこと。「〜して、さらに〜する」関数を作らない
- 関数名で何をするか分かるようにする。コメントで補足が必要な関数名は悪い名前

### 命名規則
- 略語を避ける（`usr` → `user`、`btn` → `button`、`tmp` → `temporary`）
- bool型は `is`/`has`/`can`/`should` で始める
- コレクションは複数形（`users`、`items`）

### マジックナンバー・マジックストリング禁止
- 数値・文字列リテラルは定数に抽出する
- Bad: `if (status === 3)` / Good: `if (status === STATUS_ACTIVE)`
- 例外: 0, 1, -1, "", true, false は文脈上自明な場合のみ許可

### デッドコード禁止
- コメントアウトしたコードを残さない。Gitに履歴がある
- 使われていない変数・関数・import は削除する

### 条件分岐の明確化
- 複雑な条件は変数に抽出して名前を付ける
- Bad: `if (age >= 18 && !isBanned && subscription.isActive)`
- Good: `const canAccess = age >= 18 && !isBanned && subscription.isActive; if (canAccess)`

## パフォーマンス — 並列実行の積極採用

### 独立した処理は並列化する
- 依存関係のないI/O（API呼び出し、DB問い合わせ、ファイル読み書き）は並列実行する
- Bad: `const a = await fetchA(); const b = await fetchB();`（直列: A完了まで B が待つ）
- Good: `const [a, b] = await Promise.all([fetchA(), fetchB()]);`

### 言語別パターン
- **JS/TS**: `Promise.all()` / `Promise.allSettled()` を使う。ループ内の await は原則禁止
- **Rust**: `tokio::join!` / `futures::join_all` / `rayon` の並列イテレータ
- **Python**: `asyncio.gather()` / `concurrent.futures.ThreadPoolExecutor`
- **Go**: goroutine + `sync.WaitGroup` / `errgroup`

### ループ内の await は原則禁止
- Bad: `for (const id of ids) { await process(id); }`
- Good: `await Promise.all(ids.map(id => process(id)));`
- 件数が多い場合はバッチ分割（例: 10件ずつ `Promise.all`）

### Promise.all vs Promise.allSettled の使い分け
- 全て成功が必須 → `Promise.all`（1つ失敗で即中断）
- 部分成功を許容 → `Promise.allSettled`（全完了後に結果を個別判定）
- 部分失敗時のフォールバック戦略を必ず設計する

### 並列化の判断基準
- I/O待ちが発生する処理 → 並列化する
- CPU律速の計算処理 → ワーカースレッド/プロセス分割を検討
- 共有状態への書き込みがある → 排他制御を設計してから並列化する（安易にやらない）

### エラーハンドリングとの統合
- 並列実行時のエラーは「どのタスクが失敗したか」を特定できるようにする
- Bad: `Promise.all(tasks)` のエラーで「何が失敗したか分からない」
- Good: 各タスクに名前/IDを付与し、エラー時に含める

## テスタビリティ

### 副作用の分離
- 計算ロジックとI/O（DB、API、ファイル）を分離する
- 純粋関数を優先し、副作用は境界層に押し出す

### DI可能な設計
- 外部依存（DB、API、時刻、乱数）はインターフェース経由で注入可能にする
- テスト時にモックに差し替えられる構造にする

### テストしやすい粒度
- 1関数のテストが10行を超えるなら、関数が大きすぎる
- テストデータの準備が複雑なら、関数の依存が多すぎる

## 可読性

### ネストは最大3階層
- 3階層を超えたら関数に抽出するか、早期リターンでフラットにする

### コメントは「なぜ」を書く
- 「何を」はコードが語る。「なぜこの実装を選んだか」を書く
- TODO/FIXME/HACK には理由と担当者を書く: `// TODO(tomohiro): APIv2移行後に削除`

### ファイルサイズの目安
- 1ファイル300行を超えたら分割を検討する
- 1関数50行を超えたら分割を検討する

### import の整理
- 標準ライブラリ → 外部パッケージ → 内部モジュールの順に並べる
- グループ間に空行を入れる

## セキュリティ — DOM操作

### innerHTML による動的描画の原則禁止
- 外部データ（JSON、APIレスポンス、ユーザー入力）を DOM に描画する場合、`innerHTML` を使わない
- `textContent` + `createElement` で要素を構築する
- Bad: `container.innerHTML = '<h3>' + data.title + '</h3>'`
- Good: `const h3 = document.createElement('h3'); h3.textContent = data.title;`

### 動的URLの構築時はサニタイズする
- ユーザー由来の値（slug、ID等）をURLパスに含める場合、許可文字のみ通すフィルタを適用する
- Bad: `href = '/articles/' + slug + '.html'`
- Good: `href = '/articles/' + slug.replace(/[^a-z0-9-]/g, '') + '.html'`

### 発生事例（2026-03-28）
corp.tmkproduct.com の Insights セクションで articles.json を innerHTML で描画していた。コードレビューでXSS脆弱性とパストラバーサルを検出し、textContent + createElement + slug sanitize に修正
