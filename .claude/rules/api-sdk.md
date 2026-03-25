# API / SDK 開発ルール

## 関数ドキュメントの必須化

### 全パブリック関数に JSDoc / docstring を書く
- パラメータの型・説明・制約（必須/任意、範囲、フォーマット）
- 戻り値の型・構造・エラー時の値
- 例外/エラーの種類と発生条件
- 使用例（最低1つ）

```typescript
/**
 * Fetch a user by ID from the user service.
 *
 * @param userId - UUID of the user to retrieve
 * @returns The user object, or null if not found
 * @throws {AuthenticationError} When the API token is invalid or expired
 * @throws {RateLimitError} When the rate limit (100 req/min) is exceeded
 *
 * @example
 * const user = await getUser("550e8400-e29b-41d4-a716-446655440000");
 */
```

### 内部関数にも最低限の説明
- 「なぜこの関数が存在するか」を1行で書く
- パラメータと戻り値の型がコードから自明でない場合は補足する

## 冪等性 — 再実行可能な設計

### API呼び出しは常に再実行可能にする
- 同じリクエストを2回送っても結果が変わらない設計にする
- リクエストに冪等キー（idempotency key）を含める

```typescript
// Bad: 再実行すると二重作成
await createOrder({ item: "widget", qty: 1 });

// Good: 冪等キーで二重実行を防止
await createOrder({
  item: "widget",
  qty: 1,
  idempotencyKey: `order-${userId}-${timestamp}`,
});
```

### データ変更操作の冪等パターン
- **作成（POST）**: 冪等キーで重複検出。既に存在すれば既存を返す
- **更新（PUT/PATCH）**: 全体置換（PUT）は本質的に冪等。部分更新（PATCH）はバージョンチェック付きで
- **削除（DELETE）**: 既に削除済みなら 204 を返す（エラーにしない）

### バッチ処理の再開可能設計
- 途中で失敗した場合、最初からやり直さず失敗箇所から再開できる構造にする
- 処理済みアイテムをマーク（DB, ファイル, キュー等）し、再実行時にスキップする

```typescript
// Bad: 全件やり直し
for (const item of items) { await process(item); }

// Good: 処理済みをスキップして再開可能
for (const item of items) {
  if (await isProcessed(item.id)) continue;
  await process(item);
  await markProcessed(item.id);
}
```

## リトライとエラーハンドリング

### リトライ可能なエラーを区別する
- リトライ可能: ネットワークタイムアウト、429 Too Many Requests、5xx
- リトライ不可: 400 Bad Request、401 Unauthorized、404 Not Found

```typescript
const RETRYABLE_STATUS = new Set([408, 429, 500, 502, 503, 504]);

function isRetryable(error: ApiError): boolean {
  return RETRYABLE_STATUS.has(error.statusCode);
}
```

### Exponential Backoff を標準採用
- 即時リトライしない。指数バックオフ + ジッタ
- 最大リトライ回数を定数化する（デフォルト: 3回）
- リトライ間隔の上限を設定する（デフォルト: 30秒）

```typescript
const MAX_RETRIES = 3;
const BASE_DELAY_MS = 1000;
const MAX_DELAY_MS = 30000;

function getRetryDelay(attempt: number): number {
  const delay = Math.min(BASE_DELAY_MS * 2 ** attempt, MAX_DELAY_MS);
  const jitter = delay * 0.5 * Math.random();
  return delay + jitter;
}
```

### タイムアウトを必ず設定する
- API呼び出しにはタイムアウトを必ず設定する。デフォルト無制限は禁止
- 接続タイムアウトとリードタイムアウトを分離する

## レート制限の尊重

### レート制限情報をレスポンスヘッダーから読み取る
- `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `Retry-After` を活用
- 残数が少なくなったら自発的にスロットリングする

### 並列リクエスト数を制限する
- 無制限の並列実行は API を叩き潰す。同時実行数を定数化する

```typescript
const MAX_CONCURRENT_REQUESTS = 5;

async function processInBatches<T>(
  items: T[],
  processor: (item: T) => Promise<void>,
  concurrency = MAX_CONCURRENT_REQUESTS,
): Promise<void> {
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    await Promise.allSettled(batch.map(processor));
  }
}
```

## API レスポンスの型安全

### レスポンスを必ずバリデーションする
- 外部APIのレスポンスを信頼しない。スキーマバリデーション（Zod, io-ts等）を通す
- 型アサーション（`as SomeType`）でのキャストは禁止。ランタイムで検証する

```typescript
// Bad: 型キャストで信頼
const user = (await res.json()) as User;

// Good: ランタイムバリデーション
const user = UserSchema.parse(await res.json());
```

### エラーレスポンスにも型を定義する
- 正常系だけでなくエラー系のレスポンス型も定義し、呼び出し側で適切にハンドリングする

## 監査ログ（Audit Log）

### 全ての状態変更操作にログを残す
- データの作成・更新・削除、権限変更、認証イベントは必ず監査ログに記録する
- 「誰が・いつ・何を・どう変えたか・結果はどうだったか」の5W を含める

```typescript
interface AuditLogEntry {
  timestamp: string;        // ISO 8601
  actor: {
    id: string;             // 実行者のユーザーID / システムID
    type: "user" | "system" | "api_key";
  };
  action: string;           // "user.create" | "order.update" | "role.assign"
  resource: {
    type: string;           // "user" | "order" | "asset"
    id: string;             // 対象リソースのID
  };
  changes?: {
    before: Record<string, unknown>;  // 変更前の値
    after: Record<string, unknown>;   // 変更後の値
  };
  result: "success" | "failure";
  metadata?: {
    ip?: string;
    userAgent?: string;
    requestId?: string;     // リクエスト追跡用
    reason?: string;        // 変更理由（手動操作時）
  };
}
```

### ログの設計原則
- **追記専用（append-only）**: 監査ログは変更・削除しない。別テーブル/別ストレージに分離する
- **非同期書き込み**: 監査ログの書き込み失敗が本体処理をブロックしない設計にする。ただしログ欠損時のアラートは必須
- **構造化**: JSON形式で出力し、検索・集計が可能な状態にする
- **保持期間**: 規制・コンプライアンス要件に基づいて決定。最低1年を推奨

### 記録すべきイベント（最低限）
- **認証**: ログイン成功/失敗、ログアウト、トークン発行/失効、パスワード変更
- **認可**: 権限変更、ロール付与/剥奪、アクセス拒否
- **データ変更**: CRUD操作（特に更新・削除は変更前の値も記録）
- **管理操作**: 設定変更、ユーザー招待/削除、APIキー発行/失効
- **課金**: プラン変更、支払い、返金

### before/after の記録
- 更新操作では変更前後の差分を記録する。「何が変わったか」が追跡不能なログは無価値
- 全フィールドのスナップショットではなく、変更があったフィールドのみで十分

```typescript
// Bad: 何が変わったか不明
auditLog({ action: "user.update", userId: "123" });

// Good: 変更内容が追跡可能
auditLog({
  action: "user.update",
  resource: { type: "user", id: "123" },
  changes: {
    before: { role: "viewer" },
    after: { role: "admin" },
  },
});
```

### API レスポンスへの反映
- 状態変更APIのレスポンスに `requestId` を含め、監査ログとの紐付けを可能にする
- クライアント側からも「どのリクエストで変更されたか」を追跡できるようにする

### SDKラッパーへの統合
- SDKラッパー層に監査ログのフックを組み込む。個別の呼び出し箇所で書かなくて済む構造にする

```typescript
// ラッパー層で一元的にログ記録
async function updateUser(userId: string, data: UpdateUserInput): Promise<User> {
  const before = await getUser(userId);
  const after = await sdk.users.update(userId, data);
  await auditLog({
    action: "user.update",
    resource: { type: "user", id: userId },
    changes: { before, after },
    result: "success",
  });
  return after;
}
```

## SDK ラッパーの設計

### 外部SDKは薄いラッパーで包む
- 外部SDKを直接アプリケーション層から呼ばない。ラッパー層を1枚挟む
- ラッパー層の責務: リトライ、タイムアウト、ログ、エラー変換、型変換
- SDK のバージョンアップや差し替え時の影響範囲をラッパー内に閉じ込める

### ラッパーの README に以下を記載する
- 対象SDKの名前・バージョン・公式ドキュメントURL
- ラッパーが提供する関数の一覧と概要
- 認証方法（環境変数名、設定ファイル等）
- レート制限・制約事項
- テスト方法（モック/スタブの作り方）
