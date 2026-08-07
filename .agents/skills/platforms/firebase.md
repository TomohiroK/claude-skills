---
description: Firebase の設計・構築・運用知識。Authentication、Firestore、Realtime Database、Cloud Functions、Hosting、Storage、App Check、Remote Config、Cloud Messaging を含む。
---

# Firebase スキル

## 作業開始チェックリスト

1. Firebase プロジェクト ID、GCP プロジェクトとの紐付け、環境（dev/staging/prod）を把握する
2. Spark（無料）/ Blaze（従量課金）を確認し、利用可能な機能を把握する
3. Authentication プロバイダ、Firestore ルール、Functions のデプロイ状況を確認する
4. 使用中の Firebase SDK バージョン（Web v9 modular / v8 compat / Admin SDK）を把握する
5. Blaze プランの場合、予算アラートと使用量上限の設定を確認する

## コーディングルール

### Authentication
- メール確認（email verification）を必ず有効化する
- パスワードポリシーを設定する（最低8文字、複雑性要件）
- サインアップ方法を必要最小限に制限する（不要なプロバイダを無効化）
- カスタムクレームでロールベースアクセス制御（RBAC）を実装する
- ID トークンの検証は Admin SDK で行う（クライアントのトークンを信頼しない）
- セッション管理: ID トークンの有効期限（デフォルト1時間）とリフレッシュを適切に設計する

### Firestore
- データモデリング: NoSQL の非正規化を前提とした設計（RDB の正規化を持ち込まない）
- ドキュメントサイズ上限（1MB）を意識した設計にする
- サブコレクション vs マップフィールドの選択基準:
  - 独立してクエリする → サブコレクション
  - 親と一緒に読む → マップフィールド
- 複合インデックスはクエリパターンに基づいて設計する（`firestore.indexes.json` で管理）
- バッチ書き込み（writeBatch）は500件以下で実行する
- トランザクションのリトライ上限とデッドロック回避を考慮する
- `onSnapshot` のリスナー数を最小化する（過剰なリアルタイム同期はコスト増）

### Security Rules
- **デフォルト deny**: `allow read, write: if false;` から始め、必要な権限のみ開放する
- `allow read, write: if true;` は絶対に本番で使用しない
- ルールの粒度: `read` は `get` / `list` に、`write` は `create` / `update` / `delete` に分離する
- データバリデーション: `request.resource.data` でフィールドの型・値を検証する
- 認証チェック: `request.auth != null` を全ルールの基本条件にする
- カスタムクレーム: `request.auth.token.role == 'admin'` で管理者権限を制御する
- ルールは `firestore.rules` ファイルで管理し、バージョン管理する

```
// Good: 粒度の高い Security Rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow get: if request.auth != null && request.auth.uid == userId;
      allow list: if request.auth != null && request.auth.token.role == 'admin';
      allow create: if request.auth != null && request.auth.uid == userId
                    && request.resource.data.keys().hasAll(['name', 'email'])
                    && request.resource.data.name is string;
      allow update: if request.auth != null && request.auth.uid == userId;
      allow delete: if false;
    }
  }
}
```

### Cloud Functions
- 第2世代（Gen 2 = Cloud Run ベース）を推奨
- 関数のメモリ・タイムアウトを明示的に設定する（デフォルトに依存しない）
- コールドスタートを最小化する: 不要な import を遅延読み込みにする
- Firestore トリガー: `onCreate` / `onUpdate` / `onDelete` で冪等な処理を書く
- HTTP トリガー: CORS 設定を明示する。`*` は使用しない
- シークレット管理: `defineSecret()` で Secret Manager 連携する
- リージョン: `asia-northeast1`（東京）を明示的に指定する

### Hosting
- `firebase.json` でリダイレクト・リライトを明確に設定する
- SPA: `rewrites` で全パスを `index.html` にルーティングする
- Cloud Run / Cloud Functions との連携: `rewrites` で API パスをルーティングする
- キャッシュ制御: 静的アセットに `Cache-Control` を適切に設定する
- プレビューチャネルを活用して PR ごとにプレビュー URL を発行する

### Cloud Storage
- Security Rules でファイルサイズ・MIME タイプを制限する
- アップロード先パスにユーザー ID を含め、他ユーザーのファイルにアクセスできないようにする
- 大容量ファイルは resumable upload を使用する
- 画像のリサイズ・サムネイル生成は Firebase Extensions（`storage-resize-images`）を活用する

### App Check
- 本番環境では必ず App Check を有効化する
- Web: reCAPTCHA Enterprise プロバイダを使用する
- iOS: DeviceCheck / App Attest を使用する
- Android: Play Integrity を使用する
- デバッグトークンは開発環境のみで使用し、本番には持ち込まない

## セキュリティ

- Firestore Security Rules を厳密に設定する（本番デプロイ前のレビュー必須）
- Firebase API キーのリファラー制限を設定する（GCP Console > APIとサービス > 認証情報）
- Admin SDK のサービスアカウントキーは最小権限にする
- Cloud Functions のイングレス設定を制限する（内部トラフィックのみ許可 等）
- Firebase Console へのアクセス権限を最小化する（オーナー権限の乱用禁止）
- 個人情報の保存は最小限にし、Firestore のフィールドレベルで暗号化を検討する

## テスト

- **義務**: Security Rules の変更はテスト通過後にデプロイする
- **テスト種類**:
  - Security Rules ユニットテスト: `@firebase/rules-unit-testing` で全ルールをテストする
  - Emulator Suite: `firebase emulators:start` でローカルで統合テストを実行する
  - Cloud Functions テスト: `firebase-functions-test` でトリガーの動作を検証する
  - パフォーマンステスト: Firestore のクエリレイテンシ・読み取り回数を測定する
- **完了前必須**: Emulator Suite で全サービスが正常に連携動作していることを確認する

## 完了条件

- [ ] Security Rules が最小権限で設定されている（デフォルト deny を確認）
- [ ] Security Rules のユニットテストが全てパスしている
- [ ] Authentication のサインアップ方法が必要最小限に制限されている
- [ ] App Check が有効化されている（本番環境）
- [ ] Cloud Functions のリージョンが明示的に設定されている
- [ ] API キーのリファラー制限が設定されている
- [ ] Blaze プランの場合、予算アラートが設定されている

## 連携先エージェント

- **gcp → platforms/gcp**: GCP リソース（Cloud SQL、BigQuery 等）との連携、IAM 設計
- **frontend-developer**: Firebase SDK のクライアント側統合
- **backend-architect**: Cloud Functions のAPI設計、マイクロサービス構成
- **devops-automator**: CI/CD パイプライン（Firebase CLI デプロイ自動化）
- **cloudflare → platforms/cloudflare**: Firebase Hosting + Cloudflare CDN の構成
- **infrastructure-maintainer**: 障害対応・パフォーマンスチューニング
- **finance-tracker**: Firebase 使用量・コストの月次モニタリング
