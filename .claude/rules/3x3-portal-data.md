# 3x3-portal データ管理ワークフロー

## リポジトリ
`/Users/tomohirok/Documents/Github/3x3-portal/apps/web-next/`

## エンティティ間のリレーション

```
Event (event.repository.ts)
  ├── participants: EventParticipant[] → Team (team.repository.ts)
  └── location: string → Venue (venue.repository.ts) ※ findVenueByLocation で名前マッチ
```

## イベント追加・更新時の必須ワークフロー

### 1. チームの作成とリンク
イベントに参加チームが判明した場合:
1. `team.repository.ts` の MOCK_TEAMS に既存チームがあるか確認する
2. 新規チームがあれば **Team エンティティを作成** する（ID採番、slug、category、SNS等）
3. イベントの `participants` 配列に `{ teamId: number, status: 'confirmed' | 'probable' | 'speculated' }` を追加する

**禁止**: チーム名をイベントの description テキストに直書きして終わりにすること。エンティティとして作成しリレーションを設定する。

### 2. 会場の作成とリンク
イベントの開催会場が判明した場合:
1. `venue.repository.ts` の MOCK_VENUES に既存会場があるか確認する
2. 新規会場があれば **Venue エンティティを作成** する（ID採番、slug、name、region、websiteUrl）
3. イベントの `location` 文字列に会場名を含める（`findVenueByLocation` がマッチするため）

**重要**: `findVenueByLocation` は `location.includes(venue.name)` で判定する。イベントの location 文字列に Venue の name が部分一致するよう命名する。

### 3. ナビゲーションの整合性
新しいページ（/venues, /teams 等）を追加した場合、以下の3箇所全てにリンクを追加する:
- `SiteHeader.tsx` — デスクトップナビ（NAV_LINKS）
- `MobileNav.tsx` — モバイルナビ（NAV_LINKS）
- `SiteFooter.tsx` — フッターナビ

## TeamCategory 一覧
`'EXE' | 'WT' | '代表' | 'U23' | '招待' | '一般クラブ'`

新しいカテゴリを追加する場合、以下を全て更新する:
- `types/domain.ts` — TeamCategory union type
- `TeamCategoryBadge.tsx` — バッジ色
- `TeamFilters.tsx` — CATEGORY_OPTIONS + ACTIVE_CLASSES
- `teams/page.tsx` — VALID_CATEGORIES

## デプロイ
- Vercel自動デプロイ（GitHub push）
- カスタムドメイン: https://3x3.tmkproduct.com
- 手動デプロイ: `apps/web-next/` ディレクトリから `npx vercel --prod`
