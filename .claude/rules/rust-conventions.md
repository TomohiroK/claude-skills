---
paths:
  - "src/**/*.rs"
---
# Rust コーディング規約

- 金融計算には必ず rust_decimal を使用する（f64 禁止）
- Leptos 0.7 CSR のコンポーネント規約に従う
- unused imports は許可しない（strict mode）
- エラーハンドリングは Result 型を使う。unwrap() は禁止
- テストは #[cfg(test)] モジュール内に記述
