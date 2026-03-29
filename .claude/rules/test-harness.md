# テストハーネス設計ルール

## テストハーネスとは

テストハーネスは「テストツール」ではない。**設計・コーディング・テスト・フィードバック対応の全フェーズを貫く構造的な考え方**である。

コードが「ハーネスに載る」状態で設計・実装されていなければ、テストフェーズでモックを作ろうとしても手遅れになる。

## 全フェーズへの影響

```
設計 → 「このコードはどうやってテストするか」を最初に決める
  ↓
コーディング → ハーネスに載る構造で書く（DI、状態分離、リソース管理API）
  ↓
テスト → ハーネス上で Setup → Exercise → Verify → Teardown を回す
  ↓
フィードバック → バグ報告を受けたら、まずハーネス上で再現する
```

**「テスト時にモックを考える」のでは遅い。設計時にハーネスポイントを決める。**

---

## Phase 1: 設計段階のハーネス設計

### リソース棚卸し表の作成（必須）

リアルタイム通信・ブラウザAPI・外部サービスを使う機能の設計時、以下の表を必ず作成する:

```markdown
| # | リソース | 変数名 | 作成 | 破棄 | 再利用 | モック方法 |
|---|---------|--------|------|------|--------|-----------|
| 1 | WebSocket | ws | start() | goBack() | No | MockWebSocket stub |
| 2 | AudioContext(録音) | audioCtx | start() | goBack() | No | standardized-audio-context-mock |
| 3 | AudioContext(再生) | playCtx | playNext() | goBack() | Yes | 同上 |
| 4 | MediaStream | mediaStream | getUserMedia | stopRecording() | No | navigator.mediaDevices mock |
| 5 | Timer | lateTranslateTimer | onmessage | goBack()/onclose | N/A | clearTimeout検証 |
```

**「モック方法」列が空欄 = テストできない設計 = 設計やり直し。**

### 状態遷移図の作成（必須）

状態リセットを伴う機能では、ループを含む状態遷移図を描く:

```
[初期] → start() → [会話中] → goBack() → [初期] → start() → [会話中] → ...
                         ↓ エラー
                    [エラー] → ws.onclose → [初期]
```

ループが存在する = n回テストが必須。ループがない設計図は疑う。

### ハーネスポイントの設計

コードの中で「外部依存に触れる箇所」を明示的にハーネスポイントとして設計する:

| ハーネスポイント | 本番 | テスト時 |
|----------------|------|---------|
| WebSocket接続 | `new WebSocket(url)` | MockWebSocket stub |
| マイク取得 | `navigator.mediaDevices.getUserMedia()` | mock MediaStream |
| 音声再生 | `new AudioContext()` | mock AudioContext |
| API呼び出し | `fetch("/translate", ...)` | mock fetch |
| タイマー | `setTimeout(fn, ms)` | fake timers |

**設計ドキュメントにこの表がなければ、設計レビュー不合格。**

---

## Phase 2: コーディング段階のハーネス対応

### 原則: コードはハーネスに載る構造で書く

#### リソース作成を差し替え可能にする

```javascript
// Bad: 直接newしている。テスト時に差し替え不可能
function start() {
  const ws = new WebSocket(url);
  const ctx = new AudioContext();
}

// Good: ファクトリ経由で作成。テスト時にモックを注入可能
const createWs = (url) => new WebSocket(url);
const createAudioCtx = (opts) => new AudioContext(opts);

function start(wsFactory = createWs, audioFactory = createAudioCtx) {
  const ws = wsFactory(url);
  const ctx = audioFactory({ sampleRate: 16000 });
}
```

小規模アプリ（index.html単体等）でファクトリパターンが過剰な場合は、**グローバル変数のモック差し替え**で代替可能:

```javascript
// テスト時に window.WebSocket を差し替える
window.WebSocket = MockWebSocket;
```

#### 状態を一箇所に集約する

```javascript
// Bad: 状態が散らばっている（テストで検証しづらい）
let ws, audioCtx, playCtx, isRecording, playQueue, isPlaying, timer;

// Good: 状態オブジェクトに集約（テストで一括検証可能）
const state = {
  ws: null,
  audioCtx: null,
  playCtx: null,
  isRecording: false,
  playQueue: [],
  isPlaying: false,
  timer: null,
};

// テスト時: 全状態を一括で検証
function verifyCleanState(state) {
  assert(state.ws === null);
  assert(state.audioCtx === null);
  assert(state.playCtx === null);
  assert(state.isRecording === false);
  assert(state.playQueue.length === 0);
  assert(state.isPlaying === false);
  assert(state.timer === null);
}
```

#### クリーンアップを関数として独立させる

```javascript
// Bad: クリーンアップロジックがイベントハンドラに散らばっている
ws.onclose = () => {
  ws = null;
  clearTimeout(timer);
  // ... 20行のクリーンアップ
};

// Good: 独立した関数。テストから直接呼べる
function cleanup() {
  clearTimeout(timer); timer = null;
  playQueue = []; isPlaying = false;
  stopRecording();
  audioCtx?.close(); audioCtx = null;
  playCtx?.close(); playCtx = null;
}

ws.onclose = () => {
  ws = null;
  cleanup();
  resetUI();
};
```

---

## Phase 3: テスト段階のハーネス実行

### 4フェーズパターン（Setup → Exercise → Verify → Teardown）

全テストはこの4フェーズで構成する。特に **Teardown の省略は禁止**。

```javascript
// 各ラウンドの構造
for (let round = 1; round <= 3; round++) {
  // Setup: リソース初期化
  start();

  // Exercise: 操作を実行
  simulateAIResponse("Hello");
  goBack();

  // Verify: 状態を検証
  assert(ws === null, `Round ${round}: ws not null`);
  assert(audioCtx === null, `Round ${round}: audioCtx not null`);
  assert(playCtx === null, `Round ${round}: playCtx not null`);
  assert(isRecording === false, `Round ${round}: still recording`);
  assert(document.getElementById("transcript").children.length === 0,
    `Round ${round}: transcript not cleared`);

  // Teardown: テスト環境のリセット（次ラウンドのSetupに影響しないよう）
  // → この例では goBack() が Teardown を兼ねている
}
```

### マルチラウンドテスト（最低3ラウンド）

参照: `development-workflow.md`「マルチラウンドテストの義務化」

- Round 1: 初期状態からの動作確認
- Round 2: リソースリーク・状態不整合の検出
- Round 3: 累積バグの検出

### モックツールの選定基準

| 対象 | 推奨モック | 用途 |
|------|-----------|------|
| AudioContext | `standardized-audio-context-mock` | ユニットテスト |
| MediaStream / getUserMedia | `@eatsjobs/media-mock` or `vi.fn()` | ユニットテスト |
| WebSocket（ユニット） | カスタム MockWebSocket stub | ユニットテスト |
| WebSocket（E2E） | Playwright `page.routeWebSocket()` (v1.48+) | 統合テスト |
| マイク/カメラ（Chromium） | `--use-fake-device-for-media-stream` フラグ | ブラウザ統合テスト |
| fetch | `vi.fn()` or Playwright `page.route()` | 全層 |
| タイマー | `vi.useFakeTimers()` | ユニットテスト |

### プレビュー環境でのハーネス（index.html単体アプリ）

テストフレームワークを導入できない軽量アプリ（index.html + server.js等）では、`preview_eval` でハーネスを直接実行する:

```javascript
// preview_eval で実行するハーネススクリプト
(async () => {
  const results = [];
  for (let round = 1; round <= 3; round++) {
    // Setup + Exercise
    start("Kore");
    await new Promise(r => setTimeout(r, 500));
    // 擬似メッセージでAI応答をシミュレート
    // ...
    goBack();
    await new Promise(r => setTimeout(r, 300));

    // Verify
    results.push({
      round,
      ws: ws === null,
      audioCtx: audioCtx === null,
      playCtx: playCtx === null,
      isRecording: isRecording === false,
      transcriptCleared: document.getElementById("transcript").children.length === 0,
    });
  }
  document.title = JSON.stringify(results);
})();
```

---

## Phase 4: フィードバック対応段階のハーネス活用

### バグ報告 → ハーネスで再現 → 修正 → ハーネスで検証

```
1. バグ報告を受ける
2. ハーネス上で再現テストを書く（まず壊れるテストを作る）
3. テストが赤（失敗）であることを確認
4. コードを修正する
5. テストが緑（成功）に変わることを確認
6. 3ラウンド回して他に壊れていないことを確認
```

**「修正してからテストする」のではなく「テストしてから修正する」。**

### 再現テストのテンプレート

```javascript
// バグ報告: 「2回目の会話でマイクが死ぬ」
// → 再現ハーネス
it("2回目の会話でもマイクが動作する", async () => {
  // Round 1
  start("Kore");
  simulateAIResponse("Hello");
  goBack();
  verifyCleanState();

  // Round 2（ここで壊れていた）
  start("Kore");
  // マイクが動作することを検証
  assert(isRecording === true, "Recording should be active in round 2");
  assert(audioCtx.state === "running", "AudioContext should be running in round 2");
  goBack();
  verifyCleanState();
});
```

### パニック時こそハーネスに戻る

バグが連鎖して発生した時（CTOがテンパる時）こそ、場当たり修正をやめてハーネスに戻る:

1. **止まる** — 場当たり修正を即座にやめる
2. **ハーネスを書く** — 現在の状態を検証するテストを作る
3. **赤を確認** — テストが失敗することを確認（問題の正確な特定）
4. **修正** — テストが緑になるよう修正
5. **3ラウンド** — 他に壊れていないことを確認

参照: `development-workflow.md`「パニック時プロトコル」

---

## テスト層の設計（規模別）

### 小規模（index.html 単体）
- preview_eval でハーネススクリプトを実行
- モックは window オブジェクトの差し替えで対応

### 中規模（SPA / コンポーネント）
- Vitest + JSDOM（ロジック層）
- Vitest Browser Mode（ブラウザAPI層）
- モックライブラリ活用

### 大規模（マルチサービス）
- 上記 + Playwright E2E
- CDP ヒープスナップショットによるメモリリーク自動検出
- WebSocket: `page.routeWebSocket()` でモック

---

## 発生事例（2026-03-29）

EchoTalk2で以下の全てが「ハーネス設計の欠如」に起因:

| 問題 | ハーネスがあれば |
|------|----------------|
| AudioContext枯渇（2回目でマイク死亡） | リソース棚卸し表で上限を設計段階で認識。n回テストで検出 |
| クリーンアップ順序ミス | 状態検証関数 `verifyCleanState()` で一括検出 |
| タイマー未クリア | Teardownフェーズの検証で検出 |
| transcript未クリア | DOM状態検証で検出 |
| CTOパニックによる場当たり修正 | ハーネスに戻る手順で冷静に対応 |
