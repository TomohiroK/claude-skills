# fixedassets（Ledgea）プロジェクト情報

## 概要
ASEAN 11カ国 + 日本対応の固定資産管理 SaaS（クライアントサイド完結型）

## ブランド
- 商品名: Ledgea（Ledger + Asia）— 適用済み
- タグライン EN: "The ledger built for Asia's complexity."
- タグライン JA: 「アジアの複雑さを、ひとつの台帳に。」
- ロゴ: Teal(#0D9488)基調、Plus Jakarta Sans
- ターゲット: 日系企業のASEAN現地法人（経理・管理部門）
- 参入順序: 日本 → タイ → インドネシア → ベトナム

## 技術スタック
- Rust + WebAssembly（Leptos 0.7 CSR）、ビルド: Trunk
- Tailwind CSS（mobile-first）、Space Grotesk + Inter
- IndexedDB v2、localStorage、rust_decimal（f64禁止）
- Vercel（静的SPA + Edge Middleware）
- public/ のファイルは data-trunk rel="copy-file" で明示的コピーが必要

## 対応国（11カ国 + 日本）
Japan(JPY), Singapore(SGD), Malaysia(MYR), Thailand(THB), Indonesia(IDR), Philippines(PHP), Vietnam(VND), Myanmar(MMK), Cambodia(KHR), Laos(LAK), Brunei(BND)

## 主要ファイル（/Users/tomohirok/Documents/Github/fixedassets/）
- `src/models/depreciation.rs` - 11カ国の減価償却計算
- `src/models/company.rs` - AseanCountry, Currency, CompanySetup
- `src/stores/asset_store.rs` - IndexedDB CRUD, CSV/JSON import/export
- `locales/en.json`, `locales/ja.json` - i18n
- `public/lp.html` - LP、`middleware.js` - Edge Middleware

## 注意
- fixedassets-bn は別プロジェクト（対象外）
- プラン: Free(5資産,1部門) / Paid(無制限)
