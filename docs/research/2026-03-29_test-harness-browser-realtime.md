# トレンドリサーチレポート

## エグゼクティブサマリー

- **調査テーマ**: ブラウザリアルタイムアプリ向けテストハーネスのベストプラクティス（2025〜2026）
- **調査期間**: 2026-03-29
- **主要発見事項**:
  1. Vitest 4.0（2025年末）でブラウザモードが安定版となり、JSDOM依存のテストから実ブラウザ実行への移行が加速している
  2. Playwright v1.48以降で `WebSocketRoute` APIが追加され、WebSocketのモック・傍受がネイティブサポートされた
  3. AudioContext/MediaStream のモックは専用ライブラリ（`standardized-audio-context-mock`, `@eatsjobs/media-mock`）が整備されており、ユニットテスト層での完全代替が可能になっている

---

## テストハーネスの定義と構成要素

### テストハーネスとは

テストハーネス（Test Harness）は、テストの実行を自動化するための構造的な枠組みである。テストエンジン、テストスクリプト、スタブ・ドライバ、アサーション機構の集合体として機能し、被テストコードを制御された環境で実行・検証する。

### 4フェーズパターン（Gerard Meszaros "xUnit Patterns" より）

テストハーネスの標準的な構成は以下の4フェーズで成立する。「Arrange-Act-Assert（AAA）」や「Given-When-Then」と本質的に同じ構造を持つ。

```
┌─────────────────────────────────────────────────────────┐
│  Phase 1: Setup（セットアップ）                          │
│  → テスト環境・リソース・スタブ・モックを初期化する       │
│  → beforeEach / beforeAll スコープで実施                │
├─────────────────────────────────────────────────────────┤
│  Phase 2: Exercise（実行）                               │
│  → テスト対象のコードを実際に呼び出す                    │
│  → 副作用・状態遷移・イベント発火を起動する               │
├─────────────────────────────────────────────────────────┤
│  Phase 3: Verify（検証）                                 │
│  → 実際の結果が期待値と一致するかを検証する               │
│  → リソースリーク・状態不整合・エラーも検証対象           │
├─────────────────────────────────────────────────────────┤
│  Phase 4: Teardown（解体）                               │
│  → リソースを確実に解放・リセットする                    │
│  → afterEach / afterAll スコープで実施                  │
│  → Teardown の失敗は次テストのフォールスポジティブを生む  │
└─────────────────────────────────────────────────────────┘
```

**原則**: Setup を高速に保つ。Teardown は例外発生時でも必ず実行されるよう設計する（`try/finally` パターン）。

---

## ブラウザリアルタイムアプリ向けベストプラクティス

### なぜ JSDOM では不十分か

JSDOMは長年デフォルトのテスト環境として機能してきたが、以下の問題を持つ。

- Web APIの約20%が欠損または動作が異なる
- `AudioContext`、`MediaStream`、`ResizeObserver`、CSS Custom Properties、Web Animations API が正常動作しない
- WebSocketはスタブが必要で、実際のイベントループ動作と乖離が生じる
- `getUserMedia`、`AudioWorklet` などはJSDOMでは一切動作しない

**推奨**: ユニット/ロジックテストはJSDOM（高速）、ブラウザAPIを直接使うテストは実ブラウザ実行（Vitest Browser Mode or Playwright）に二分する設計を採用する。

### テスト層の分離設計

```
┌──────────────────────────────────────────────────────────┐
│ Layer 3: E2E テスト（Playwright E2E）                     │
│ - シナリオ全体の動作検証                                  │
│ - WebSocket + Audio + Media の統合フロー                  │
│ - 実際のサーバー or WS Mock サーバーを使用                │
├──────────────────────────────────────────────────────────┤
│ Layer 2: ブラウザ統合テスト（Vitest Browser Mode）         │
│ - 実ブラウザAPIを使うコンポーネント・モジュールの検証       │
│ - AudioContext, MediaStream のリアル動作確認              │
│ - Playwright フェイクデバイスフラグを活用                 │
├──────────────────────────────────────────────────────────┤
│ Layer 1: ユニットテスト（Vitest + JSDOM）                 │
│ - ビジネスロジック、状態管理、純粋関数の検証              │
│ - AudioContext → standardized-audio-context-mock         │
│ - getUserMedia → @eatsjobs/media-mock / jest.fn()        │
│ - WebSocket → Vitest vi.fn() + カスタムスタブ            │
└──────────────────────────────────────────────────────────┘
```

### AudioContext のモック戦略（ユニットテスト層）

```typescript
// standardized-audio-context-mock を使用
import { AudioBuffer, AudioContext, registrar } from 'standardized-audio-context-mock';

describe('AudioPlayer', () => {
  let audioContextMock: AudioContext;

  beforeEach(() => {
    audioContextMock = new AudioContext();
  });

  afterEach(() => {
    registrar.reset(); // テスト間の状態汚染を防ぐ
  });

  it('バッファソースを作成して再生する', () => {
    const buffer = new AudioBuffer({ length: 10, sampleRate: 44100 });
    play(buffer, audioContextMock);
    // assertions...
  });
});
```

### getUserMedia / MediaStream のモック戦略（ユニットテスト層）

```typescript
// パターンA: Object.defineProperty（シンプルケース）
const mockMediaDevices = {
  getUserMedia: vi.fn().mockResolvedValueOnce(new MediaStream()),
};
Object.defineProperty(window.navigator, 'mediaDevices', {
  writable: true,
  value: mockMediaDevices,
});

// パターンB: @eatsjobs/media-mock（フル機能が必要な場合）
import { MediaMock } from '@eatsjobs/media-mock';
MediaMock.install();
afterEach(() => MediaMock.restore());
```

### WebSocket のモック戦略

**Playwright を使う場合（統合・E2E テスト層）**:

```typescript
// Playwright v1.48以降: page.routeWebSocket() でネイティブモック
await page.routeWebSocket('wss://example.com/ws', (ws) => {
  ws.onMessage((message) => {
    if (message === 'ping') ws.send('pong');
  });
  // connectToServer() を呼ばない → 完全モックモード
});
```

**Vitest を使う場合（ユニットテスト層）**:

```typescript
// カスタム WebSocket スタブ
class MockWebSocket {
  static OPEN = 1;
  readyState = MockWebSocket.OPEN;
  onmessage: ((ev: MessageEvent) => void) | null = null;
  onclose: (() => void) | null = null;
  sentMessages: string[] = [];

  send(data: string) { this.sentMessages.push(data); }
  close() { this.onclose?.(); }
  simulateMessage(data: string) {
    this.onmessage?.(new MessageEvent('message', { data }));
  }
}

vi.stubGlobal('WebSocket', MockWebSocket);
```

### Playwright でのメディアデバイス対応（ブラウザ統合テスト層）

```typescript
// Chromium のフェイクデバイスフラグを使用
const browser = await chromium.launch({
  args: [
    '--use-fake-device-for-media-stream',   // フェイクカメラ・マイク
    '--use-fake-ui-for-media-stream',       // メディア許可ダイアログを自動許可
  ],
});
```

**注意**: WebKit と Firefox での `getUserMedia` サポートは2026年3月時点でも制限がある（[Playwright Issue #5444](https://github.com/microsoft/playwright/issues/5444)）。クロスブラウザが必要な場合は WebdriverIO を検討。

---

## リソースライフサイクルハーネスの設計パターン

### 基本原則：n サイクルテスト

「作成 → 使用 → 破棄 → 再作成」を n 回繰り返す設計で、以下を検証する。

- メモリリークがないか（ヒープが増加し続けないか）
- 2回目以降のサイクルで初回と同じ動作が再現されるか
- リソース解放漏れが蓄積されないか

```typescript
// リソースライフサイクルテスト の雛形
describe('WebSocket ライフサイクル（n サイクル）', () => {
  const CYCLE_COUNT = 10;

  it(`接続 → メッセージ → 切断 を ${CYCLE_COUNT} 回繰り返しても状態が一定`, async () => {
    for (let i = 0; i < CYCLE_COUNT; i++) {
      // Setup（各サイクル）
      const ws = new MockWebSocket('wss://example.com/ws');
      let receivedMessages: string[] = [];
      ws.onmessage = (ev) => receivedMessages.push(ev.data);

      // Exercise
      ws.send(`request-${i}`);
      ws.simulateMessage(`response-${i}`);

      // Verify
      expect(receivedMessages).toHaveLength(1);
      expect(receivedMessages[0]).toBe(`response-${i}`);
      expect(ws.sentMessages).toHaveLength(1);

      // Teardown（各サイクル）
      ws.close();
      receivedMessages = [];
    }
  });
});
```

### メモリリーク検出パターン

**Chrome DevTools Heap Snapshot を用いた手動検証フロー**:

1. 1サイクル実行してヒープスナップショットを取得（ウォームアップ）
2. n サイクル実行してヒープスナップショットを再取得
3. スナップショット差分で `AudioBufferSourceNode`、`MediaStreamTrack`、`EventListener` の残存を確認

**自動化アプローチ（Playwright + CDP）**:

```typescript
// Chrome DevTools Protocol でヒープサイズを計測
const client = await page.context().newCDPSession(page);
await client.send('HeapProfiler.enable');

// n サイクル実行前後のヒープサイズ差分を検証
const before = await client.send('Runtime.getHeapUsage');
await runNCycles(page, 10);
await client.send('Runtime.collectGarbage');
const after = await client.send('Runtime.getHeapUsage');

// 許容範囲内（例: 1MB以内）であることを検証
expect(after.usedSize - before.usedSize).toBeLessThan(1_000_000);
```

### WebSocket リソースリーク防止パターン

```typescript
// BAD: リスナーを解放しない
beforeEach(() => {
  socket = new WebSocket('wss://...');
  socket.addEventListener('message', handler);
});
// afterEach がない → リスナーが蓄積される

// GOOD: afterEach で確実に解放
let socket: WebSocket;

beforeEach(() => {
  socket = new WebSocket('wss://...');
  socket.addEventListener('message', handler);
});

afterEach(() => {
  socket.removeEventListener('message', handler);
  socket.close();
  socket = null!; // 参照を切って GC を促す
});
```

### AudioContext リソースリーク防止パターン

**既知の問題**: 一部のラッパーライブラリ（`standardized-audio-context` 等）で、再生ループ時にメモリが増加するバグが報告されている（[GitHub Issue #410](https://github.com/chrisguttandin/standardized-audio-context/issues/410)）。

```typescript
// AudioContext は close() を確実に呼ぶ
afterEach(async () => {
  await audioContext.close(); // Promise を await する
  audioContext = null!;
});

// AudioBufferSourceNode は再利用しない（使い捨て設計が仕様）
// 毎回 createBufferSource() で新規作成 → 使用後 disconnect()
function playOnce(buffer: AudioBuffer, ctx: AudioContext) {
  const source = ctx.createBufferSource();
  source.buffer = buffer;
  source.connect(ctx.destination);
  source.start();
  source.onended = () => source.disconnect(); // 終了後に接続を切る
}
```

### タイマーのリーク防止

```typescript
// setInterval / setTimeout は必ずクリア
let timerId: ReturnType<typeof setInterval>;

beforeEach(() => {
  timerId = setInterval(pollStatus, 100);
});

afterEach(() => {
  clearInterval(timerId);
});
```

---

## 状態検証の自動化

### リソースリーク検出

| 手法 | 適用層 | ツール |
|------|--------|--------|
| ヒープスナップショット差分 | E2E / ブラウザ統合 | Playwright + CDP |
| WeakRef を使った参照監視 | ユニット | Vitest カスタムアサーション |
| `performance.memory` API | ブラウザ統合 | Vitest Browser Mode |
| drool（自動リーク検出） | E2E | [drool](https://github.com/samccone/drool) |

### 状態不整合検出

WebSocket や AudioContext の状態は enum で追跡し、遷移が期待通りかを検証する。

```typescript
// WebSocket readyState の遷移検証
const CONNECTING = 0, OPEN = 1, CLOSING = 2, CLOSED = 3;

it('接続 → 切断 の状態遷移が正しい', async () => {
  const ws = new WebSocket('wss://...');
  expect(ws.readyState).toBe(CONNECTING);
  await waitForOpen(ws);
  expect(ws.readyState).toBe(OPEN);
  ws.close();
  await waitForClose(ws);
  expect(ws.readyState).toBe(CLOSED);
});
```

---

## 推奨ツール/フレームワーク比較

| ツール | 用途 | 実ブラウザ | WS サポート | Audio/Media | 速度 | 推奨度 |
|--------|------|-----------|------------|-------------|------|--------|
| **Vitest 4.x** | ユニット + コンポーネント | Browser Mode で可 | カスタムスタブ | mock ライブラリ | 最速 | ★★★★★ |
| **Vitest Browser Mode** | ブラウザ統合 | Chromium/Firefox/Safari | カスタムスタブ | 実 API 使用可 | 中速 | ★★★★☆ |
| **Playwright** | E2E + 統合 | Chromium/Firefox/WebKit | `routeWebSocket()` ネイティブ | フェイクデバイス対応 | 低速 | ★★★★★ |
| **WebdriverIO** | クロスブラウザ統合 | 含む実 Safari | WebDriver経由 | 実デバイス対応 | 低速 | ★★★☆☆ |
| **k6** | 負荷/ストレステスト | 非対応（Node） | WS サポートあり | 非対応 | 高速 | ★★★★☆ |
| **drool** | メモリリーク自動検出 | Chromium | 非直接対応 | 非直接対応 | - | ★★★☆☆ |

### Vitest 4.0 (2025年末リリース) の主要変更点

- ブラウザモードが **stable** に昇格
- `toMatchScreenshot()` による組み込みビジュアルリグレッションテスト
- Playwright Traces 統合（テスト失敗時の操作履歴記録）
- Schema Matching API（Zod / Valibot / ArkType）
- VS Code 拡張で **Debug Test** ボタンが Browser Mode に対応
- npm 週次ダウンロード数: Vitest ~7M、@playwright/test ~8M（2026年3月時点）

---

## 我々の組織への導入提案

### 背景

現在の `.claude/rules/development-workflow.md` に定められた Phase 8（テスト）は qa-engineer が担当するが、ブラウザリアルタイムアプリ（EchoTalk など WebSocket + AudioContext + MediaStream を持つアプリ）においてモックテスト義務化（`no-shortcuts.md`、`development-workflow.md` 追記事例）が課題になっている。

### 推奨アーキテクチャ

```
Layer 1: Vitest + JSDOM（ユニット）
  └── ビジネスロジック、状態管理、純粋関数
  └── AudioContext → standardized-audio-context-mock
  └── getUserMedia → @eatsjobs/media-mock
  └── WebSocket → カスタム MockWebSocket スタブ

Layer 2: Vitest Browser Mode（ブラウザ統合）
  └── 実ブラウザAPIを使うモジュール・コンポーネント
  └── AudioContext のリアル動作確認
  └── MediaStream の実デバイス模倣（Chromium フェイクデバイスフラグ）

Layer 3: Playwright（E2E + 統合）
  └── WebSocket シナリオの統合テスト（routeWebSocket）
  └── n サイクル ライフサイクルテスト
  └── メモリリーク検出（CDP ヒープスナップショット）
```

### .claude/rules/ への追加提案

`.claude/rules/testing-browser-realtime.md` として以下のルールを新設することを推奨する。

**新設ルールの骨子**:

1. **モック層の明示**: `AudioContext` は `standardized-audio-context-mock`、`getUserMedia` は `@eatsjobs/media-mock`、`WebSocket` はカスタムスタブを使用する
2. **afterEach での必須クリーンアップ**: `audioContext.close()`、`ws.close()`、`registrar.reset()` を `afterEach` に必ず記述する
3. **n サイクルテストの義務化**: リソース作成・使用・破棄を含む機能は最低10サイクルのライフサイクルテストを作成する
4. **メモリリーク検出の組み込み**: 新機能のE2Eテストに CDP ヒープスナップショット差分チェックを含める
5. **`development-workflow.md` のモックテスト義務化ルールとの整合**: 「プレビューで動かせない機能はモックで代替して検証する」の具体的実装パターンとして本ルールを参照する

### 段階的導入ロードマップ

| フェーズ | 内容 | 工数目安 |
|---------|------|---------|
| Phase 1（即時） | `standardized-audio-context-mock` + `@eatsjobs/media-mock` を開発依存に追加、モックパターンを `rules/` に文書化 | 0.5日 |
| Phase 2（次スプリント） | Vitest Browser Mode を既存プロジェクトに導入、WebSocket スタブの共通ユーティリティ整備 | 1〜2日 |
| Phase 3（月次） | Playwright E2E に CDP ヒープスナップショット差分チェックを追加 | 1日 |

---

## 機会と脅威

**機会**:
- Vitest 4.0 の安定リリースにより、追加インフラなしで実ブラウザテストが可能になった
- Playwright の `routeWebSocket()` により WebSocket モックが1行で記述可能になった
- ブラウザAPIのモックライブラリが整備され、ユニットテスト層でのカバレッジが高められる

**脅威**:
- JSDOM 依存のレガシーテストは AudioContext / MediaStream のリグレッションを検出できない（フォールスポジティブ生産）
- `development-workflow.md` の「モックテスト義務化」ルールが具体的実装パターンを持たないと遵守されにくい
- Playwright の Firefox / WebKit での `getUserMedia` 制限が継続しており、クロスブラウザ音声テストに制約がある

---

## 推奨アクション

1. `.claude/rules/testing-browser-realtime.md` を新設し、モックパターン・ライフサイクルパターン・リソース解放パターンを明文化する
2. 既存の `development-workflow.md` の「モックテスト義務化」セクションに、本レポートの Layer 1〜3 アーキテクチャへの参照を追記する
3. EchoTalk 等のリアルタイムアプリのテストに `standardized-audio-context-mock` + Playwright `routeWebSocket()` を導入し、qa-engineer のテストチェックリストを具体化する

---

## 情報源一覧

| # | 出典名 | URL | 参照日 | 信頼度 |
|---|--------|-----|--------|--------|
| 1 | Playwright: WebSocketRoute API | https://playwright.dev/docs/api/class-websocketroute | 2026-03-29 | 高（公式） |
| 2 | Playwright: Network / Mock | https://playwright.dev/docs/mock | 2026-03-29 | 高（公式） |
| 3 | Vitest: Browser Mode Guide | https://vitest.dev/guide/browser/ | 2026-03-29 | 高（公式） |
| 4 | Vitest: Why Browser Mode | https://vitest.dev/guide/browser/why | 2026-03-29 | 高（公式） |
| 5 | InfoQ: Vitest 4.0 Browser Mode Stable | https://www.infoq.com/news/2025/12/vitest-4-browser-mode/ | 2026-03-29 | 高 |
| 6 | PkgPulse: Vitest BM vs Playwright CT vs WebdriverIO 2026 | https://www.pkgpulse.com/blog/vitest-browser-mode-vs-playwright-component-testing-vs-webdriverio-2026 | 2026-03-29 | 中 |
| 7 | npm: standardized-audio-context-mock | https://www.npmjs.com/package/standardized-audio-context-mock | 2026-03-29 | 高 |
| 8 | npm: @eatsjobs/media-mock | https://www.npmjs.com/package/@eatsjobs/media-mock | 2026-03-29 | 中 |
| 9 | oneuptime: Memory Leak Test Detection | https://oneuptime.com/blog/post/2026-01-24-memory-leak-test-detection/view | 2026-03-29 | 中 |
| 10 | GitHub: drool（自動メモリリーク検出） | https://github.com/samccone/drool | 2026-03-29 | 中 |
| 11 | Green Report: WebSocket Testing Essentials | https://www.thegreenreport.blog/articles/websocket-testing-essentials-strategies-and-code-for-real-time-apps/websocket-testing-essentials-strategies-and-code-for-real-time-apps.html | 2026-03-29 | 中 |
| 12 | DZone: Playwright Testing WebSockets | https://dzone.com/articles/playwright-for-real-time-applications-testing-webs | 2026-03-29 | 中 |
| 13 | GitHub: standardized-audio-context Issue #410（メモリリーク） | https://github.com/chrisguttandin/standardized-audio-context/issues/410 | 2026-03-29 | 中 |
| 14 | GitHub: Playwright Issue #5444（getUserMedia制限） | https://github.com/microsoft/playwright/issues/5444 | 2026-03-29 | 高（公式） |
| 15 | k6: WebSocket Load Testing | https://grafana.com/docs/k6/latest/using-k6/protocols/websockets/ | 2026-03-29 | 高（公式） |
| 16 | adequatica.medium: Is It Worth Mocking WebSockets by Playwright | https://adequatica.medium.com/is-it-worth-mocking-websockets-by-playwright-e611cb016ec5 | 2026-03-29 | 中 |
