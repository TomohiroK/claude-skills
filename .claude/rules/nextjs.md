# Next.js 開発ルール

## Server Component / Client Component の境界

### Client Component追加の判断基準
- Client Componentは「最後の手段」。まずServer Componentで実現可能かを検討する
- 以下の全てに該当する場合のみClient Componentを追加する:
  1. useState, useEffect, useRouter等のReact Hooksが必須
  2. Server Componentでは実現不可能であることを確認済み
  3. 静的HTML + CSS（:hover, :focus等）で代替できないことを確認済み

### Server Componentで十分なケース
- ナビゲーションリンク → `<a>` タグの静的リンクで十分
- ホバーエフェクト → CSS `:hover` で十分
- アクティブ状態の表示 → サーバーサイドでのクラス付与で十分
- 静的コンテンツの条件表示 → サーバーサイドの条件分岐で十分

### Client Component追加時の注意
- Server Component内にClient Componentを埋め込む場合、propsのシリアライズ制約を確認する
- `--prefix` 等の非標準実行環境ではwebpackのモジュール解決が壊れる可能性がある
- 新規Client Component追加後は必ず `npm run dev` で動作確認する（ビルド成功 != ランタイム正常）

## --prefix 環境での開発

- `npm run dev --prefix <path>` はwebpackのcwd（カレントワーキングディレクトリ）に影響する
- この環境でClient Componentの追加は特にリスクが高い（モジュール解決の失敗）
- `TypeError: Cannot read properties of undefined (reading 'call')` はwebpackのモジュール解決失敗の典型的なエラー
- このエラーが出た場合、まず直近のClient Component変更を疑う

## metadata API の対応範囲

- `metadata.alternates.types` は RSS/Atom フィード等の標準的な alternate type のみサポートする
- 任意の MIME type（`text/plain` 等）を設定しても `<link>` タグは生成されない
- 非対応のケースでは `layout.tsx` の `<head>` タグに直接 `<link>` を記述する
- フレームワークの抽象化APIを使う前に、公式ドキュメントで対応範囲を確認する

```tsx
// metadata API では動かないケース
export const metadata = {
  alternates: { types: { "text/plain": "/llms.txt" } }, // link タグが生成されない
};

// 代替: layout.tsx で直接記述
<html>
  <head>
    <link rel="alternate" type="text/plain" title="LLMs.txt" href="/llms.txt" />
  </head>
  ...
</html>
```

## 動的ルートと静的ファイルの競合

- `[slug]` や `[...catchAll]` 動的ルートがある場合、`public/` に追加した静的ファイルが動的ルートに遮蔽される場合がある
- 例: `app/(serverLayout)/[jobType]/` があると、`/llms.txt` が `[jobType]` にマッチし404になる
- 対策: `next.config.js` の `rewrites({ beforeFiles })` で静的ファイルを明示的にルーティングする

```js
// next.config.js
async rewrites() {
  return {
    beforeFiles: [
      { source: "/llms.txt", destination: "/llms.txt" },
      { source: "/llms-full.txt", destination: "/llms-full.txt" },
    ],
  }
}
```

- `public/` にファイルを追加する際は、動的ルートとの競合を必ず確認する
- `.xml` 等の拡張子は動的ルートにマッチしにくいが、`.txt` はマッチしやすい

## JSX の HTML 属性名はキャメルケース

- JSXではHTML属性名をキャメルケースに変換する必要がある
- よくある変換: `class` → `className`, `hreflang` → `hrefLang`, `tabindex` → `tabIndex`, `for` → `htmlFor`
- TypeScriptの型チェックで検出されるが、ビルド時にしか分からないため注意

## デプロイ前のローカル検証

- metadata / head の変更後は、デプロイ前にローカルでHTML出力を確認する
- `curl -s localhost:3000 | grep -i '<link'` で link タグの存在を確認する
- 「静的な設定変更だからローカル確認は不要」と判断しない

## --prefix 環境でのHMR制限

- `--prefix` 環境では layout.tsx 等のルートレベルファイル変更後に HMR が機能しない場合がある（黒画面）
- この場合、devサーバーの再起動が必要
- 本番環境では正常動作するため、ローカルの黒画面だけで「壊れた」と判断しない
- layout.tsx 変更時は devサーバー再起動 → curl でSSR確認 の手順を取る

## ビルド成功とランタイム正常は別

- `next build` の成功はSSRの正常動作を保証するが、クライアントサイドのランタイムエラーは検出しない
- **ローカルビルド成功は本番（Vercel）での動作を保証しない** — 特にネイティブ依存ライブラリ
- 新規コンポーネント追加後は:
  1. `npm run build` でビルド確認
  2. `npm run dev` でブラウザ確認（または curl でSSR確認 + ブラウザでCSR確認）
  3. SSRは正常だがブラウザでエラーが出る場合、Client Component/webpack関連を疑う

## サーバーレス環境でのライブラリ互換性

### Vercel Serverless で動作しないライブラリのパターン
- **ネイティブバイナリ依存**: `jsdom`, `canvas`, `sharp`（sharpはVercel内蔵で別途対応）等
- **OS固有API依存**: ファイルシステムの永続書き込み、子プロセス起動等
- **大きなバンドルサイズ**: サーバーレス関数の50MBサイズ制限を超えるもの

### 新規ライブラリ追加時のチェック（Server Component で使用する場合）
1. `npm ls <package>` で依存ツリーにネイティブモジュールがないか確認
2. Vercel公式の既知の非互換リストを確認
3. 代替手段を検討（例: `isomorphic-dompurify` → データ制御下なら不要、`sharp` → `next/image` 内蔵）

### 発生事例（2026-03-28）
`isomorphic-dompurify`（`jsdom` 依存）をブログのHTML sanitization に採用。ローカルビルド・dev 環境では正常動作したが、Vercel 本番で 500 Internal Server Error。原因: `jsdom` のネイティブ依存がサーバーレスランタイムで利用不可。修正: データが自前 JSON（ユーザー入力経路なし）のため sanitization 自体を削除
