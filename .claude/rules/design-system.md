---
paths:
  - "**/design-system/**"
  - "**/tokens/**"
  - "**/components/**/*.tsx"
  - "**/components/**/*.vue"
  - "**/components/**/*.rs"
  - "**/theme/**"
---
# デザインシステム構築・運用ルール

デザインパターンを再現可能にするためのシステム設計・運用規約。

---

## デザイントークン

### トークンの3層構造

```
[Layer 1: Primitive（生の値）]
  color-teal-500: #0D9488
  color-gray-100: #F3F4F6
  space-4: 16px
  font-size-400: 16px

[Layer 2: Semantic（意味付き）]
  color-bg-primary: {color-white}        → ライト
  color-bg-primary: {color-gray-900}     → ダーク
  color-text-primary: {color-gray-900}   → ライト
  color-text-primary: {color-gray-100}   → ダーク
  color-action-primary: {color-teal-500}
  space-card-padding: {space-4}

[Layer 3: Component（コンポーネント固有）]
  button-bg: {color-action-primary}
  button-padding-x: {space-4}
  card-border-radius: {space-3}
```

### トークン命名規約
- 形式: `{category}-{property}-{variant}`
- カテゴリ: `color`, `space`, `font`, `radius`, `shadow`, `motion`
- 属性: `bg`, `text`, `border`, `size`, `weight`, `duration`
- バリアント: `primary`, `secondary`, `tertiary`, `inverse`, `error`, `success`

```
例:
  color-bg-primary       → メイン背景色
  color-text-secondary   → 補助テキスト色
  space-component-gap    → コンポーネント間の間隔
  radius-card            → カードの角丸
  shadow-elevated        → 浮き上がり影
  motion-duration-fast   → 短いアニメーション時間
```

### 禁止事項
- ハードコード値の直接使用（`#0D9488` → `var(--color-action-primary)` を使う）
- トークン名に具体的な値を含めない（`color-teal` → `color-action-primary`）
- ダークモード対応をメディアクエリ内のハードコードで行わない（Semantic層で切り替える）

---

## コンポーネント設計

### コンポーネント仕様テンプレート

新規コンポーネント作成時に以下を定義する:

```markdown
## コンポーネント名: Button

### バリアント
| バリアント | 用途 |
|-----------|------|
| primary   | 主要アクション（1画面に1つ推奨） |
| secondary | 副次アクション |
| ghost     | 三次アクション、ナビゲーション |
| danger    | 破壊的アクション（削除等） |

### ステート
| ステート | 視覚変化 |
|---------|---------|
| default | 標準表示 |
| hover   | 背景色 10% 暗く |
| focus   | フォーカスリング（2px, offset 2px） |
| active  | 背景色 20% 暗く、scale(0.98) |
| disabled | opacity 0.5, cursor not-allowed |
| loading | テキスト非表示、スピナー表示 |

### サイズ
| サイズ | 高さ | パディング | フォント |
|-------|------|-----------|--------|
| sm    | 32px | 12px 16px | 14px   |
| md    | 40px | 12px 20px | 16px   |
| lg    | 48px | 16px 24px | 18px   |

### アクセシビリティ
- role: button
- aria-label: アイコンのみのボタンには必須
- aria-disabled: disabled時にtrueを設定
- キーボード: Enter/Space で発火
```

### コンポーネント階層

```
[Atoms（原子）] — 最小単位、単独で意味を持つ
  Button, Input, Label, Icon, Badge, Avatar, Toggle

[Molecules（分子）] — Atomsの組み合わせ
  SearchBar (Input + Icon + Button)
  FormField (Label + Input + ErrorMessage)
  Card (Image + Title + Description + Badge)

[Organisms（有機体）] — Moleculesの組み合わせ、画面の大きなセクション
  Header (Logo + Navigation + Avatar)
  CardGrid (Card × N + Pagination)
  FilterPanel (SearchBar + FilterChips + ResultCount)
```

### 禁止事項
- 全ステート（default, hover, focus, active, disabled, loading, error, empty）を定義せずにコンポーネントを完成とみなさない
- 1コンポーネントに5つ以上のバリアントを持たせない（設計を見直す）
- コンポーネント内にビジネスロジックを含めない（表示と操作に専念）

---

## ビジュアルQA（デザイン品質チェック）

### チェックリスト

機能テストに加えて、以下のビジュアル品質チェックを実施する:

| # | 項目 | 基準 | 検証方法 |
|---|------|------|---------|
| 1 | スペーシングの一貫性 | トークンで定義された値のみ使用 | DevTools で computed style を確認 |
| 2 | タイポグラフィの一貫性 | トークンで定義されたスケールのみ使用 | DevTools で font-size/weight 確認 |
| 3 | カラーの一貫性 | Primitiveにない色が使われていない | CSS変数参照の確認 |
| 4 | ダークモード | 全画面・全ステートで破綻なし | prefers-color-scheme: dark で確認 |
| 5 | エンプティステート | 全データ空画面にイラスト+CTA | データ0件で全画面を巡回 |
| 6 | ローディングステート | スケルトン or プレースホルダー表示 | ネットワーク遅延シミュレーション |
| 7 | エラーステート | エラーメッセージが具体的 | 各フォームで不正入力 |
| 8 | テキスト溢れ | 長い文字列でレイアウトが崩れない | 最大長テキストで確認 |
| 9 | 画像なし | 画像読み込み失敗時にフォールバック表示 | 画像URLを無効化 |
| 10 | RTL対応 | 必要な場合、右から左のレイアウト | dir="rtl" で確認 |

### テキスト溢れテスト用データ

```
短い: "OK"
普通: "設定を変更しました"
長い: "このアクションは取り消すことができません。本当に実行してもよろしいですか？"
最大: "あ" × 200文字（各フィールドの最大長）
```

---

## デザイン・開発ハンドオフ

### ハンドオフ時の必須納品物

| 成果物 | 責任者 | 内容 |
|--------|-------|------|
| コンポーネント仕様 | ui-designer | 上記テンプレートに準拠 |
| デザイントークン一覧 | ui-designer | JSON or CSS Custom Properties |
| インタラクション仕様 | ui-designer + motion-designer | トリガー・duration・easing |
| レスポンシブブレークポイント | ui-designer | 各ブレークポイントでの変化点 |
| アクセシビリティ要件 | ui-designer | role, aria属性, キーボード操作 |

### 確認フロー

```
ui-designer → brand-guardian（ブランド整合性）
           → frontend-developer（技術的実現可能性）
           → qa-engineer（テスト計画策定）
```

---

## 参照ルール
- UIパターン詳細: `ui-design-patterns.md`
- レスポンシブ設計: `css-responsive.md`
- ブランド管理: `branding.md`
- コーディング品質: `coding-standards.md`
