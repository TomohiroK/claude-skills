---
paths:
  - "public/**/*.html"
  - "**/lp.html"
  - "**/index.html"
---
# LP / 静的HTML ルール

- mobile-first で設計する（Tailwind CSS の sm: 以上でデスクトップ対応）
- 画像は必ず圧縮する（300KB以下目安）
- img タグには必ず alt, width, height, loading 属性を付ける
- SEO: title, meta description, OGP, canonical を必ず設定する
- フォント読み込みは display=swap を使う
- 外部CDN（Tailwind, FontAwesome等）使用時は integrity 属性を検討する
- absolute/fixed 配置の装飾要素には必ず `pointer-events-none` を付与する（詳細: `css-responsive.md`）
- リリース前にモバイル（375px幅）で全CTA/リンクを実際にクリックし遷移を確認する
- レスポンシブ検証は「見た目」だけでなく「操作（クリック/タッチ）」も含める
