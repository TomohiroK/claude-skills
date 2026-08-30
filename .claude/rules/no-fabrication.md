# 実在を確認していないものを書かない

## 第一原則: URL・ドメイン・製品名は、実在を確認してからでなければ書けない

もっともらしいURLを書くことは、推測を事実として配布することである。
そのアドレスが第三者の実在サイトだった場合、被害は自分のプロジェクトの外に出る。

## 禁止事項

| 禁止 | 理由 |
|------|------|
| デプロイ前に「たぶんこのURLになる」と推測して書く | まだ存在しないか、他人のものである |
| ドメインをコードに直書きする | 変わったとき全箇所を直すことになる。そして直し漏れる |
| 確認していない外部サービスの挙動を断定して書く | 実装の根拠にされ、後で全部崩れる |
| 「〜のはず」で製品名・APIの仕様を書く | 検証コストを読み手に転嫁している |

## URL の扱い

### コードに書かない。環境から受け取る

```ts
// Bad: 推測でも実在でも、直書きした時点で負債
export const siteUrl = "https://example.vercel.app";

// Good: 環境変数から受け取る。コードにドメインを書く余地をなくす
function resolveSiteUrl(): string {
  const explicit = process.env.NEXT_PUBLIC_SITE_URL;
  if (explicit) return explicit.replace(/\/$/, "");
  const platform = process.env.VERCEL_PROJECT_PRODUCTION_URL;
  if (platform) return `https://${platform}`;
  return "http://localhost:3000";
}
```

### 書く必要がある場合の手順

1. 実際にアクセスして 200 が返ることを確認する
2. アクセスできない環境にいる場合、**書かずにユーザーに確認する**
3. DNS が解決することは「存在する」の証拠にならない。ワイルドカードのサブドメインは
   デタラメな名前でも解決する

```bash
# これは判定に使えない
$ getent hosts prompt.vercel.app                    → 216.198.79.195
$ getent hosts zzq7x-does-not-exist-9931.vercel.app → 216.198.79.67   # 同じように解決する
```

## 発生事例（2026-08-30）

サイトの canonical・OGP・sitemap の全42URLに、デプロイ前の推測で
`https://chat-game.vercel.app` を書き込んだ。**このアドレスは第三者の実在サイト**
（ポルトガル語のチャットアプリ）だった。

ユーザーはそのURLを開き、他人のサイトを自分のサイトだと思ってデザインの
フィードバックを返した。こちらはその指摘に従って、自分の作ったデザインを
削除した。指摘の対象は最初から存在しないものだった。

修正: コードからドメインを削除し、環境変数から受け取る形にした。
`lib/site.ts` に「ここに固定値を書かない」理由をコメントとして残した。

## 関連
- `.claude/rules/deploy-target.md`
- `.claude/rules/verification-scope.md`
