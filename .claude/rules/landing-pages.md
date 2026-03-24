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
